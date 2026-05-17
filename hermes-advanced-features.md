# Hermes Agent - Advanced Features & Configuration

Advanced guide for power users: multi-platform integration, autonomous workflows, custom skills, and production deployment.

## Part 1: Multi-Platform Integration

### 1.1 GitHub Integration

**Setup GitHub access:**

```bash
# Generate GitHub Personal Access Token
# Go to: https://github.com/settings/tokens
# Scopes needed: repo, workflow, admin:org, user

# Save token
mkdir -p ~/.agent/credentials
echo "GITHUB_TOKEN=ghp_your_token_here" > ~/.agent/credentials/github-pat.env
chmod 600 ~/.agent/credentials/github-pat.env
```

**Configure in config.yaml:**

```yaml
integrations:
  github:
    enabled: true
    token_file: ~/.agent/credentials/github-pat.env
    default_user: your-github-username
    auto_pr: true
    auto_branch: true
```

**Usage examples:**

```
"Create a new repo called 'my-project'"
"Open a PR for the feature I just pushed"
"List all open issues in my-org/my-repo"
"Review PR #123 in my-repo"
"Create an issue: Bug in login flow"
```

### 1.2 Discord Integration

**Setup Discord bot:**

1. Go to https://discord.com/developers/applications
2. Create New Application
3. Go to Bot section → Add Bot
4. Copy bot token
5. Enable these intents:
   - Presence Intent
   - Server Members Intent
   - Message Content Intent
6. Get bot invite URL from OAuth2 → URL Generator
   - Scopes: `bot`
   - Permissions: `Send Messages`, `Read Messages`, `Manage Channels`

**Save credentials:**

```bash
echo "DISCORD_TOKEN=your_bot_token_here" > ~/.agent/credentials/discord-token.env
chmod 600 ~/.agent/credentials/discord-token.env
```

**Configure:**

```yaml
gateways:
  discord:
    enabled: true
    token_file: ~/.agent/credentials/discord-token.env
    home_channel_id: "1234567890"  # Your default channel
    allowed_guilds:
      - "9876543210"  # Your server ID
```

### 1.3 Email Integration (Gmail)

**Setup Gmail app password:**

1. Go to Google Account → Security
2. Enable 2-Step Verification
3. Go to App Passwords
4. Generate password for "Mail"

**Save credentials:**

```bash
cat > ~/.agent/credentials/email.env << 'EOF'
EMAIL=your-email@gmail.com
PASSWORD=your_app_password_here
IMAP_SERVER=imap.gmail.com
SMTP_SERVER=smtp.gmail.com
EOF
chmod 600 ~/.agent/credentials/email.env
```

**Configure:**

```yaml
integrations:
  email:
    enabled: true
    credentials_file: ~/.agent/credentials/email.env
    auto_check: true
    check_interval: 300  # seconds
```

### 1.4 Twitter/X Integration

**Method A: Browser automation (recommended)**

```bash
# Login to X via browser and export cookies
# Use browser extension like "Cookie Editor"
# Export cookies to JSON

cat > ~/.agent/credentials/x-cookies.json << 'EOF'
[
  {
    "name": "auth_token",
    "value": "your_auth_token",
    "domain": ".x.com"
  },
  {
    "name": "ct0",
    "value": "your_csrf_token",
    "domain": ".x.com"
  }
]
EOF
chmod 600 ~/.agent/credentials/x-cookies.json
```

**Method B: Using xurl CLI**

```bash
# Install xurl
npm install -g xurl-cli

# Authenticate
xurl auth

# Test
xurl tweet "Hello from Hermes Agent!"
```

**Configure:**

```yaml
integrations:
  twitter:
    enabled: true
    method: browser  # or 'xurl'
    cookies_file: ~/.agent/credentials/x-cookies.json
    account: "@your_handle"
```

## Part 2: Autonomous Workflows

### 2.1 Scheduled Tasks (Cron Jobs)

**Create scheduled tasks via Telegram:**

```
"Run 'git pull' in /home/user/project every day at 9 AM"
"Check Bitcoin price every hour and notify if > $50k"
"Backup /home/user/documents to /backup every night at 2 AM"
"Post a tweet every day at 10 AM with AI-generated content"
```

**Manual cron job creation:**

