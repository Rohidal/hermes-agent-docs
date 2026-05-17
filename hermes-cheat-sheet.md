# Hermes Agent - Command Cheat Sheet

Quick reference untuk command-command penting. Print atau bookmark halaman ini!

## Installation & Setup

```bash
# Install (Linux / macOS / WSL2)
curl -fsSL https://raw.githubusercontent.com/NousResearch/hermes-agent/main/scripts/install.sh | bash

# Install (Windows PowerShell)
irm https://raw.githubusercontent.com/NousResearch/hermes-agent/main/scripts/install.ps1 | iex

# Setup wizard
hermes setup

# Check version
hermes --version

# Validate config
hermes config validate

# Test connections
hermes gateway test telegram
hermes model test
```

## Starting & Stopping

```bash
# Start (foreground)
hermes start

# Start with specific config
hermes start --config ~/.hermes/config.prod.yaml

# Start in debug mode
hermes start --debug --verbose

# Stop (if running in background)
pkill -f hermes

# Restart systemd service
sudo systemctl restart hermes-agent

# Check status
sudo systemctl status hermes-agent

# View logs
sudo journalctl -u hermes-agent -f
```

## Configuration

```bash
# View config
hermes config show

# Get specific value
hermes config get model.provider
hermes config get gateways.telegram.bot_token

# Set value
hermes config set model.provider anthropic
hermes config set model.api_key "sk-..."

# Edit config manually
nano ~/.hermes/config.yaml
```

## Telegram Bot Commands

Send these to your bot in Telegram:

```
/start              - Start conversation
/new                - New conversation (clear context)
/stop               - Stop current task
/help               - Show help
/status             - Agent status
/memory             - View memories
/skills             - List skills
```

## Common Agent Commands

```
# Information
"Who are you?"
"What can you do?"
"What's your status?"

# Web & Research
"Search for [topic]"
"Summarize this article: [URL]"
"What's trending on Twitter about [topic]?"

# Files & Code
"List files in [directory]"
"Read [filename]"
"Create a Python script that [does X]"
"Review this code: [paste code]"

# Git & GitHub
"Git status in [repo]"
"Create a new branch called [name]"
"Commit changes with message: [msg]"
"Push to GitHub"
"Create a PR for this feature"

# System
"Check disk space"
"What processes are running?"
"Monitor CPU usage"
"Check if [service] is running"

# Scheduling
"Run [command] every day at 9 AM"
"Check [X] every hour and notify me if [condition]"
"Remind me to [task] at [time]"
```

## Cron Jobs

```bash
# List all cron jobs
hermes cron list

# Create cron job
hermes cron create \
  --name "daily-backup" \
  --schedule "0 2 * * *" \
  --command "bash /path/to/backup.sh"

# View job details
hermes cron show JOB_ID

# View job logs
hermes cron logs JOB_ID

# Pause job
hermes cron pause JOB_ID

# Resume job
hermes cron resume JOB_ID

# Delete job
hermes cron delete JOB_ID

# Run job manually
hermes cron run JOB_ID
```

## Skills

```bash
# List skills
hermes skills list

# View skill
hermes skills view SKILL_NAME

# Create skill
hermes skills create my-skill

# Edit skill
nano ~/.hermes/skills/my-skill/SKILL.md

# Delete skill
hermes skills delete my-skill

# Reload skills
hermes skills reload
```

## Memory

```bash
# View all memories
hermes memory list

# Search memories
hermes memory search "keyword"

# Add memory
hermes memory add "User prefers Python over JavaScript"

# Delete memory
hermes memory delete "old fact"
```

## Logs

```bash
# View live logs
tail -f ~/.hermes/logs/hermes.log

# View last 100 lines
tail -n 100 ~/.hermes/logs/hermes.log

# Search logs
grep "error" ~/.hermes/logs/hermes.log

# View systemd logs
sudo journalctl -u hermes-agent -n 100

# Follow systemd logs
sudo journalctl -u hermes-agent -f

# Export logs
hermes logs export --output hermes-logs.zip
```

## Debugging

```bash
# Check system status
hermes status

# Run diagnostics
hermes doctor

# Test individual components
hermes test gateway telegram
hermes test model
hermes test tools terminal
hermes test tools browser

# Verbose output
hermes start --debug --verbose

# Check config syntax
hermes config validate

# View active sessions
hermes sessions list
```

