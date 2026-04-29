# Harness Engineer Playground

A testbed for AI coding-agent **harness engineering** — the discipline of designing the scaffolding (tools, prompts, permissions, feedback loops) that wraps a model into a useful agent. Forked from [fisand/f3-app](https://github.com/fisand/f3-app), then wired with a Claude-driven harness in GitHub Actions.

## Branch model

- `develop` — default integration branch. PRs land here. CI gates merges.
- `main` — released/stable. Manually promoted from `develop` when ready.

## Harness components (Agent = Model + Harness)

| Role | File | Purpose |
|---|---|---|
| **Generator** | `.github/workflows/claude-coder.yml` | `@claude` mention in PR/issue → Claude implements & commits to the branch |
| **Sensor** | `.github/workflows/ci.yml` | `vp run check` + `vp run build` on every push/PR. Verification feedback. |
| **Evaluator** | `.github/workflows/claude-reviewer.yml` | A separate Claude (separate context — important) reviews the diff and posts comments |
| **Guides** | this file (`AGENTS.md` ↔ `CLAUDE.md`) | Feedforward — what both Claudes read before acting |

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

---

<!--VITE PLUS START-->

# Using Vite+, the Unified Toolchain for the Web

This project is using Vite+, a unified toolchain built on top of Vite, Rolldown, Vitest, tsdown, Oxlint, Oxfmt, and Vite Task. Vite+ wraps runtime management, package management, and frontend tooling in a single global CLI called `vp`. Vite+ is distinct from Vite, but it invokes Vite through `vp dev` and `vp build`.

## Vite+ Workflow

`vp` is a global binary that handles the full development lifecycle. Run `vp help` to print a list of commands and `vp <command> --help` for information about a specific command.

### Start

- create - Create a new project from a template
- migrate - Migrate an existing project to Vite+
- config - Configure hooks and agent integration
- staged - Run linters on staged files
- install (`i`) - Install dependencies
- env - Manage Node.js versions

### Develop

- dev - Run the development server
- check - Run format, lint, and TypeScript type checks
- lint - Lint code
- fmt - Format code
- test - Run tests

### Execute

- run - Run monorepo tasks
- exec - Execute a command from local `node_modules/.bin`
- dlx - Execute a package binary without installing it as a dependency
- cache - Manage the task cache

### Build

- build - Build for production
- pack - Build libraries
- preview - Preview production build

### Manage Dependencies

Vite+ automatically detects and wraps the underlying package manager such as pnpm, npm, or Yarn through the `packageManager` field in `package.json` or package manager-specific lockfiles.

- add - Add packages to dependencies
- remove (`rm`, `un`, `uninstall`) - Remove packages from dependencies
- update (`up`) - Update packages to latest versions
- dedupe - Deduplicate dependencies
- outdated - Check for outdated packages
- list (`ls`) - List installed packages
- why (`explain`) - Show why a package is installed
- info (`view`, `show`) - View package information from the registry
- link (`ln`) / unlink - Manage local package links
- pm - Forward a command to the package manager

### Maintain

- upgrade - Update `vp` itself to the latest version

These commands map to their corresponding tools. For example, `vp dev --port 3000` runs Vite's dev server and works the same as Vite. `vp test` runs JavaScript tests through the bundled Vitest. The version of all tools can be checked using `vp --version`. This is useful when researching documentation, features, and bugs.

## Common Pitfalls

- **Using the package manager directly:** Do not use pnpm, npm, or Yarn directly. Vite+ can handle all package manager operations.
- **Always use Vite commands to run tools:** Don't attempt to run `vp vitest` or `vp oxlint`. They do not exist. Use `vp test` and `vp lint` instead.
- **Running scripts:** Vite+ built-in commands (`vp dev`, `vp build`, `vp test`, etc.) always run the Vite+ built-in tool, not any `package.json` script of the same name. To run a custom script that shares a name with a built-in command, use `vp run <script>`. For example, if you have a custom `dev` script that runs multiple services concurrently, run it with `vp run dev`, not `vp dev` (which always starts Vite's dev server).
- **Do not install Vitest, Oxlint, Oxfmt, or tsdown directly:** Vite+ wraps these tools. They must not be installed directly. You cannot upgrade these tools by installing their latest versions. Always use Vite+ commands.
- **Use Vite+ wrappers for one-off binaries:** Use `vp dlx` instead of package-manager-specific `dlx`/`npx` commands.
- **Import JavaScript modules from `vite-plus`:** Instead of importing from `vite` or `vitest`, all modules should be imported from the project's `vite-plus` dependency. For example, `import { defineConfig } from 'vite-plus';` or `import { expect, test, vi } from 'vite-plus/test';`. You must not install `vitest` to import test utilities.
- **Type-Aware Linting:** There is no need to install `oxlint-tsgolint`, `vp lint --type-aware` works out of the box.

## CI Integration

For GitHub Actions, consider using [`voidzero-dev/setup-vp`](https://github.com/voidzero-dev/setup-vp) to replace separate `actions/setup-node`, package-manager setup, cache, and install steps with a single action.

```yaml
- uses: voidzero-dev/setup-vp@v1
  with:
    cache: true
- run: vp check
- run: vp test
```

## Review Checklist for Agents

- [ ] Run `vp install` after pulling remote changes and before getting started.
- [ ] Run `vp check` and `vp test` to validate changes.
<!--VITE PLUS END-->