```bash
# Via Hermes CLI
hermes cron create \
  --name "daily-backup" \
  --schedule "0 2 * * *" \
  --command "rsync -av /home/user/docs /backup/" \
  --notify telegram

# Via Python script
python3 << 'EOF'
from hermes_tools import cronjob

cronjob(
    action="create",
    name="bitcoin-monitor",
    schedule="0 * * * *",  # Every hour
    prompt="Check Bitcoin price and notify if > $50000",
    deliver="telegram"
)
EOF
```

**List and manage cron jobs:**

```
"List all my scheduled tasks"
"Pause the daily-backup job"
"Resume the bitcoin-monitor job"
"Delete the old-task job"
```

### 2.2 Event-Driven Automation

**Webhook subscriptions:**

```yaml
webhooks:
  enabled: true
  port: 8080
  endpoints:
    - path: /github
      source: github
      events:
        - push
        - pull_request
        - issues
      action: "Notify me about GitHub events"
    
    - path: /alerts
      source: custom
      action: "Process incoming alerts and take action"
```

**Usage:**

```bash
# GitHub webhook URL
https://your-server.com:8080/github

# Custom alert webhook
curl -X POST https://your-server.com:8080/alerts \
  -H "Content-Type: application/json" \
  -d '{"type": "disk_full", "server": "prod-1", "usage": 95}'
```

### 2.3 Monitoring & Alerts

**System monitoring:**

```bash
# Create monitoring script
cat > ~/.hermes/scripts/monitor_system.sh << 'EOF'
#!/bin/bash

# Check disk usage
DISK_USAGE=$(df -h / | awk 'NR==2 {print $5}' | sed 's/%//')
if [ $DISK_USAGE -gt 80 ]; then
    echo "⚠️ Disk usage: ${DISK_USAGE}%"
fi

# Check memory
MEM_USAGE=$(free | awk 'NR==2 {printf "%.0f", $3/$2 * 100}')
if [ $MEM_USAGE -gt 90 ]; then
    echo "⚠️ Memory usage: ${MEM_USAGE}%"
fi

# Check CPU load
CPU_LOAD=$(uptime | awk -F'load average:' '{print $2}' | awk '{print $1}' | sed 's/,//')
if (( $(echo "$CPU_LOAD > 4.0" | bc -l) )); then
    echo "⚠️ CPU load: ${CPU_LOAD}"
fi
EOF

chmod +x ~/.hermes/scripts/monitor_system.sh
```

**Schedule monitoring:**

```
"Run monitor_system.sh every 15 minutes and notify if there are warnings"
```

### 2.4 Auto-Response Workflows

**GitHub auto-response:**

```
"When someone opens an issue in my-repo, automatically:
1. Label it as 'needs-triage'
2. Reply with: 'Thanks for reporting! We'll review this soon.'
3. Notify me on Telegram"
```

**Discord auto-moderation:**

```
"In my Discord server:
- Delete messages containing spam keywords
- Warn users who post too many messages in 1 minute
- Notify me of any @everyone mentions"
```

## Part 3: Wallet & Crypto Integration

### 3.1 Multi-Chain Wallet Setup

**Generate wallet:**

```bash
# Install required tools
pip install web3 solana eth-account

# Generate wallet
python3 << 'EOF'
from eth_account import Account
from solders.keypair import Keypair
import secrets

# Generate Ethereum wallet
eth_account = Account.create()
print(f"EVM Address: {eth_account.address}")
print(f"EVM Private Key: {eth_account.key.hex()}")

# Generate Solana wallet
sol_keypair = Keypair()
print(f"\nSolana Address: {sol_keypair.pubkey()}")
print(f"Solana Private Key: {list(sol_keypair.secret())}")

# Generate seed phrase (BIP39)
from mnemonic import Mnemonic
mnemo = Mnemonic("english")
seed_phrase = mnemo.generate(strength=256)
print(f"\nSeed Phrase: {seed_phrase}")
EOF
```

**Save credentials securely:**

```bash
cat > ~/.agent/credentials/wallet.env << 'EOF'
# NEVER SHARE THIS FILE OR COMMIT TO GIT

# Seed phrase (BIP39)
SEED_PHRASE="your twelve or twenty four word seed phrase here"

# Solana
SOLANA_ADDRESS=YourSolanaAddressHere
SOLANA_PRIVATE_KEY=your_base58_private_key

# EVM chains (Ethereum, BSC, Polygon, etc.)
EVM_ADDRESS=0xYourEthereumAddressHere
EVM_PRIVATE_KEY=0xyour_private_key_hex

# RPC endpoints
ETHEREUM_RPC=https://eth-mainnet.g.alchemy.com/v2/YOUR_KEY
BSC_RPC=https://bsc-dataseed.binance.org/
POLYGON_RPC=https://polygon-rpc.com/
ARBITRUM_RPC=https://arb1.arbitrum.io/rpc
BASE_RPC=https://mainnet.base.org
OPTIMISM_RPC=https://mainnet.optimism.io
AVALANCHE_RPC=https://api.avax.network/ext/bc/C/rpc
SOLANA_RPC=https://api.mainnet-beta.solana.com
EOF

chmod 600 ~/.agent/credentials/wallet.env
```

