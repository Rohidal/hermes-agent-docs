# Hermes Agent - Troubleshooting Guide & FAQ

Complete troubleshooting guide for common issues and frequently asked questions.

## Common Issues & Solutions

### Issue 1: Agent Not Responding to Telegram Messages

**Symptoms:**
- Bot doesn't reply to messages
- No response after sending `/start`
- Messages show as delivered but no reply

**Diagnosis:**

```bash
# Check if agent is running
ps aux | grep hermes

# Check logs
tail -f ~/.hermes/logs/hermes.log

# Test Telegram connection
hermes gateway test telegram
```

**Solutions:**

**A. Bot token is invalid:**

```bash
# Verify token in config
hermes config get gateways.telegram.bot_token

# Test token manually
curl "https://api.telegram.org/bot<YOUR_TOKEN>/getMe"

# If invalid, update config
hermes config set gateways.telegram.bot_token "NEW_TOKEN"
```

**B. Chat ID not in allowed_users:**

```yaml
# Edit ~/.hermes/config.yaml
gateways:
  telegram:
    allowed_users:
      - 123456789  # Add your chat ID here
```

**C. Agent process crashed:**

```bash
# Restart agent
hermes start

# Or restart systemd service
sudo systemctl restart hermes-agent

# Check for errors
sudo journalctl -u hermes-agent -n 50
```

**D. Network/firewall issues:**

```bash
# Test Telegram API connectivity
curl https://api.telegram.org/bot<TOKEN>/getUpdates

# Check firewall
sudo ufw status

# Allow outbound HTTPS
sudo ufw allow out 443/tcp
```

**E. Webhook conflicts:**

```bash
# Delete webhook if set
curl "https://api.telegram.org/bot<TOKEN>/deleteWebhook"

# Verify no webhook is set
curl "https://api.telegram.org/bot<TOKEN>/getWebhookInfo"
```

### Issue 2: Model API Errors

**Symptoms:**
- "API key invalid" errors
- "Rate limit exceeded"
- "Model not found"
- Slow or no responses

**Solutions:**

**A. Invalid API key:**

```bash
# Check current key
hermes config get model.api_key

# Update key
hermes config set model.api_key "sk-..."

# Test model connection
hermes model test
```

**B. Rate limit exceeded:**

```yaml
# Add rate limiting in config
model:
  rate_limit:
    requests_per_minute: 50
    tokens_per_minute: 100000
  
  # Add retry logic
  retry:
    max_attempts: 3
    backoff_factor: 2
```

**C. Model not available:**

```bash
# List available models
hermes model list

# Switch to different model
hermes config set model.model "anthropic/claude-sonnet-4"

# Or use local model
hermes config set model.provider "ollama"
hermes config set model.model "llama3.1:70b"
```

**D. Insufficient credits:**

```bash
# Check OpenRouter balance
curl https://openrouter.ai/api/v1/auth/key \
  -H "Authorization: Bearer YOUR_KEY"

# Check Anthropic usage
curl https://api.anthropic.com/v1/usage \
  -H "x-api-key: YOUR_KEY"
```

### Issue 3: Permission Denied Errors

**Symptoms:**
- "Permission denied" when accessing files
- "Operation not permitted"
- Credential files can't be read

**Solutions:**

**A. Fix file permissions:**

```bash
# Fix credentials directory
chmod 700 ~/.agent/credentials
chmod 600 ~/.agent/credentials/*

# Fix config file
chmod 600 ~/.hermes/config.yaml

# Fix scripts
chmod +x ~/.hermes/scripts/*.sh
chmod +x ~/.hermes/scripts/*.py
```

**B. Ownership issues:**

```bash
# Check ownership
ls -la ~/.hermes
ls -la ~/.agent

# Fix ownership
sudo chown -R $USER:$USER ~/.hermes
sudo chown -R $USER:$USER ~/.agent
```

**C. SELinux/AppArmor blocking:**

```bash
# Check SELinux status
getenforce

# Temporarily disable (for testing)
sudo setenforce 0

# Or add exception
sudo ausearch -c 'hermes' --raw | audit2allow -M hermes-agent
sudo semodule -i hermes-agent.pp
```

### Issue 4: Wallet Transaction Failures

**Symptoms:**
- "Insufficient funds" errors
- "Transaction failed"
- "Nonce too low/high"
- Gas estimation errors

**Solutions:**

**A. Check wallet balance:**

```bash
# Solana
solana balance YOUR_ADDRESS

# Ethereum
cast balance YOUR_ADDRESS --rpc-url $ETHEREUM_RPC

# Or ask agent
"Check my wallet balance on all chains"
```