## Backup & Restore

```bash
# Backup
tar -czf hermes-backup-$(date +%Y%m%d).tar.gz \
  ~/.hermes \
  ~/.agent/credentials

# Restore
tar -xzf hermes-backup-20260517.tar.gz -C ~/

# Backup to S3
aws s3 sync ~/.hermes s3://my-bucket/hermes-backup/

# Restore from S3
aws s3 sync s3://my-bucket/hermes-backup/ ~/.hermes
```

## Docker

```bash
# Build image
docker build -t hermes-agent .

# Run container
docker run -d \
  --name hermes \
  --restart unless-stopped \
  -v ~/.hermes:/root/.hermes \
  -v ~/.agent:/root/.agent \
  hermes-agent

# View logs
docker logs -f hermes

# Stop container
docker stop hermes

# Start container
docker start hermes

# Restart container
docker restart hermes

# Remove container
docker rm -f hermes

# Docker Compose
docker-compose up -d
docker-compose logs -f
docker-compose down
```

## Systemd Service

```bash
# Enable service
sudo systemctl enable hermes-agent

# Start service
sudo systemctl start hermes-agent

# Stop service
sudo systemctl stop hermes-agent

# Restart service
sudo systemctl restart hermes-agent

# Check status
sudo systemctl status hermes-agent

# View logs
sudo journalctl -u hermes-agent -f

# Reload systemd
sudo systemctl daemon-reload
```

## Monitoring

```bash
# Check resource usage
htop
top -p $(pgrep -f hermes)

# Memory usage
ps aux | grep hermes

# Disk usage
du -sh ~/.hermes
du -sh ~/.agent

# Network connections
netstat -tulpn | grep hermes
ss -tulpn | grep hermes

# Check open files
lsof -p $(pgrep -f hermes)
```

## Security

```bash
# Fix permissions
chmod 700 ~/.agent/credentials
chmod 600 ~/.agent/credentials/*
chmod 600 ~/.hermes/config.yaml

# Check for exposed secrets
grep -r "password\|token\|key" ~/.hermes/
git log -p | grep -i "password\|token\|key"

# Rotate credentials
# 1. Generate new tokens/keys
# 2. Update config
# 3. Restart agent
# 4. Revoke old tokens

# Firewall
sudo ufw status
sudo ufw allow 22/tcp
sudo ufw allow 443/tcp
sudo ufw enable
```

## Performance

```bash
# Check API usage
hermes stats api

# Check cache hit rate
hermes stats cache

# Clear cache
hermes cache clear

# Optimize database
sqlite3 ~/.hermes/memory.db "VACUUM;"

# Check context size
hermes context size

# Compact context
hermes context compact
```

## Troubleshooting Quick Fixes

```bash
# Agent not responding
sudo systemctl restart hermes-agent

# Permission denied
chmod 600 ~/.hermes/config.yaml
chmod 700 ~/.agent/credentials

# API errors
hermes model test
hermes config get model.api_key

# Telegram not working
hermes gateway test telegram
curl "https://api.telegram.org/bot<TOKEN>/getMe"

# High memory usage
hermes context compact
hermes cache clear
sudo systemctl restart hermes-agent

# Corrupted database
mv ~/.hermes/memory.db ~/.hermes/memory.db.backup
hermes start

# Reset everything (CAREFUL!)
rm -rf ~/.hermes
rm -rf ~/.agent
hermes setup
```

## Useful One-Liners

```bash
# Check if agent is running
pgrep -f hermes && echo "Running" || echo "Not running"

# Get agent PID
pgrep -f hermes

# Kill agent
pkill -f hermes

# Restart agent (systemd)
sudo systemctl restart hermes-agent && sudo systemctl status hermes-agent

# View last 50 errors
grep -i error ~/.hermes/logs/hermes.log | tail -50

# Count API calls today
grep "$(date +%Y-%m-%d)" ~/.hermes/logs/hermes.log | grep -c "API call"

# Check disk space
df -h | grep -E "/$|/home"

# Find large files
find ~/.hermes -type f -size +10M -exec ls -lh {} \;

# Backup config only
cp ~/.hermes/config.yaml ~/.hermes/config.yaml.backup

# Test Telegram bot
curl "https://api.telegram.org/bot<TOKEN>/getUpdates" | jq

# Get chat ID from Telegram
curl "https://api.telegram.org/bot<TOKEN>/getUpdates" | jq '.result[0].message.chat.id'

# Check SSL cert expiry
echo | openssl s_client -servername agent.yourdomain.com -connect agent.yourdomain.com:443 2>/dev/null | openssl x509 -noout -dates
```

