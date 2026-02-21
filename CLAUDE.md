# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Real-time status line for Claude Code that displays rate limit usage in the IDE status bar. Scrapes `/usage` in a background tmux session every 10 minutes, caches results to JSON, and renders color-coded progress bars.

**Stack**: Bash, jq, tmux, python3 (for parsing). No build step, no external test framework.

## Commands

```bash
# Run tests
bash test_statusline.sh

# Manual test (pipe JSON to statusline)
echo '{"model":"claude-sonnet-4-6","context_window":{"used_percentage":42}}' | bash statusline.sh

# Install locally
bash install.sh
bash install.sh --refresh 300  # custom interval
```

## Architecture

Three files, single-purpose each:

- **statusline.sh** (core) — Claude Code status line hook. Reads JSON from stdin (model, context_window, workspace), outputs a formatted status string. Manages background scraper lifecycle.
- **install.sh** — Copies `statusline.sh` to `~/.claude/hooks/`, updates `~/.claude/settings.json`, checks/installs dependencies.
- **test_statusline.sh** — Unit + integration tests with simple assert helpers (`assert_eq`, `assert_contains`, `assert_not_contains`).

### Data Flow

```
Claude Code → JSON stdin → statusline.sh → formatted status string
                              ↓ (async, if cache stale)
                         background tmux scraper → /usage → python3 parser → ~/.claude/usage-exact.json
```

### Key Design Decisions

- **Detached scraper**: Hook processes get killed by Claude Code, so the scraper uses `setsid nohup` + tmux to survive. Lock file + tmux session check prevent duplicates.
- **120s global timeout**: Watchdog kills scraper to prevent zombie tmux sessions.
- **Cross-platform**: GNU stat (Linux) vs BSD stat (macOS) detection in `file_mtime()`. Avoids `grep -P` (not available on macOS).
- **Staleness indicator**: ⚠ appended when cache age > 2× REFRESH_INTERVAL.

### Configuration (env vars)

| Variable | Default | Notes |
|----------|---------|-------|
| `TIMEZONE` | system | Override for display (e.g. `America/New_York`) |
| `REFRESH_INTERVAL` | `600` | Seconds between scrapes |
| `USAGE_FILE` | `~/.claude/usage-exact.json` | Cache location |
| `LOCK_FILE` | `/tmp/claude-usage-refresh.lock` | Scraper coordination |
| `TMUX_SESSION` | `claude-usage-bg` | Background session name |

## Testing Patterns

Tests extract `make_bar()` via awk and eval it for unit testing. Integration tests pipe JSON through `statusline.sh` with overridden env vars (`USAGE_FILE`, `REFRESH_INTERVAL`) to control behavior without triggering the real scraper. Temp files are tracked in `TMPFILES` array and cleaned via trap.

To add a test: create a temp JSON cache file, use `run_statusline` helper with appropriate env overrides, assert on stdout.