**B. Gas price issues (EVM):**

```bash
# Check current gas price
cast gas-price --rpc-url $ETHEREUM_RPC

# Set higher gas price in transaction
"Swap 0.1 ETH to USDC with 50 gwei gas price"
```

**C. Nonce issues:**

```bash
# Check current nonce
cast nonce YOUR_ADDRESS --rpc-url $ETHEREUM_RPC

# Reset nonce in wallet state
rm ~/.agent/wallet/nonce_cache.json
```

**D. RPC endpoint issues:**

```bash
# Test RPC
curl -X POST $ETHEREUM_RPC \
  -H "Content-Type: application/json" \
  -d '{"jsonrpc":"2.0","method":"eth_blockNumber","params":[],"id":1}'

# Switch to backup RPC
hermes config set wallet.ethereum_rpc "https://eth.llamarpc.com"
```

**E. Slippage too low:**

```bash
# Increase slippage tolerance
"Swap 0.1 ETH to USDC with 2% slippage"
```

### Issue 5: Browser Automation Failures

**Symptoms:**
- "Browser not found"
- "Timeout waiting for element"
- "Navigation failed"
- Cookie/session expired

**Solutions:**

**A. Install browser dependencies:**

```bash
# Install Chromium
sudo apt-get install chromium-browser

# Or install Chrome
wget https://dl.google.com/linux/direct/google-chrome-stable_current_amd64.deb
sudo dpkg -i google-chrome-stable_current_amd64.deb
sudo apt-get install -f

# Install Playwright browsers
playwright install chromium
```

**B. Headless mode issues:**

```yaml
# Try non-headless mode for debugging
tools:
  browser:
    headless: false
    
# Or use different browser
tools:
  browser:
    browser_type: firefox  # or webkit
```

**C. Cookie/session expired:**

```bash
# Re-export cookies from browser
# Use "Cookie Editor" extension
# Save to ~/.agent/credentials/x-cookies.json

# Or re-authenticate
"Login to Twitter and save session"
```

**D. Element not found:**

```bash
# Increase timeout
"Navigate to example.com and wait 30 seconds for element"

# Or use different selector
# CSS selector, XPath, text content, etc.
```

**E. CAPTCHA blocking:**

```bash
# Use authenticated session (cookies)
# Or use API instead of browser automation
# Or integrate CAPTCHA solving service
```

### Issue 6: Memory/Context Issues

**Symptoms:**
- Agent forgets previous conversation
- "Context too long" errors
- Slow responses
- Repeated questions

**Solutions:**

**A. Context window full:**

```yaml
# Enable auto-compaction
context:
  max_tokens: 200000
  auto_compact: true
  compact_threshold: 0.8
```

**B. Memory not persisting:**

```bash
# Check memory file
ls -la ~/.hermes/memory.db

# Verify memory is enabled
hermes config get memory.enabled

# Manually save important info
"Remember: I prefer Python over JavaScript"
```

**C. Session not resuming:**

```bash
# Check session storage
ls -la ~/.hermes/sessions/

# Clear corrupted sessions
rm ~/.hermes/sessions/*.json

# Restart agent
hermes start
```

### Issue 7: Cron Job Not Running

**Symptoms:**
- Scheduled tasks don't execute
- No notifications from cron jobs
- Jobs stuck in "pending" state

**Solutions:**

**A. Check job status:**

```bash
# List all cron jobs
hermes cron list

# Check specific job
hermes cron status JOB_ID

# View job logs
hermes cron logs JOB_ID
```

**B. Invalid schedule:**

```bash
# Test cron expression
# Use: https://crontab.guru/

# Fix schedule
hermes cron update JOB_ID --schedule "0 9 * * *"
```

**C. Job disabled:**

```bash
# Enable job
hermes cron enable JOB_ID

# Or resume paused job
hermes cron resume JOB_ID
```

**D. Script errors:**

```bash
# Test script manually
bash ~/.hermes/scripts/your_script.sh

# Check script permissions
chmod +x ~/.hermes/scripts/your_script.sh

# View error logs
tail -f ~/.hermes/logs/cron_errors.log
```

**E. Notification not delivered:**

```yaml
# Check delivery config
cronjob:
  default_delivery: telegram  # or local, discord, etc.
  
# Test notification
hermes notify test "Test message"
```

### Issue 8: GitHub Integration Issues

**Symptoms:**
- "Authentication failed"
- "Repository not found"
- "Permission denied" on push
- PR creation fails

**Solutions:**

**A. Invalid GitHub token:**

