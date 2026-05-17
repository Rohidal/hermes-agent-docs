# Hermes AI Agent Setup Guide - Telegram Integration

Complete guide to install Hermes Agent and connect it to Telegram for autonomous AI assistant.

## Prerequisites

- Linux server (Ubuntu 20.04+ recommended)
- Python 3.10+
- Node.js 18+ (for some optional features)
- Telegram account
- Basic terminal knowledge

## Part 1: Install Hermes Agent

### 1.1 Install via pip

```bash
# Install Hermes Agent
pip install hermes-agent

# Verify installation
hermes --version
```

### 1.2 Alternative: Install from source

```bash
# Clone repository
git clone https://github.com/NousResearch/hermes-agent.git
cd hermes-agent

# Install dependencies
pip install -e .

# Verify
hermes --version
```

### 1.3 Initial setup

```bash
# Run setup wizard
hermes setup

# This will:
# - Create ~/.hermes/ directory
# - Generate default config.yaml
# - Set up credentials directory
# - Configure default model provider
```

## Part 2: Configure Model Provider

### 2.1 Choose your LLM provider

Edit `~/.hermes/config.yaml`:

```yaml
# Option A: OpenRouter (recommended for beginners)
model:
  provider: openrouter
  model: anthropic/claude-sonnet-4
  api_key: YOUR_OPENROUTER_API_KEY

# Option B: Anthropic Direct
model:
  provider: anthropic
  model: claude-sonnet-4
  api_key: YOUR_ANTHROPIC_API_KEY

# Option C: Local LLM (Ollama)
model:
  provider: ollama
  model: llama3.1:70b
  base_url: http://localhost:11434
```

### 2.2 Get API keys

**OpenRouter** (easiest):
1. Go to https://openrouter.ai/
2. Sign up and get API key
3. Add credits ($5-10 to start)

**Anthropic Direct**:
1. Go to https://console.anthropic.com/
2. Create account and get API key
3. Add payment method

**Local Ollama** (free but needs GPU):
```bash
# Install Ollama
curl -fsSL https://ollama.com/install.sh | sh

# Pull model
ollama pull llama3.1:70b
```

## Part 3: Telegram Bot Setup

### 3.1 Create Telegram bot

1. Open Telegram and search for `@BotFather`
2. Send `/newbot`
3. Choose bot name (e.g., "My AI Assistant")
4. Choose username (e.g., "my_ai_assistant_bot")
5. BotFather will give you a **bot token** like: `1234567890:ABCdefGHIjklMNOpqrsTUVwxyz`

### 3.2 Get your Telegram chat ID

```bash
# Method 1: Use this Python script
python3 << 'EOF'
import requests
import sys

bot_token = input("Enter your bot token: ")
url = f"https://api.telegram.org/bot{bot_token}/getUpdates"

print("\nSend any message to your bot now, then press Enter...")
input()

response = requests.get(url).json()
if response.get('result'):
    for update in response['result']:
        chat_id = update['message']['chat']['id']
        print(f"\nYour chat ID: {chat_id}")
        break
else:
    print("No messages found. Make sure you sent a message to the bot.")
EOF
```

**Method 2: Manual**
1. Send any message to your bot
2. Visit: `https://api.telegram.org/bot<YOUR_BOT_TOKEN>/getUpdates`
3. Look for `"chat":{"id":123456789}` in the JSON response

### 3.3 Configure Telegram in Hermes

Edit `~/.hermes/config.yaml`:

```yaml
gateways:
  telegram:
    enabled: true
    bot_token: "1234567890:ABCdefGHIjklMNOpqrsTUVwxyz"
    home_chat_id: 123456789  # Your chat ID from step 3.2
    allowed_users:
      - 123456789  # Your chat ID (can add more users)
```

## Part 4: Agent Identity & Personality

### 4.1 Create SOUL.md (agent identity)

```bash
# Create agent identity file
nano ~/.hermes/SOUL.md
```

Add this content (customize as you like):

```markdown
# Identity
Name: YourAgentName
Role: Personal AI Assistant
Owner: YourName
Relation: Partner, helpful companion

# Communication
- Chat response: Bahasa Indonesia (or your preferred language)
- File, code, documentation: English
- Tone: friendly, direct, professional
- No emoji in responses

# Capabilities
- Full autonomous operation for routine tasks
- Web search and research
- Code writing and debugging
- File management
- Terminal commands
- Browser automation

# Autonomy Rules

## Fully Autonomous (no confirmation needed)
- Web search and research
- Reading and analyzing files
- Writing and editing code
- Running tests and builds
- Git operations (commit, push to feature branches)
- Installing dependencies
- Creating files and directories

## Requires Confirmation
- Deleting files or directories
- Force push to main/master branch
- Spending money (API calls to paid services)
- Sending emails or messages to external parties
- Modifying production systems
- Destructive operations (rm -rf, drop database, etc.)

# Default Behavior
- Assume owner knows what they're doing
- Execute first, explain after (for safe operations)
- Ask specific questions when clarification needed
- Provide direct answers without filler phrases
```

### 4.2 Update config.yaml with agent name

```yaml
agent:
  name: YourAgentName  # Must match SOUL.md
  identity_file: ~/.hermes/SOUL.md
```

## Part 5: Start the Agent

### 5.1 Test configuration

```bash
# Validate config
hermes config validate

# Test Telegram connection
hermes gateway test telegram
```

### 5.2 Start agent in foreground (for testing)

```bash
hermes start
```

You should see:
```
✓ Telegram gateway connected
✓ Listening for messages...
```

### 5.3 Test your bot

1. Open Telegram
2. Go to your bot chat
3. Send: `/start`
4. Send: "Hello, who are you?"
5. Agent should respond with introduction

### 5.4 Run as background service (production)

