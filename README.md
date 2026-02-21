# claude-code-statusline

**Know your Claude Code rate limits in real time.** No more guessing when your session or weekly quota resets — see your actual `/usage` data live in the status bar.

```
🌿 main │ 🤖 Sonnet 4.6 │ 🟢 Ctx: ▓▓░░░░░░ 34% │ ⏱ 46% → 19h00 (3h20m) │ 📅 ~59% (mon 14h)
```

## Why?

Claude Code has rate limits but no built-in way to see them while you work. The `/usage` command exists, but you have to stop what you're doing to check it manually.

This script **automatically scrapes `/usage` in the background** and displays the results directly in your status line — session rate limit, weekly quota, reset countdown, all updated every 5 minutes.

## What you get

| Segment | Description |
|---------|-------------|
| ⏱ `46% → 19h00 (3h20m)` | **Session rate limit** — % used, reset time, countdown until reset |
| 📅 `~59% (mon 14h)` | **Weekly quota** — % used, next weekly reset |
| 🟢/🟡/🔴 `Ctx: ▓░ 34%` | Context window usage with color-coded progress bar |
| 🤖 `Sonnet 4.6` | Active model |
| 🌿 `main` | Current git branch |

## How it works

Every 5 minutes, the script silently launches a **background tmux session**, opens Claude Code, runs `/usage`, parses the output, and caches the result. Your active session is never interrupted — the scraping happens in a completely separate process.

The cached data (`~/.claude/usage-exact.json`) is then read on each status line render to display up-to-date rate limit info.

## Install

### One-liner

```bash
curl -fsSL https://raw.githubusercontent.com/ohugonnot/claude-code-statusline/main/install.sh | bash
```

With custom refresh interval (e.g. every 60 seconds):

```bash
curl -fsSL https://raw.githubusercontent.com/ohugonnot/claude-code-statusline/main/install.sh | bash -s -- --refresh 60
```

### Manual

```bash
git clone https://github.com/ohugonnot/claude-code-statusline.git
cd claude-code-statusline
bash install.sh
```

### Fully manual

```bash
mkdir -p ~/.claude/hooks
cp statusline.sh ~/.claude/hooks/statusline.sh
chmod +x ~/.claude/hooks/statusline.sh

# Add this key to ~/.claude/settings.json:
# "statusLine": { "type": "command", "command": "bash ~/.claude/hooks/statusline.sh" }
```

## Requirements

- Linux or WSL (macOS: untested, contributions welcome)
- `bash`, `jq`, `tmux`, `python3`
- [Claude Code](https://docs.anthropic.com/en/docs/claude-code) CLI installed

## Configuration

Edit the top of `statusline.sh` or export in your shell profile:

| Variable | Default | Description |
|----------|---------|-------------|
| `TIMEZONE` | *(system)* | Timezone for reset times (e.g. `America/New_York`, `Europe/Paris`) |
| `REFRESH_INTERVAL` | `300` | Seconds between `/usage` scrapes |

## Troubleshooting

**Usage segments (⏱ 📅) missing?**
Normal on first launch — the background scraper needs ~30 seconds. Send a message and wait.

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
