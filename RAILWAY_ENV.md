# Railway Environment Variables

This document describes all environment variables needed for deploying Moltbot to Railway.

## Core Configuration

### `PORT`
- **Type**: Integer
- **Default**: `8080`
- **Description**: The port the Gateway HTTP server listens on. Railway will automatically set this based on the service configuration.

### `NODE_ENV`
- **Type**: String
- **Default**: `production`
- **Options**: `production`, `development`
- **Description**: Environment mode. Set to `production` for Railway deployments.

## Data Storage

### `CLAWDBOT_STATE_DIR`
- **Type**: String
- **Default**: `/data/.clawdbot`
- **Description**: Directory where Moltbot stores configuration, credentials, and state. Must be mounted to a persistent volume for data to survive restarts.
- **Railway**: Recommended to mount to `/data` persistent disk.

### `CLAWDBOT_WORKSPACE_DIR`
- **Type**: String
- **Default**: `/data/workspace`
- **Description**: Directory where workspace files (agents, skills, sessions) are stored. Mount to persistent volume.

## Gateway Authentication

### `CLAWDBOT_GATEWAY_TOKEN`
- **Type**: String
- **Auto-generated**: Yes (recommended)
- **Description**: Secure token for Gateway WebSocket authentication. Should be auto-generated in Railway.
- **Security**: Keep this value secret; store in Railway environment variables, not in code.

### `GATEWAY_PASSWORD`
- **Type**: String
- **Optional**: Yes
- **Description**: Alternative authentication using password instead of token.
- **Security**: If set, use a strong password.

## Model Configuration

### `ANTHROPIC_API_KEY`
- **Type**: String
- **Optional**: Yes
- **Description**: API key for Anthropic Claude models. Generate from https://console.anthropic.com/
- **Security**: Must be kept secret.

### `OPENAI_API_KEY`
- **Type**: String
- **Optional**: Yes
- **Description**: API key for OpenAI models. Generate from https://platform.openai.com/api-keys
- **Security**: Must be kept secret.

## Channel Configuration

### `TELEGRAM_BOT_TOKEN`
- **Type**: String
- **Optional**: Yes
- **Description**: Telegram bot token for Telegram channel integration.

### `SLACK_BOT_TOKEN`
- **Type**: String
- **Optional**: Yes
- **Description**: Slack bot token for Slack workspace integration.

### `SLACK_APP_TOKEN`
- **Type**: String
- **Optional**: Yes
- **Description**: Slack app token (socket mode) for Slack integration.

### `DISCORD_BOT_TOKEN`
- **Type**: String
- **Optional**: Yes
- **Description**: Discord bot token for Discord server integration.

### Twilio (WhatsApp via Twilio)
```
TWILIO_ACCOUNT_SID=ACxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
TWILIO_AUTH_TOKEN=your_auth_token_here
TWILIO_WHATSAPP_FROM=whatsapp:+17343367101
```

## Development & Debug

### `LOG_LEVEL`
- **Type**: String
- **Default**: `info`
- **Options**: `trace`, `debug`, `info`, `warn`, `error`
- **Description**: Logging verbosity level.

### `CLAWDBOT_DEBUG`
- **Type**: Boolean
- **Default**: `false`
- **Description**: Enable verbose debug logging.

## Railway-Specific Variables

These are set automatically by Railway:

- `RAILWAY_ENVIRONMENT_NAME` - Environment name from Railway
- `RAILWAY_DOMAIN` - Public domain assigned by Railway
- `RAILWAY_PUBLIC_DOMAIN` - Public domain name
- `RAILWAY_PRIVATE_DOMAIN` - Internal domain name

## Setup Instructions

1. **Create a new Railway project** or use an existing one
2. **Set required variables** in Railway dashboard:
   - `CLAWDBOT_GATEWAY_TOKEN` (auto-generate or provide)
   - `ANTHROPIC_API_KEY` or `OPENAI_API_KEY` (choose your LLM provider)
3. **Configure optional channels** as needed (Telegram, Discord, Slack, etc.)
4. **Mount persistent volume** at `/data` with at least 1GB
5. **Deploy** via Railway dashboard or CLI

## Recommended Railway Configuration

```json
{
  "services": [
    {
      "name": "moltbot-gateway",
      "runtime": "docker",
      "healthCheckPath": "/health",
      "environmentVariables": {
        "PORT": "8080",
        "NODE_ENV": "production",
        "CLAWDBOT_STATE_DIR": "/data/.clawdbot",
        "CLAWDBOT_WORKSPACE_DIR": "/data/workspace",
        "LOG_LEVEL": "info"
      },
      "volumes": [
        {
          "name": "moltbot-data",
          "mountPath": "/data",
          "sizeGB": 5
        }
      ]
    }
  ]
}
```

## Security Recommendations

1. **Never commit secrets** - Use Railway's environment variable management
2. **Use strong tokens** - Auto-generated tokens are recommended
3. **Enable HTTPS** - Railway provides free HTTPS for all deployments
4. **Restrict access** - Use `GATEWAY_PASSWORD` or gateway token for authentication
5. **Monitor logs** - Check Railway logs for any unauthorized access attempts
6. **Rotate tokens regularly** - Change sensitive tokens periodically
7. **Use API keys wisely** - Restrict API key permissions to minimum required

## Troubleshooting

### Gateway won't start
- Check `NODE_ENV` is set to `production`
- Verify `PORT` is set correctly
- See logs in Railway dashboard

### Data not persisting
- Ensure persistent volume is mounted at `/data`
- Check `CLAWDBOT_STATE_DIR` and `CLAWDBOT_WORKSPACE_DIR` point to `/data`

### Models not responding
- Verify `ANTHROPIC_API_KEY` or `OPENAI_API_KEY` is set
- Check API key is valid and has remaining quota
- View logs for authentication errors

### Channels not connecting
- Verify channel-specific tokens (Telegram, Discord, etc.)
- Check channel allowlist configuration in settings
- Review logs for connection errors
