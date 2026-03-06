# local-review

AI-powered local code review for git repos. Supports both Claude CLI and Copilot CLI as review backends.

Catches bugs, security issues, and logic errors before code hits a PR. Runs as a pre-push hook or standalone script.

## Install

From your target repo:

```sh
path/to/local-review/install
```

This copies the hook and review script into your repo, sets up default review criteria, and configures `git core.hooksPath`. Won't overwrite existing files (use `--force` to replace).

Files installed:

| File | Purpose |
|------|---------|
| `.githooks/pre-push` | Pre-push hook that triggers review |
| `extras/review` | Standalone review script |
| `.claude/review-criteria.md` | Review criteria (for Claude CLI + Claude Code) |
| `.github/copilot-instructions.md` | Review criteria (for @copilot PR reviewer) |
| `.claude/review-model` | Claude model ID (default: `claude-sonnet-4-6`) |

## Usage

### Standalone review

```sh
extras/review              # review local changes vs origin/main
extras/review 42           # review PR #42
extras/review --raw        # raw markdown output (no pager)
```

### Pre-push hook

Triggers automatically on `git push`. Asks before running. If issues are found, you can:

1. Push anyway
2. Abort
3. Address interactively with Claude (claude backend only)

Inside a Claude Code session, the hook emits the diff to stderr so Claude Code picks it up for interactive review.

## Backend selection

The review script supports `claude` and `copilot` as backends. Selection order:

1. **Environment variable**: `REVIEWER=copilot extras/review`
2. **Config file**: Create `.review-config` in repo root with `REVIEWER=claude`
3. **Auto-detect**: Uses whichever CLI is installed. Prefers `claude` if both are present.

### Claude backend

Pipes diff + criteria into `claude -p`. Requires the [Claude CLI](https://docs.anthropic.com/en/docs/claude-code).

Model is read from `.claude/review-model` (one model ID per line, e.g. `claude-sonnet-4-6`).

### Copilot backend

Pipes diff + criteria into `copilot` CLI (standalone binary). Falls back to `gh copilot` if `copilot` isn't installed.

## Configuration

### Base branch

Default base branch is `main`. Override with `REVIEW_BASE_BRANCH`:

```sh
REVIEW_BASE_BRANCH=develop extras/review
```

### Review criteria

Edit `.claude/review-criteria.md` (or `.github/copilot-instructions.md`) to customize what the reviewer checks for. Both files start with the same default content. The `.github/copilot-instructions.md` file also feeds into @copilot's PR reviews on GitHub.

### Review model

Edit `.claude/review-model` to change the Claude model used for reviews (claude backend only).
