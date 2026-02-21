# claude-code-statusline

A real-time status line for [Claude Code](https://docs.anthropic.com/en/docs/claude-code) that displays git branch, model, context window usage, session rate limits, and weekly usage — right in your terminal.

## Preview

```
🌿 main │ 🤖 Sonnet 4.6 │ 🟢 Ctx: ▓▓░░░░░░ 34% │ ⏱ 46% → 19h00 (3h20m) │ 📅 ~59% (mon 14h)
```

## What it shows

| Segment | Description |
|---------|-------------|
| 🌿 `branch` | Current git branch |
| 🤖 `model` | Active Claude model (Sonnet 4.6, Opus 4.6, etc.) |
| 🟢/🟡/🔴 `Ctx: ▓░ XX%` | Context window usage with color-coded progress bar |
| ⏱ `XX% → HHhMM (XhYm)` | Session rate limit: % used, reset time, and countdown |
| 📅 `~XX% (day HHh)` | Weekly usage estimate with next reset |

## Requirements

- Linux or WSL (macOS: untested, contributions welcome)
- `bash`, `jq`, `tmux`, `python3`
- [Claude Code](https://docs.anthropic.com/en/docs/claude-code) CLI installed

## Install

### One-liner

```bash
curl -fsSL https://raw.githubusercontent.com/ohugonnot/claude-code-statusline/main/install.sh | bash
```

> Replace `ohugonnot` with your actual GitHub username after creating the repo.

### Manual

```bash
# Clone the repo
git clone https://github.com/ohugonnot/claude-code-statusline.git
cd claude-code-statusline

# Run the installer
bash install.sh
```

### Fully manual

```bash
# Copy the script
mkdir -p ~/.claude/hooks
cp statusline.sh ~/.claude/hooks/statusline.sh
chmod +x ~/.claude/hooks/statusline.sh

# Add to Claude Code settings (merge with existing settings.json)
# Add this key to ~/.claude/settings.json:
# "statusLine": { "type": "command", "command": "bash ~/.claude/hooks/statusline.sh" }
```

## Configuration

All settings are configurable via environment variables. Edit the top of `statusline.sh` or export them in your shell profile:

| Variable | Default | Description |
|----------|---------|-------------|
| `TIMEZONE` | *(system)* | Timezone for reset times (e.g. `America/New_York`) |
| `REFRESH_INTERVAL` | `300` | Seconds between usage scrapes |

## How it works

1. **Status line**: Claude Code calls the script on each render, passing session JSON via stdin. The script extracts model, context %, and git branch.

2. **Usage tracking**: Every 5 minutes (configurable), the script launches a background tmux session, opens Claude Code, runs `/usage`, parses the output, and caches the result in `~/.claude/usage-exact.json`.

3. **Display**: Cached usage data is read and formatted into the session rate limit (⏱) and weekly usage (📅) segments.

## Troubleshooting

**Status line not appearing?**
Check that `~/.claude/settings.json` contains the `statusLine` key and restart Claude Code.

**Usage segments (⏱ 📅) missing?**
Normal on first launch — the background scraper needs ~30 seconds to fetch data. Send a message and wait.

**Force a refresh:**
```bash
rm -f ~/.claude/usage-exact.json /tmp/claude-usage-refresh.lock
```

**Background scraper stuck:**
```bash
rm -f /tmp/claude-usage-refresh.lock
tmux kill-session -t claude-usage-bg 2>/dev/null
```

**Check cached data:**
```bash
cat ~/.claude/usage-exact.json | python3 -m json.tool
```

## Uninstall

```bash
rm -f ~/.claude/hooks/statusline.sh
rm -f ~/.claude/usage-exact.json
# Remove the "statusLine" key from ~/.claude/settings.json
```

## License

MIT