```bash
# Test token
curl -H "Authorization: token YOUR_TOKEN" \
  https://api.github.com/user

# Generate new token with correct scopes:
# - repo (full control)
# - workflow
# - admin:org (if needed)

# Update token
echo "GITHUB_TOKEN=ghp_new_token" > ~/.agent/credentials/github-pat.env
```

**B. Repository access:**

```bash
# Check if repo exists
gh repo view owner/repo

# Clone with correct credentials
git clone https://YOUR_TOKEN@github.com/owner/repo.git

# Or use SSH
git clone git@github.com:owner/repo.git
```

**C. Push rejected:**

```bash
# Pull latest changes first
git pull origin main

# Force push (careful!)
git push --force-with-lease

# Or create new branch
git checkout -b feature-branch
git push -u origin feature-branch
```

**D. PR creation fails:**

```bash
# Check gh CLI auth
gh auth status

# Re-authenticate
gh auth login

# Create PR manually
gh pr create --title "Title" --body "Description"
```

### Issue 9: Discord Bot Issues

**Symptoms:**
- Bot offline in server
- Can't send messages
- "Missing permissions"
- Commands not working

**Solutions:**

**A. Bot token invalid:**

```bash
# Test token
curl -H "Authorization: Bot YOUR_TOKEN" \
  https://discord.com/api/v10/users/@me

# Get new token from Discord Developer Portal
# Update config
echo "DISCORD_TOKEN=new_token" > ~/.agent/credentials/discord-token.env
```

**B. Missing permissions:**

```
# Bot needs these permissions:
- Send Messages
- Read Messages/View Channels
- Embed Links
- Attach Files
- Read Message History
- Add Reactions

# Re-invite bot with correct permissions
# Use Discord Developer Portal → OAuth2 → URL Generator
```

**C. Intents not enabled:**

```
# Enable in Discord Developer Portal → Bot:
- Presence Intent
- Server Members Intent
- Message Content Intent (required!)
```

**D. Bot not in server:**

```bash
# Generate invite URL
https://discord.com/api/oauth2/authorize?client_id=YOUR_CLIENT_ID&permissions=8&scope=bot

# Or use Discord Developer Portal → OAuth2 → URL Generator
```

### Issue 10: Email Integration Issues

**Symptoms:**
- Can't send emails
- Not receiving emails
- "Authentication failed"
- "Connection timeout"

**Solutions:**

**A. Gmail app password:**

```bash
# Generate new app password:
# 1. Google Account → Security
# 2. 2-Step Verification (must be enabled)
# 3. App passwords → Generate

# Update credentials
cat > ~/.agent/credentials/email.env << 'EOF'
EMAIL=your-email@gmail.com
PASSWORD=your_16_char_app_password
IMAP_SERVER=imap.gmail.com
SMTP_SERVER=smtp.gmail.com
EOF
```

**B. IMAP/SMTP not enabled:**

```
# Enable IMAP in Gmail:
# Settings → Forwarding and POP/IMAP → Enable IMAP
```

**C. Firewall blocking:**

```bash
# Allow IMAP (993) and SMTP (587)
sudo ufw allow out 993/tcp
sudo ufw allow out 587/tcp

# Test connection
telnet imap.gmail.com 993
telnet smtp.gmail.com 587
```

**D. 2FA issues:**

```bash
# Use app password, not account password
# Or use OAuth2 (more complex setup)
```

## Performance Issues

### Slow Response Times

**Diagnosis:**

```bash
# Check system resources
htop

# Check network latency
ping api.anthropic.com
ping api.openrouter.ai

# Check disk I/O
iostat -x 1

# Check logs for bottlenecks
tail -f ~/.hermes/logs/hermes.log | grep -i "slow\|timeout"
```

**Solutions:**

**A. Optimize context:**

```yaml
context:
  max_tokens: 100000  # Reduce if too large
  auto_compact: true
  compact_threshold: 0.7  # Compact earlier
```

**B. Enable caching:**

```yaml
cache:
  enabled: true
  backend: redis
  ttl:
    web_search: 3600
    file_content: 1800
```

**C. Use faster model:**

```yaml
model:
  model: "anthropic/claude-haiku-4"  # Faster, cheaper
  # or
  model: "openai/gpt-4o-mini"
```

**D. Parallel execution:**

```yaml
execution:
  parallel:
    enabled: true
    max_workers: 4
```

**E. Local LLM for simple tasks:**

```yaml
# Use local model for routine tasks
model:
  provider: ollama
  model: llama3.1:8b  # Smaller, faster
  
  # Fallback to cloud for complex tasks
  fallback:
    provider: anthropic
    model: claude-sonnet-4
```

### High Memory Usage

**Solutions:**

