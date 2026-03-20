# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Running the service

```bash
# Install dependencies
pip install -r requirements.txt

# Copy and configure environment variables
cp .env.example .env
# Edit .env with real Slack/Telegram credentials

# Run the monitor
python monitor.py
```

## Architecture

Three-module design:

- **`espn.py`** — All ESPN API interaction. `fetch_games()` polls the scoreboard and returns in-progress NCAA Tournament games in 2nd half or OT. `fetch_odds()` hits the per-game summary endpoint for live/closing odds (prefers DraftKings).
- **`monitor.py`** — Main loop. Polls ESPN every 30s, calls `check_and_notify()` per game. Alert thresholds: 5, 3, and 1 minute(s) remaining when score diff ≤ 6 points. Deduplication is in-memory (`sent` set keyed by `(game_id, period, threshold)`). On startup, skips already-passed thresholds to avoid spam.
- **`notify.py`** — Sends formatted messages to Slack (webhook) and/or Telegram (bot API). Channels are configured via env vars; absent vars silently skip that channel.

## Deployment

Deployed on Railway via Docker (`Dockerfile` runs `python monitor.py`). `railway.toml` has no healthcheck since this is a long-running worker with no HTTP server.

## Environment variables

| Variable | Purpose |
|---|---|
| `SLACK_WEBHOOK_URL` | Slack incoming webhook |
| `TELEGRAM_BOT_TOKEN` | Telegram bot token |
| `TELEGRAM_CHAT_ID` | Telegram chat/channel ID |

At least one notification channel must be configured for alerts to be delivered.