**Configure wallet in config.yaml:**

```yaml
wallet:
  enabled: true
  credentials_file: ~/.agent/credentials/wallet.env
  
  # Autonomy rules
  autonomous_limit: 5  # USD
  require_confirmation_above: 5
  
  # Supported chains
  chains:
    - ethereum
    - bsc
    - polygon
    - arbitrum
    - base
    - optimism
    - avalanche
    - solana
  
  # Known addresses (no confirmation needed)
  known_addresses:
    - "0xYourOtherWalletAddress"
    - "YourOtherSolanaAddress"
```

### 3.2 Autonomous Trading & DeFi

**Token swaps:**

```
"Swap 0.1 ETH to USDC on Uniswap"
"Swap 10 USDC to SOL on Jupiter"
"Check my wallet balance on all chains"
```

**Bridge assets:**

```
"Bridge 100 USDC from Ethereum to Arbitrum"
"Bridge 0.5 ETH from Base to Optimism"
```

**Staking:**

```
"Stake 1 ETH on Lido"
"Unstake my SOL from Marinade"
"Check my staking rewards"
```

**Price monitoring:**

```
"Alert me when ETH drops below $3000"
"Notify when BTC/ETH ratio > 20"
"Track PEPE price and alert on 10% moves"
```

### 3.3 NFT Operations

```
"Mint NFT from contract 0x..."
"Check floor price of Bored Ape Yacht Club"
"List my NFT #1234 for 1 ETH on OpenSea"
"Buy cheapest Pudgy Penguin under 5 ETH"
```

## Part 4: Custom Skills Development

### 4.1 Create a Custom Skill

**Skill structure:**

```bash
mkdir -p ~/.hermes/skills/my-custom-skill
cd ~/.hermes/skills/my-custom-skill
```

**Create SKILL.md:**

```markdown
---
name: my-custom-skill
description: Custom workflow for my specific use case
version: 1.0.0
author: Your Name
tags: [automation, custom]
---

# My Custom Skill

## When to Use

Use this skill when you need to [describe use case].

## Prerequisites

- Tool X installed
- Access to service Y
- Credentials configured

## Steps

### 1. Preparation

Check prerequisites:
```bash
which tool-x
test -f ~/.credentials/service-y.env
```

### 2. Main Workflow

Execute the main task:
```bash
tool-x --option value
service-y-cli command
```

### 3. Verification

Verify the result:
```bash
tool-x status
service-y-cli check
```

## Pitfalls

- **Issue 1**: If X happens, do Y
- **Issue 2**: Common error Z means...

## Examples

### Example 1: Basic usage

```bash
hermes skill run my-custom-skill --param value
```

### Example 2: Advanced usage

```bash
hermes skill run my-custom-skill --advanced --param1 value1 --param2 value2
```

## Related Skills

- [other-skill](../other-skill/SKILL.md)
- [another-skill](../another-skill/SKILL.md)
```

**Add supporting files:**

```bash
# Scripts
mkdir -p scripts
cat > scripts/main.sh << 'EOF'
#!/bin/bash
# Main script for the skill
echo "Running custom skill..."
EOF
chmod +x scripts/main.sh

# Templates
mkdir -p templates
cat > templates/config.yaml << 'EOF'
# Template configuration
key: value
EOF

# References
mkdir -p references
cat > references/api-docs.md << 'EOF'
# API Documentation
Endpoint: https://api.example.com/v1
EOF
```

### 4.2 Use Custom Skills

**Load and run:**

```
"Load my-custom-skill and run it with param=value"
"Use my-custom-skill to process this data"
```

**Via CLI:**

```bash
hermes skill run my-custom-skill --param value
```

### 4.3 Share Skills

**Export skill:**

```bash
cd ~/.hermes/skills
tar -czf my-custom-skill.tar.gz my-custom-skill/
```

**Import skill:**