```yaml
# Limit context size
context:
  max_tokens: 50000

# Disable unnecessary tools
tools:
  browser:
    enabled: false  # If not needed
  
# Clear cache regularly
cache:
  max_size_mb: 500
  eviction_policy: lru
```

```bash
# Restart agent daily
0 3 * * * systemctl restart hermes-agent
```

### High API Costs

**Solutions:**

```yaml
# Use cheaper models
model:
  model: "anthropic/claude-haiku-4"  # $0.25/$1.25 per 1M tokens
  # vs claude-sonnet-4: $3/$15 per 1M tokens

# Enable caching
cache:
  enabled: true

# Rate limiting
model:
  rate_limit:
    requests_per_minute: 20
    tokens_per_minute: 50000

# Use local models when possible
model:
  provider: ollama
  model: llama3.1:70b
  base_url: http://localhost:11434
```

## Security Issues

### Credentials Exposed

**If credentials are compromised:**

```bash
# 1. Immediately revoke all tokens
# - GitHub: Settings → Developer settings → Personal access tokens
# - OpenRouter: Dashboard → API Keys
# - Telegram: @BotFather → /revoke
# - Discord: Developer Portal → Bot → Regenerate Token

# 2. Generate new credentials
# Follow setup guides for each service

# 3. Update all credential files
rm -rf ~/.agent/credentials/*
# Re-create with new credentials

# 4. Rotate wallet if private key exposed
# CRITICAL: Move all funds to new wallet immediately!

# 5. Check for unauthorized access
# - GitHub: Settings → Security log
# - Discord: User Settings → Authorized Apps
# - Check wallet transactions on block explorer

# 6. Update .gitignore
cat >> .gitignore << 'EOF'
.hermes/config.yaml
.agent/credentials/
*.env
*-cookies.json
EOF

# 7. Scan git history for leaked secrets
git log -p | grep -i "password\|token\|key\|secret"

# 8. Use git-secrets to prevent future leaks
git secrets --install
git secrets --register-aws
```

### Unauthorized Access

**If agent is accessed by unauthorized user:**

```yaml
# Restrict to specific users
gateways:
  telegram:
    allowed_users:
      - 123456789  # Only your chat ID

# Enable IP whitelist
security:
  ip_whitelist:
    enabled: true
    allowed_ips:
      - "YOUR_IP"

# Require authentication
security:
  require_auth: true
  auth_token: "your_secret_token"
```

```bash
# Check access logs
grep "unauthorized" ~/.hermes/logs/hermes.log

# Block suspicious IPs
sudo ufw deny from SUSPICIOUS_IP
```

## FAQ

### General Questions

**Q: How much does it cost to run Hermes Agent?**

A: Costs depend on your model provider:
- **OpenRouter**: ~$0.50-5/day for moderate use (Claude Haiku/Sonnet)
- **Anthropic Direct**: Similar to OpenRouter
- **Local (Ollama)**: Free, but needs GPU (RTX 3090+ recommended)
- **Server**: $5-20/month for VPS (DigitalOcean, Linode, etc.)

**Q: Can I run multiple agents?**

A: Yes! Each agent needs:
- Separate config directory: `~/.hermes-agent1/`, `~/.hermes-agent2/`
- Different Telegram bot tokens
- Separate credential directories

```bash
# Start multiple agents
hermes start --config ~/.hermes-agent1/config.yaml
hermes start --config ~/.hermes-agent2/config.yaml
```

**Q: Does the agent work offline?**

A: Partially:
- ✅ Local LLM (Ollama) works offline
- ✅ File operations work offline
- ✅ Terminal commands work offline
- ❌ Web search requires internet
- ❌ Telegram requires internet
- ❌ API integrations require internet

**Q: How do I backup my agent?**

A: Backup these directories:
```bash
# Essential
~/.hermes/config.yaml
~/.hermes/SOUL.md
~/.agent/credentials/

# Optional (can be regenerated)
~/.hermes/memory.db
~/.hermes/skills/
~/.hermes/sessions/
```

**Q: Can I use this for commercial purposes?**

A: Yes, but:
- Check Hermes Agent license (Apache 2.0)
- Check your LLM provider's terms of service
- Ensure compliance with platform ToS (Twitter, Discord, etc.)
- Consider privacy/data protection laws

### Technical Questions

**Q: What's the difference between memory and skills?**

A:
- **Memory**: Facts about you, preferences, environment
  - "User prefers TypeScript"
  - "Project uses pytest"
- **Skills**: Reusable procedures and workflows
  - "How to deploy to production"
  - "How to create a GitHub PR"