## Environment Variables

```bash
# Set API key via env var
export ANTHROPIC_API_KEY="sk-ant-..."
export OPENROUTER_API_KEY="sk-or-..."

# Set Telegram token
export TELEGRAM_BOT_TOKEN="123456:ABC..."

# Set config path
export HERMES_CONFIG="~/.hermes/config.prod.yaml"

# Set log level
export HERMES_LOG_LEVEL="DEBUG"

# Load from .env file
source ~/.hermes/.env
```

## Cron Schedule Examples

```bash
# Every minute
* * * * *

# Every 5 minutes
*/5 * * * *

# Every hour
0 * * * *

# Every day at 9 AM
0 9 * * *

# Every Monday at 9 AM
0 9 * * 1

# Every 1st of month at 2 AM
0 2 1 * *

# Every weekday at 6 PM
0 18 * * 1-5

# Every 6 hours
0 */6 * * *
```

## Quick Config Templates

### Minimal Config
```yaml
agent:
  name: MyAgent

model:
  provider: openrouter
  model: anthropic/claude-sonnet-4
  api_key: sk-or-...

gateways:
  telegram:
    enabled: true
    bot_token: "123456:ABC..."
    home_chat_id: 123456789
    allowed_users: [123456789]
```

### Production Config
```yaml
agent:
  name: MyAgent

model:
  provider: anthropic
  model: claude-sonnet-4
  api_key: sk-ant-...
  rate_limit:
    requests_per_minute: 50

gateways:
  telegram:
    enabled: true
    bot_token: "123456:ABC..."
    home_chat_id: 123456789
    allowed_users: [123456789]

tools:
  web:
    enabled: true
  terminal:
    enabled: true
  file:
    enabled: true
  browser:
    enabled: true

cache:
  enabled: true
  backend: redis
  redis:
    host: localhost
    port: 6379

monitoring:
  enabled: true
  prometheus:
    enabled: true
    port: 9090

logging:
  level: INFO
  outputs:
    - type: file
      path: /var/log/hermes/agent.log
```

## Emergency Commands

```bash
# Agent consuming too much memory
sudo systemctl restart hermes-agent

# Agent stuck/frozen
pkill -9 -f hermes
sudo systemctl start hermes-agent

# Disk full
du -sh ~/.hermes/* | sort -h
rm -rf ~/.hermes/cache/*
rm -rf ~/.hermes/logs/*.log.old

# Database corrupted
mv ~/.hermes/memory.db ~/.hermes/memory.db.corrupt
sqlite3 ~/.hermes/memory.db.corrupt ".recover" | sqlite3 ~/.hermes/memory.db

# Config broken
cp ~/.hermes/config.yaml.backup ~/.hermes/config.yaml
# or
hermes setup

# Credentials compromised
# 1. Revoke all tokens immediately
# 2. Generate new credentials
# 3. Update config
# 4. Restart agent
# 5. Check logs for unauthorized access
```

## Useful Aliases

Add to `~/.bashrc`:

```bash
# Hermes shortcuts
alias hs='hermes start'
alias hst='hermes status'
alias hlog='tail -f ~/.hermes/logs/hermes.log'
alias hrestart='sudo systemctl restart hermes-agent'
alias hstatus='sudo systemctl status hermes-agent'

# Quick config edit
alias hconfig='nano ~/.hermes/config.yaml'
alias hsoul='nano ~/.hermes/SOUL.md'

# Logs
alias hlogs='sudo journalctl -u hermes-agent -f'
alias herrors='grep -i error ~/.hermes/logs/hermes.log | tail -20'

# Backup
alias hbackup='tar -czf ~/hermes-backup-$(date +%Y%m%d).tar.gz ~/.hermes ~/.agent'
```

---

**Print this page and keep it handy! 📋**

For detailed explanations, see the full guides:
- Quick Start: hermes-quick-start.md
- Setup: hermes-telegram-setup-guide.md
- Troubleshooting: hermes-troubleshooting-faq.md