```bash
cd ~/.hermes/skills
tar -xzf my-custom-skill.tar.gz
hermes skill reload
```

## Part 5: Production Deployment

### 5.1 Docker Deployment

**Create Dockerfile:**

```dockerfile
FROM python:3.11-slim

# Install dependencies
RUN apt-get update && apt-get install -y \
    git \
    curl \
    build-essential \
    && rm -rf /var/lib/apt/lists/*

# Install Hermes Agent (official installer)
RUN curl -fsSL https://raw.githubusercontent.com/NousResearch/hermes-agent/main/scripts/install.sh | bash

# Create directories
RUN mkdir -p /root/.hermes /root/.agent/credentials

# Copy configuration
COPY config.yaml /root/.hermes/config.yaml
COPY SOUL.md /root/.hermes/SOUL.md

# Copy credentials (use secrets in production)
# COPY credentials/ /root/.agent/credentials/

# Expose webhook port
EXPOSE 8080

# Start agent
CMD ["hermes", "start"]
```

**Build and run:**

```bash
# Build image
docker build -t hermes-agent .

# Run container
docker run -d \
  --name hermes \
  --restart unless-stopped \
  -v ~/.hermes:/root/.hermes \
  -v ~/.agent:/root/.agent \
  -p 8080:8080 \
  hermes-agent

# View logs
docker logs -f hermes
```

**Docker Compose:**

```yaml
version: '3.8'

services:
  hermes:
    image: hermes-agent
    container_name: hermes-agent
    restart: unless-stopped
    volumes:
      - ~/.hermes:/root/.hermes
      - ~/.agent:/root/.agent
    ports:
      - "8080:8080"
    environment:
      - TZ=Asia/Jakarta
    logging:
      driver: "json-file"
      options:
        max-size: "10m"
        max-file: "3"
```

### 5.2 Kubernetes Deployment

**Create deployment:**

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: hermes-agent
spec:
  replicas: 1
  selector:
    matchLabels:
      app: hermes-agent
  template:
    metadata:
      labels:
        app: hermes-agent
    spec:
      containers:
      - name: hermes
        image: hermes-agent:latest
        ports:
        - containerPort: 8080
        volumeMounts:
        - name: config
          mountPath: /root/.hermes
        - name: credentials
          mountPath: /root/.agent/credentials
          readOnly: true
        resources:
          requests:
            memory: "512Mi"
            cpu: "500m"
          limits:
            memory: "2Gi"
            cpu: "2000m"
      volumes:
      - name: config
        configMap:
          name: hermes-config
      - name: credentials
        secret:
          secretName: hermes-credentials
---
apiVersion: v1
kind: Service
metadata:
  name: hermes-agent
spec:
  selector:
    app: hermes-agent
  ports:
  - port: 8080
    targetPort: 8080
  type: ClusterIP
```

**Create secrets:**

```bash
# Create secret from credentials directory
kubectl create secret generic hermes-credentials \
  --from-file=~/.agent/credentials/

# Create configmap from config
kubectl create configmap hermes-config \
  --from-file=config.yaml=~/.hermes/config.yaml \
  --from-file=SOUL.md=~/.hermes/SOUL.md
```

### 5.3 High Availability Setup

**Load balancer configuration:**

```yaml
# Multiple agent instances
services:
  hermes-1:
    image: hermes-agent
    # ... config ...
  
  hermes-2:
    image: hermes-agent
    # ... config ...
  
  hermes-3:
    image: hermes-agent
    # ... config ...
  
  nginx:
    image: nginx:alpine
    ports:
      - "80:80"
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf
```

**nginx.conf:**

```nginx
upstream hermes_backend {
    least_conn;
    server hermes-1:8080;
    server hermes-2:8080;
    server hermes-3:8080;
}

