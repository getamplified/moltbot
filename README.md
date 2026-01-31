# NiftyBot Railway Template (1‑click deploy)

This repo packages **NiftyBot** for Railway with a small **/setup** web wizard so users can deploy and onboard **without running any commands**.

## What you get

- **NiftyBot Gateway + Control UI** (served at `/` and `/moltbot`)
- A friendly **Setup Wizard** at `/setup` (protected by a password)
- Persistent state via **Railway Volume** (so config/credentials/memory survive redeploys)
- One-click **Export backup** (so users can migrate off Railway later)
 
## How it works
- The container runs a wrapper web server.
- **Auto-config**: If `OPENAI_API_KEY` is set in env (or `.env`), the system configures on boot—no `/setup` needed. Optionally set `TELEGRAM_BOT_TOKEN`, `SLACK_BOT_TOKEN`, `SLACK_APP_TOKEN` for channels.
- **Setup wizard**: If not auto-configured, the wrapper protects `/setup` with `SETUP_PASSWORD`. During setup, it runs `moltbot onboard` and configures channels.
- After setup, **`/` is NiftyBot**. The wrapper reverse-proxies all traffic (including WebSockets) to the local gateway process.

## Railway deploy instructions (what you’ll publish as a Template)

In Railway Template Composer:

1. Create a new template from this GitHub repo.
2. Add a **Volume** mounted at `/data`.
3. Set the following variables:

Required:

- `SETUP_PASSWORD` — user-provided password to access `/setup` (or other routes if not auto-configuring)

Auto-config (bypasses /setup):

- `OPENAI_API_KEY` — when set, system configures on boot (OpenAI API key)
- `TELEGRAM_BOT_TOKEN` — optional, enables Telegram channel
- `SLACK_BOT_TOKEN` — optional, enables Slack channel
- `SLACK_APP_TOKEN` — optional, for Slack Socket Mode

Recommended:

- `MOLTBOT_STATE_DIR=/data/.moltbot`
- `MOLTBOT_WORKSPACE_DIR=/data/workspace`

Optional:

- `MOLTBOT_GATEWAY_TOKEN` — if not set, the wrapper generates one (not ideal). In a template, set it using a generated secret.

Notes:

- This template pins the underlying Moltbot to a known-good version by default via Docker build arg `MOLTBOT_GIT_REF`.

4. Enable **Public Networking** (HTTP). Railway will assign a domain.
5. Deploy.

Then:

- If you set `OPENAI_API_KEY`: visit `https://<your-app>.up.railway.app/` directly (auto-configured).
- Otherwise: visit `https://<your-app>.up.railway.app/setup`, complete setup, then visit `/` and `/moltbot`.

## Getting chat tokens (so you don’t have to scramble)

### Telegram bot token

1. Open Telegram and message **@BotFather**
2. Run `/newbot` and follow the prompts
3. BotFather will give you a token that looks like: `123456789:AA...`
4. Set `TELEGRAM_BOT_TOKEN` in your `.env` or Railway variables (or paste into `/setup`)

### Discord bot token

1. Go to the Discord Developer Portal: https://discord.com/developers/applications
2. **New Application** → pick a name
3. Open the **Bot** tab → **Add Bot**
4. Copy the **Bot Token** and paste it into `/setup`
5. Invite the bot to your server (OAuth2 URL Generator → scopes: `bot`, `applications.commands`; then choose permissions)

## Local smoke test

```bash
docker build -t niftybot-railway-template .

docker run --rm -p 8080:8080 \
  -e PORT=8080 \
  -e SETUP_PASSWORD=test \
  -e MOLTBOT_STATE_DIR=/data/.moltbot \
  -e MOLTBOT_WORKSPACE_DIR=/data/workspace \
  -v $(pwd)/.tmpdata:/data \
  niftybot-railway-template

# With auto-config: add -e OPENAI_API_KEY=sk-... -e TELEGRAM_BOT_TOKEN=...
# Otherwise: open http://localhost:8080/setup (password: test)
```
