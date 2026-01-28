# Moltbot Railway Deployment Guide

Complete guide to deploy Moltbot to Railway.

## Prerequisites

- Railway account at https://railway.app (free tier available)
- Railway CLI installed: `npm i -g @railway/cli`
- Git repository with Moltbot code
- Required API keys (Anthropic/OpenAI for LLM models)

## Quick Start (5 minutes)

### 1. Connect GitHub Repository
```bash
# Go to https://railway.app/dashboard
# Click "Create a new project"
# Select "Deploy from GitHub"
# Authorize Railway and select the moltbot repository
# Click "Deploy"
```

### 2. Configure Environment Variables
In Railway dashboard, go to **Variables** and add:

```
PORT=8080
NODE_ENV=production
CLAWDBOT_STATE_DIR=/data/.clawdbot
CLAWDBOT_WORKSPACE_DIR=/data/workspace
LOG_LEVEL=info

# Choose your LLM provider:
# Option A: Anthropic
ANTHROPIC_API_KEY=sk-ant-v0-... (from https://console.anthropic.com/)

# Option B: OpenAI
# OPENAI_API_KEY=sk-... (from https://platform.openai.com/api-keys)
```

The `CLAWDBOT_GATEWAY_TOKEN` will be auto-generated.

### 3. Create Persistent Storage
1. In Railway dashboard, go to the project
2. Click **Volumes**
3. Create a new volume:
   - Name: `moltbot-data`
   - Mount path: `/data`
   - Size: 5GB (recommended)

### 4. Deploy
Railway will automatically:
- Build the Docker image
- Install dependencies with `pnpm`
- Build the TypeScript
- Start the Gateway with `node dist/index.js`
- Expose via HTTPS

You'll see a public Railway domain (e.g., `https://moltbot-production.railway.app`)

## Detailed Setup

### Step 1: Repository Setup

```bash
# Clone/navigate to your moltbot repository
cd moltbot

# Ensure files are present
ls -la railway.json .railwayignore RAILWAY_ENV.md

# Commit deployment files
git add railway.json .railwayignore RAILWAY_ENV.md RAILWAY_DEPLOYMENT.md
git commit -m "chore: add Railway deployment configuration"
git push origin main
```

### Step 2: Railway Project Creation

#### Via Web Dashboard
1. Go to https://railway.app/dashboard
2. Click **+ New Project**
3. Select **Deploy from GitHub repo**
4. Connect GitHub account and authorize Railway
5. Select your moltbot repository
6. Choose the branch (usually `main`)
7. Click **Deploy**

#### Via Railway CLI
```bash
# Install Railway CLI
npm i -g @railway/cli

# Login to Railway
railway login

# Create new project
railway init

# Follow prompts and select your repo
```

### Step 3: Environment Configuration

**Via Dashboard:**
1. Go to your Railway project
2. Click the service (moltbot)
3. Go to **Variables** tab
4. Click **Raw Editor** and paste:

```
PORT=8080
NODE_ENV=production
CLAWDBOT_STATE_DIR=/data/.clawdbot
CLAWDBOT_WORKSPACE_DIR=/data/workspace
LOG_LEVEL=info
ANTHROPIC_API_KEY=your-key-here
```

**Via CLI:**
```bash
railway variable add PORT=8080
railway variable add NODE_ENV=production
railway variable add CLAWDBOT_STATE_DIR=/data/.clawdbot
railway variable add CLAWDBOT_WORKSPACE_DIR=/data/workspace
railway variable add LOG_LEVEL=info
railway variable add ANTHROPIC_API_KEY=your-key-here
```

### Step 4: Add Persistent Storage

1. In Railway dashboard, click your service
2. Go to **Volumes** tab
3. Click **+ New Volume**
4. Configure:
   - **Name**: `moltbot-data`
   - **Mount Path**: `/data`
   - **Size**: 5 GB

Data will persist across deployments.

### Step 5: Health Checks (Optional)

Railway can be configured for health checks (already in `railway.json`):
- Path: `/health`
- Checks if Gateway is healthy
- Automatically restarts on failure

### Step 6: Deploy!

Push to your main branch and Railway will automatically:
1. Clone the repository
2. Build the Docker image
3. Run `pnpm install --frozen-lockfile`
4. Run the build command from `Dockerfile`
5. Start the service with `node dist/index.js`

## Post-Deployment

### Access Your Gateway

Railway assigns a public URL like:
```
https://moltbot-production.railway.app
```

The Gateway WebSocket is available at:
```
wss://moltbot-production.railway.app
```

### Initial Setup

Once deployed, onboard your channels:

```bash
# Via CLI (if available in your location)
moltbot onboard

# Or via web dashboard at the Railway domain
# Then configure channels, models, etc.
```

### Connect Local CLI

To control the gateway from your local machine:

```bash
# Set environment to point to Railway
export CLAWDBOT_GATEWAY_URL=wss://moltbot-production.railway.app
export CLAWDBOT_GATEWAY_TOKEN=your-token-from-railway

# Test connection
moltbot agent --message "Hello from Railway!"
```

### View Logs

```bash
# Via Dashboard: Click your service → Logs tab

# Via CLI:
railway logs
railway logs --follow  # Real-time logs
```

### Monitor Performance

1. Go to **Metrics** tab in Railway dashboard
2. Monitor:
   - CPU usage
   - Memory usage
   - Network I/O
   - Response times

## Troubleshooting

### Service fails to start

**Check logs:**
```bash
railway logs --follow
```