server {
    listen 80;
    
    location / {
        proxy_pass http://hermes_backend;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}
```

### 5.4 Monitoring & Observability

**Prometheus metrics:**

```yaml
# config.yaml
monitoring:
  enabled: true
  prometheus:
    enabled: true
    port: 9090
    metrics:
      - requests_total
      - requests_duration
      - errors_total
      - active_sessions
```

**Grafana dashboard:**

```bash
# Import Hermes dashboard
curl -X POST http://grafana:3000/api/dashboards/db \
  -H "Content-Type: application/json" \
  -d @hermes-dashboard.json
```

**Logging:**

```yaml
logging:
  level: INFO
  format: json
  outputs:
    - type: file
      path: /var/log/hermes/agent.log
      rotation: daily
      retention: 7
    - type: syslog
      host: logs.example.com
      port: 514
```

## Part 6: Security Hardening

### 6.1 Credential Management

**Use environment variables:**

```bash
# .env file (never commit)
export TELEGRAM_BOT_TOKEN="..."
export ANTHROPIC_API_KEY="..."
export GITHUB_TOKEN="..."

# Load in config
source .env
hermes start
```

**Use secrets manager:**

```bash
# AWS Secrets Manager
aws secretsmanager get-secret-value \
  --secret-id hermes/credentials \
  --query SecretString \
  --output text > ~/.agent/credentials/secrets.json

# HashiCorp Vault
vault kv get -format=json secret/hermes/credentials \
  | jq -r '.data.data' > ~/.agent/credentials/secrets.json
```

### 6.2 Network Security

**Firewall rules:**

```bash
# Allow only necessary ports
ufw default deny incoming
ufw default allow outgoing
ufw allow 22/tcp    # SSH
ufw allow 8080/tcp  # Webhook port
ufw enable
```

**Reverse proxy with SSL:**

```nginx
server {
    listen 443 ssl http2;
    server_name agent.example.com;
    
    ssl_certificate /etc/letsencrypt/live/agent.example.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/agent.example.com/privkey.pem;
    
    location / {
        proxy_pass http://localhost:8080;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

### 6.3 Access Control

**IP whitelist:**

```yaml
security:
  ip_whitelist:
    enabled: true
    allowed_ips:
      - "1.2.3.4"      # Your IP
      - "5.6.7.0/24"   # Your network
```

**Rate limiting:**

```yaml
security:
  rate_limit:
    enabled: true
    max_requests: 100
    window_seconds: 3600
    per_user: true
```

### 6.4 Audit Logging

```yaml
audit:
  enabled: true
  log_file: /var/log/hermes/audit.log
  events:
    - command_execution
    - file_access
    - network_request
    - credential_access
    - configuration_change
```

## Part 7: Performance Optimization

### 7.1 Context Management

```yaml
context:
  max_tokens: 200000
  auto_compact: true
  compact_threshold: 0.8
  preserve_recent: 10000
```

### 7.2 Caching

```yaml
cache:
  enabled: true
  backend: redis
  redis:
    host: localhost
    port: 6379
    db: 0
  ttl:
    web_search: 3600
    file_content: 1800
    api_response: 600
```

### 7.3 Parallel Execution

```yaml
execution:
  parallel:
    enabled: true
    max_workers: 4
  timeout:
    default: 300
    long_running: 3600
```

## Part 8: Backup & Recovery

### 8.1 Backup Configuration

```bash
# Backup script
cat > ~/.hermes/scripts/backup.sh << 'EOF'
#!/bin/bash
BACKUP_DIR="/backup/hermes/$(date +%Y%m%d)"
mkdir -p "$BACKUP_DIR"

# Backup config
cp -r ~/.hermes "$BACKUP_DIR/"

# Backup credentials (encrypted)
tar -czf - ~/.agent/credentials | \
  openssl enc -aes-256-cbc -salt -out "$BACKUP_DIR/credentials.tar.gz.enc"

# Backup database
sqlite3 ~/.hermes/memory.db ".backup '$BACKUP_DIR/memory.db'"

echo "Backup completed: $BACKUP_DIR"
EOF

chmod +x ~/.hermes/scripts/backup.sh
```

**Schedule backups:**

```
"Run backup.sh every day at 3 AM"
```

### 8.2 Disaster Recovery

```bash
# Restore from backup
BACKUP_DIR="/backup/hermes/20260517"

# Restore config
cp -r "$BACKUP_DIR/.hermes" ~/

# Restore credentials
openssl enc -aes-256-cbc -d -in "$BACKUP_DIR/credentials.tar.gz.enc" | \
  tar -xzf - -C ~/

# Restore database
cp "$BACKUP_DIR/memory.db" ~/.hermes/memory.db

# Restart agent
systemctl restart hermes-agent
```

## Resources

- **Official Docs**: https://hermes-agent.nousresearch.com/docs
- **GitHub**: https://github.com/NousResearch/hermes-agent
- **Discord**: Nous Research Community
- **Examples**: https://github.com/NousResearch/hermes-examples

---

**You now have a production-ready Hermes Agent setup with advanced features, security, and monitoring.**