**Q: How does autonomous operation work?**

A: Agent has autonomy rules:
- **Fully autonomous**: Safe operations (read files, web search, etc.)
- **Autonomous with logging**: Medium-risk operations (install packages)
- **Requires confirmation**: High-risk operations (delete files, spend money)

Configure in SOUL.md:
```markdown
## Fully Autonomous
- Web search and research
- Reading files
- Git operations (non-destructive)

## Requires Confirmation
- Deleting files
- Spending money
- Production deployments
```

**Q: Can the agent learn from mistakes?**

A: Yes, through:
1. **Memory**: Saves corrections and preferences
2. **Skills**: Updates procedures when they fail
3. **Session search**: Recalls past solutions

Example:
```
You: "Don't use npm, always use pnpm"
Agent: [Saves to memory]
[Next time automatically uses pnpm]
```

**Q: How secure is it?**

A: Security measures:
- ✅ Credentials stored locally, encrypted at rest
- ✅ No credentials sent to LLM provider
- ✅ File access restrictions
- ✅ Command whitelisting
- ✅ Rate limiting
- ⚠️ Depends on your server security
- ⚠️ Depends on credential management

**Q: What happens if the agent makes a mistake?**

A: Safety features:
- **Confirmation for risky operations**: Asks before destructive actions
- **Audit logging**: All actions logged
- **Rollback capabilities**: Git, backups, etc.
- **Rate limiting**: Prevents runaway operations
- **Timeout protection**: Long-running tasks auto-terminate

**Q: Can I extend the agent with custom tools?**

A: Yes! Three ways:
1. **Custom skills**: Markdown-based procedures
2. **Scripts**: Shell/Python scripts in `~/.hermes/scripts/`
3. **MCP servers**: Model Context Protocol integrations

Example custom tool:
```python
# ~/.hermes/tools/custom_tool.py
def my_custom_function(param):
    """Custom tool description"""
    # Your logic here
    return result
```

### Platform-Specific Questions

**Q: Why use Telegram instead of Discord/Slack?**

A: You can use any! Telegram is popular because:
- Easy bot setup (no OAuth)
- Good mobile app
- Fast message delivery
- Supports media files
- Free for personal use

But Discord, Slack, Matrix, etc. also work.

**Q: Can I use this with ChatGPT/GPT-4?**

A: Yes! Configure OpenAI provider:
```yaml
model:
  provider: openai
  model: gpt-4o
  api_key: sk-...
```

Or via OpenRouter:
```yaml
model:
  provider: openrouter
  model: openai/gpt-4o
  api_key: sk-or-...
```

**Q: Does it work on Windows?**

A: Partially:
- ✅ Core agent works
- ✅ Most tools work
- ⚠️ Some shell scripts need WSL
- ⚠️ Better on Linux/macOS

Recommended: Use WSL2 on Windows.

**Q: Can I run this on Raspberry Pi?**

A: Yes, but:
- ✅ Agent runs fine
- ⚠️ Use cloud LLM (no local GPU)
- ⚠️ Limited RAM (4GB+ recommended)
- ⚠️ Slower performance

**Q: How do I migrate to a new server?**

A:
```bash
# On old server
tar -czf hermes-backup.tar.gz ~/.hermes ~/.agent

# Transfer to new server
scp hermes-backup.tar.gz user@newserver:~

# On new server
tar -xzf hermes-backup.tar.gz
hermes start
```

## Getting Help

### Documentation
- **Official Docs**: https://hermes-agent.nousresearch.com/docs
- **GitHub**: https://github.com/NousResearch/hermes-agent
- **Examples**: https://github.com/NousResearch/hermes-examples

### Community
- **Discord**: Nous Research Discord server
- **GitHub Issues**: Report bugs and feature requests
- **Discussions**: Ask questions on GitHub Discussions

### Debugging Tips

**Enable debug logging:**
```yaml
logging:
  level: DEBUG
  outputs:
    - type: file
      path: ~/.hermes/logs/debug.log
```

**Verbose mode:**
```bash
hermes start --debug --verbose
```

**Test individual components:**
```bash
hermes gateway test telegram
hermes model test
hermes tools test terminal
hermes tools test browser
```

**Check system status:**
```bash
hermes status
hermes doctor  # Runs diagnostics
```

**Export logs for support:**
```bash
hermes logs export --output hermes-logs.zip
# Share this file when asking for help
```

---

**Still having issues? Open a GitHub issue with:**
1. Hermes version: `hermes --version`
2. OS and version: `uname -a`
3. Error logs: `~/.hermes/logs/hermes.log`
4. Steps to reproduce
5. Expected vs actual behavior
