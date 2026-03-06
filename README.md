# local-review

AI-powered local code review for git repos. Supports both Claude CLI and Copilot CLI as review backends.

Catches bugs, security issues, and logic errors before code hits a PR. Runs as a pre-push hook or standalone script.

## Developer setup (one-time)

```sh
git clone git@github.com:84codes/local-review.git ~/.local/share/local-review
~/.local/share/local-review/local-review setup
```

This symlinks `local-review` into `~/.local/bin`. Make sure `~/.local/bin` is on your PATH.

To update, `git pull` in `~/.local/share/local-review`.

## Repo setup

From the target repo:

```sh
local-review install
```

This creates thin wrappers (2-line scripts that call `local-review`) and default config files. Won't overwrite existing files (use `--force` to replace).

### What gets created

| File | Purpose |
|------|---------|
| `.githooks/pre-push` | Wrapper: `exec local-review hook` |
| `extras/review` | Wrapper: `exec local-review review` |
| `.claude/commands/local-review.md` | `/local-review` slash command for Claude Code |
| `.claude/review-criteria.md` | Review criteria (for Claude CLI + Claude Code) |
| `.github/copilot-instructions.md` | Review criteria (for @copilot PR reviewer) |
| `.claude/review-model` | Claude model ID (default: `claude-sonnet-4-6`) |

The wrappers are tiny and don't drift. All logic lives in the `local-review` command.

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

Or via the repo wrapper:

```sh
extras/review
extras/review 42
```

### Pre-push hook

Triggers automatically on `git push`. Asks before running. If issues are found, you can:

1. Push anyway
2. Abort
3. Address interactively (pipes findings into claude/copilot for follow-up)

Inside a Claude Code session, the hook emits the diff to stderr so Claude Code picks it up for interactive review.

### Claude Code (`/review`)

Inside Claude Code, use the slash command:

```
/local-review          # review local changes
/local-review 42       # review PR #42
```

This reviews in-session with full codebase access. Unlike the CLI path (which only sees the diff), Claude Code can read source files to verify findings before reporting them. Same criteria, better context.

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
| Review criteria | Edit `.claude/review-criteria.md` |
| Claude model | Edit `.claude/review-model` |
