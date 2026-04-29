# Harness Engineer Playground

A testbed for AI coding-agent **harness engineering** — the discipline of designing the scaffolding (tools, prompts, permissions, feedback loops) that wraps a model into a useful agent. Forked from [fisand/f3-app](https://github.com/fisand/f3-app), then wired with a Claude-driven harness in GitHub Actions.

For the underlying Vite+ toolchain reference (commands, conventions, common pitfalls), see [AGENTS.md](AGENTS.md).

## Branch model

- `develop` — default integration branch. PRs land here. CI gates merges.
- `main` — released/stable. Manually promoted from `develop` when ready.

## Harness components (Agent = Model + Harness)

| Role | File | Purpose |
|---|---|---|
| **Generator** | `.github/workflows/claude-coder.yml` | `@claude` mention in PR/issue → Claude implements & commits to the branch |
| **Sensor** | `.github/workflows/ci.yml` | `vp run check` + `vp run build` on every push/PR. Verification feedback. |
| **Evaluator** | `.github/workflows/claude-reviewer.yml` | A separate Claude (separate context — important) reviews the diff and posts comments |
| **Guides** | this file | Feedforward — what both Claudes read before acting |

The generator and evaluator run in **separate workflow runs** so they never share context. Per Anthropic's harness-design research, single-context self-review consistently fails (the model praises its own work). Cross-context separation is the cheapest way to enforce honest evaluation.

## How to drive the harness

1. **Plan** — open a PR (or issue) with a clear description. The description IS the sprint contract.
2. **Code** — comment `@claude <task>` on the PR. The coder workflow takes over and pushes commits to the PR branch.
3. **Test** — CI runs automatically (`vp run check`, `vp run build`).
4. **Review** — the reviewer workflow runs on every PR push and posts inline comments.
5. **Merge** — once CI is green and review is satisfied, merge to `develop`.

## Review criteria (for the reviewer agent)

Be substantive. Skip nitpicks — `oxlint`/`oxfmt` already handle style. Specifically look for:
- Logic errors, missed edge cases
- Security: prompt-injection in workflow YAML, secret exposure, unsafe shell expansion
- Mismatch between PR description and the actual diff
- Cross-file consistency in this monorepo (apps/* and packages/*)
- Test coverage for new behavior

Do not rubber-stamp. If the PR is clean, say so in one line.

## Test commands

- `vp install` — install dependencies
- `vp run check` — format + lint + typecheck (oxfmt, oxlint, tsc — see root `package.json`)
- `vp run build` — turbo build all apps
- `vp test` — run vitest

## Public-repo safety

This repo is public, so `@claude` triggers gate on `author_association` (OWNER / COLLABORATOR / MEMBER only). Do not relax this without thinking about prompt-injection from external comments. See `.github/workflows/claude-coder.yml`.