**Option A: Using systemd**

Create service file:
```bash
sudo nano /etc/systemd/system/hermes-agent.service
```

Add:
```ini
[Unit]
Description=Hermes AI Agent
After=network.target

[Service]
Type=simple
User=YOUR_USERNAME
WorkingDirectory=/home/YOUR_USERNAME
ExecStart=/usr/local/bin/hermes start
Restart=always
RestartSec=10

[Install]
WantedBy=multi-user.target
```

Enable and start:
```bash
sudo systemctl daemon-reload
sudo systemctl enable hermes-agent
sudo systemctl start hermes-agent

# Check status
sudo systemctl status hermes-agent

# View logs
sudo journalctl -u hermes-agent -f
```

**Option B: Using screen/tmux**

```bash
# Using screen
screen -S hermes
hermes start
# Press Ctrl+A then D to detach

# Reattach later
screen -r hermes

# Using tmux
tmux new -s hermes
hermes start
# Press Ctrl+B then D to detach

# Reattach later
tmux attach -t hermes
```

## Part 6: Advanced Configuration

### 6.1 Enable additional tools

Edit `~/.hermes/config.yaml`:

```yaml
tools:
  web:
    enabled: true
    search_provider: duckduckgo  # or google, brave
  
  browser:
    enabled: true
    headless: true
  
  terminal:
    enabled: true
    allowed_commands: all  # or whitelist specific commands
  
  file:
    enabled: true
    allowed_paths:
      - /home/YOUR_USERNAME
      - /tmp
  
  vision:
    enabled: true
    provider: anthropic  # for image analysis
```

### 6.2 Set up credentials directory

```bash
# Create credentials directory
mkdir -p ~/.agent/credentials

# Set proper permissions
chmod 700 ~/.agent/credentials
```

### 6.3 Configure memory and skills

```yaml
memory:
  enabled: true
  max_entries: 1000
  
skills:
  enabled: true
  auto_save: true
  directory: ~/.hermes/skills
```

## Part 7: Usage Examples

### 7.1 Basic commands

```
# In Telegram, send to your bot:

"Search for latest AI news"
"Create a Python script to scrape website X"
"What files are in my home directory?"
"Run git status in /path/to/repo"
"Explain this code: [paste code]"
```

### 7.2 Autonomous operations

```
"Monitor this GitHub repo and notify me of new issues"
"Check Bitcoin price every hour"
"Backup my documents folder daily"
"Remind me to exercise every morning at 7 AM"
```

### 7.3 Slash commands

```
/start - Start conversation
/new - Start new conversation (clear context)
/stop - Stop current task
/help - Show available commands
/status - Show agent status
/memory - View saved memories
/skills - List available skills
```

## Part 8: Troubleshooting

### 8.1 Agent not responding

```bash
# Check if agent is running
ps aux | grep hermes

# Check logs
tail -f ~/.hermes/logs/hermes.log

# Test Telegram connection
hermes gateway test telegram
```

### 8.2 "Unauthorized" error from Telegram

- Verify bot token is correct in config.yaml
- Make sure you sent `/start` to the bot first
- Check that your chat ID is in `allowed_users`

### 8.3 Model API errors

```bash
# Test model connection
hermes model test

# Check API key
hermes config get model.api_key

# View detailed error logs
hermes start --debug
```

### 8.4 Permission errors

```bash
# Fix credentials directory permissions
chmod 700 ~/.agent/credentials
chmod 600 ~/.agent/credentials/*

# Fix config permissions
chmod 600 ~/.hermes/config.yaml
```

## Part 9: Security Best Practices

### 9.1 Protect sensitive files

```bash
# Never commit these files to git
echo ".hermes/config.yaml" >> .gitignore
echo ".agent/credentials/" >> .gitignore

# Use environment variables for secrets
export TELEGRAM_BOT_TOKEN="your_token"
export ANTHROPIC_API_KEY="your_key"
```

### 9.2 Limit bot access

In `config.yaml`:
```yaml
gateways:
  telegram:
    allowed_users:
      - 123456789  # Only your chat ID
    rate_limit:
      max_requests: 100
      window_seconds: 3600
```

### 9.3 Restrict file access

```yaml
tools:
  file:
    allowed_paths:
      - /home/YOUR_USERNAME/projects
      - /tmp
    blocked_paths:
      - /etc
      - /root
      - ~/.ssh
```

## Part 10: Next Steps

### 10.1 Explore skills

```bash
# List available skills
hermes skills list

# Load a skill
hermes skills load github-pr-workflow

# Create custom skill
hermes skills create my-custom-workflow
```

### 10.2 Set up cron jobs

```
# In Telegram, tell your agent:
"Run 'git pull' in /path/to/repo every day at 9 AM"
"Check disk space every hour and alert if > 80%"
"Backup database every night at 2 AM"
```

### 10.3 Connect more platforms

- GitHub (for code management)
- Discord (for community management)
- Email (for notifications)
- Twitter/X (for social media)

See documentation: https://hermes-agent.nousresearch.com/docs

## Support & Resources

- **Documentation**: https://hermes-agent.nousresearch.com/docs
- **GitHub**: https://github.com/NousResearch/hermes-agent
- **Discord**: Join Nous Research Discord
- **Issues**: Report bugs on GitHub Issues

## Quick Reference Card

```bash
# Installation
pip install hermes-agent

# Setup
hermes setup

# Start agent
hermes start

# Background service
sudo systemctl start hermes-agent

# View logs
tail -f ~/.hermes/logs/hermes.log

# Test Telegram
hermes gateway test telegram

# Validate config
hermes config validate
```

---

**Congratulations!** You now have a fully functional AI agent connected to Telegram. Start chatting with your bot and explore its capabilities.

For questions or issues, refer to the official documentation or community support channels.