**Common issues:**
- Missing `CLAWDBOT_STATE_DIR` volume mount
- Invalid API keys
- Port mismatch (should be 8080)

**Solution:**
1. Fix the issue
2. Go to **Deployments** tab
3. Click **Redeploy** on the latest successful deployment
4. Or push a commit to trigger auto-deploy

### Out of memory

If you see `SIGKILL` errors:
1. Increase plan size in Railway (pay plan)
2. Or optimize application memory usage
3. Check which features consuming most memory

### Data not persisting

1. Verify volume is mounted at `/data`
2. Check `CLAWDBOT_STATE_DIR=/data/.clawdbot`
3. Check `CLAWDBOT_WORKSPACE_DIR=/data/workspace`
4. View volume in Railway dashboard → Volumes

### Can't connect from local CLI

```bash
# Verify token
railway variable show CLAWDBOT_GATEWAY_TOKEN

# Test WebSocket
wscat -c "wss://moltbot-production.railway.app"

# Check CORS/authentication
curl -H "Authorization: Bearer YOUR_TOKEN" \
  https://moltbot-production.railway.app/health
```

## Scaling

### Vertical Scaling
- Click service → **Settings**
- Increase compute/memory tier
- Railway supports up to 8GB RAM and 4 CPU on paid plans

### Horizontal Scaling
Railway doesn't support multiple replicas for stateful apps like Moltbot (due to persistent gateway state). For HA:
1. Use the highest tier available
2. Enable auto-backups
3. Monitor disk space

## Backup Strategy

### Automatic Backups
Railway doesn't have built-in backup, so:

1. **Weekly Script Backup:**
```bash
#!/bin/bash
# backup.sh
BACKUP_DIR="/Volumes/Backup/moltbot-$(date +%Y%m%d)"
mkdir -p "$BACKUP_DIR"

# Download via Railway
tar -czf "$BACKUP_DIR/moltbot-state.tar.gz" \
  --exclude-from=.railwayignore .
```

2. **Store in S3/GCS:**
   - Use Railway volumes with automatic S3 sync
   - Or use GitHub Actions to backup states

### Restore from Backup

If you need to restore:
1. Delete the volume in Railway
2. Create a new volume at `/data`
3. Restore files from backup
4. Restart the service

## Advanced Configuration

### Custom Domain

1. Go to Railway **Settings** → **Domains**
2. Add your domain (e.g., `gateway.yourdomain.com`)
3. Update DNS records per Railway instructions
4. Update your local CLI config:

```bash
export CLAWDBOT_GATEWAY_URL=wss://gateway.yourdomain.com
```

### Environment-Specific Variables

For multiple environments (dev/staging/prod):

Use Railway **Environments** feature:
1. Create new environment (e.g., "staging")
2. Deploy from different branch
3. Each has separate variables and volumes

### Webhooks & Integrations

Configure webhook notifications:
1. Go to Railway **Settings** → **Integrations**
2. Add Discord/Slack webhooks
3. Get notified of deployments

## Security

### Secrets Management

✅ **Good:**
- Store API keys in Railway Variables (encrypted)
- Use environment-specific tokens
- Enable HTTPS (auto-enabled)

❌ **Avoid:**
- Committing secrets to Git
- Using test tokens in production
- Disabling HTTPS

### Access Control

1. Go to Railway **Settings** → **Members**
2. Invite team members with specific roles
3. Restrict who can deploy or change variables

### API Key Rotation

Periodically rotate your API keys:
1. Generate new key from provider (Anthropic/OpenAI)
2. Update in Railway Variables
3. Delete old key from provider
4. Monitor logs during transition

## Cost Optimization

### Railway Pricing

- **Free tier**: Up to $5 USD credit/month
  - Good for testing, dev deployments
  - Limited to 7GB storage total
  
- **Pay as you go**: After free credits
  - CPU: $0.0000463/cpu-minute
  - RAM: $0.0000463/gb-minute
  - Disk: $0.24/GB/month
  
### Tips to Save

1. **Start small**: Use `starter` plan initially
2. **Monitor usage**: Check Metrics tab regularly
3. **Scale only when needed**: Upgrade plan only if hitting limits
4. **Cache aggressively**: Reduce API calls to LLM providers
5. **Use spot instances**: (when available) for ~50% savings

## Next Steps

1. ✅ Deploy to Railway
2. ✅ Configure your first channel (WhatsApp/Telegram/Discord)
3. ✅ Set up your LLM model
4. ✅ Test with a message
5. ✅ Configure backup strategy
6. ✅ Set up monitoring/alerts

## Support & Resources

- **Railway Docs**: https://docs.railway.app
- **Moltbot Docs**: https://docs.molt.bot
- **Railway Discord**: https://discord.gg/railway
- **Moltbot Discord**: https://discord.gg/clawd

## Deployment Checklist

- [ ] Repository cloned and updated
- [ ] Railway account created
- [ ] GitHub connected to Railway
- [ ] Project created in Railway
- [ ] Persistent volume created at `/data` (5GB+)
- [ ] Environment variables set
- [ ] API key configured (ANTHROPIC_API_KEY or OPENAI_API_KEY)
- [ ] Deployment triggered and built successfully
- [ ] Logs checked - no errors
- [ ] Public URL accessible
- [ ] Health check passing
- [ ] Gateway token saved
- [ ] Backup strategy planned
- [ ] Monitoring configured

✨ Your Moltbot is now live on Railway!
