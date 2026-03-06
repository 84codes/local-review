# local-review

AI-powered local code review for git repos. Supports both Claude CLI and Copilot CLI as review backends.

Catches bugs, security issues, and logic errors before code hits a PR. Runs as a pre-push hook or standalone script.

## Developer setup (one-time)

```sh
git clone git@github.com:84codes/local-review.git ~/.local/share/local-review
~/.local/share/local-review/local-review setup
```

This symlinks `local-review` into `~/.local/bin` and installs `/local-review` and `/local-review-fix` slash commands into `~/.claude/commands/`. Make sure `~/.local/bin` is on your PATH.

To update, `git pull` in `~/.local/share/local-review` and re-run `local-review setup`.

## Repo setup

From the target repo:

```sh
local-review install
```

This creates thin wrappers (2-line scripts that call `local-review`) and default config files. Won't overwrite existing files (use `--force` to replace).

### What gets created

| File | Purpose |
|------|---------|
| `.githooks/pre-push` | Wrapper: `exec local-review hook` (git needs a file) |
| `.claude/review-criteria.md` | Review criteria (for Claude CLI + Claude Code) |
| `.github/copilot-instructions.md` | Review criteria (for @copilot PR reviewer) |
| `.claude/review-model` | Claude model ID (default: `claude-sonnet-4-6`) |

All logic lives in the `local-review` command. Slash commands (`/local-review`, `/local-review-fix`) are installed globally by `local-review setup`.

### Hook manager integration

The installer detects existing hook managers:

| Manager | What happens |
|---------|--------------|
| **husky** (`.husky/` exists) | Creates/appends `.husky/pre-push` calling `local-review hook` |
| **overcommit** (`.overcommit.yml` exists) | Adds `PrePush: LocalReview` to `.overcommit.yml` |
| **None** | Creates `.githooks/pre-push` + sets `git config core.hooksPath .githooks` |

## Usage

### Standalone review

```sh
local-review review              # review local changes vs origin/main
local-review review 42           # review PR #42
local-review review --raw        # raw markdown output (no pager)
```

### Pre-push hook

Triggers automatically on `git push`. Asks before running. If issues are found, you can:

1. Push anyway
2. Abort
3. Address interactively (pipes findings into claude/copilot for follow-up)

Inside a Claude Code session, the hook emits the diff to stderr so Claude Code picks it up for interactive review.

### Review and fix loop

Automatically review, fix, and re-review until clean:

```sh
local-review review --fix              # review-fix loop (max 5 iterations)
local-review review --fix --max=3      # limit iterations
local-review review --fix 42           # checkout and fix a PR
```

Each iteration: review the diff, fix findings, re-review. Stops when "No issues found" or max iterations reached. Requires the claude backend.

### Claude Code (`/local-review`)

Inside Claude Code, use the slash commands:

```
/local-review          # review local changes (full codebase access)
/local-review 42       # review PR #42
/local-review-fix      # review-fix loop (auto-accept, no confirmation)
/local-review-fix 42   # checkout and fix a PR
```

The in-session commands are better than the CLI path: Claude Code can read source files to verify findings and make surgical edits.

## Backend selection

Supports `claude` and `copilot` as backends. Selection order:

1. **Environment variable**: `REVIEWER=copilot local-review review`
2. **Config file**: Create `.review-config` in repo root with `REVIEWER=claude`
3. **Auto-detect**: Uses whichever CLI is installed. Prefers `claude` if both are present.

### Claude backend

Pipes diff + criteria into `claude -p`. Model is read from `.claude/review-model`.

### Copilot backend

Pipes diff + criteria into `copilot` CLI. Falls back to `gh copilot` if the standalone binary isn't installed.

## Configuration

| Setting | How to configure |
|---------|-----------------|
| Review backend | `REVIEWER=claude` in `.review-config` or env var |
| Base branch | `REVIEW_BASE_BRANCH=develop` env var (default: `main`) |
| Review criteria | Edit `.claude/review-criteria.md` (built-in defaults used when absent) |
| Claude model | Edit `.claude/review-model` |
