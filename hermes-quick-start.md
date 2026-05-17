# Hermes Agent - Quick Start Guide (5 Minutes)

Get your AI agent running in 5 minutes. Perfect for beginners.

## Step 1: Install Hermes (1 minute)

```bash
# Install via pip
pip install hermes-agent

# Verify installation
hermes --version
```

**Expected output:**
```
Hermes Agent v1.x.x
```

## Step 2: Get API Key (2 minutes)

Choose ONE option:

### Option A: OpenRouter (Recommended for Beginners)

1. Go to https://openrouter.ai/
2. Sign up with Google/GitHub
3. Click "Keys" → "Create Key"
4. Copy your API key (starts with `sk-or-v1-...`)
5. Add $5-10 credits

### Option B: Anthropic Direct

1. Go to https://console.anthropic.com/
2. Sign up
3. Go to "API Keys" → "Create Key"
4. Copy your API key (starts with `sk-ant-...`)
5. Add payment method

### Option C: Local (Free, but needs GPU)

```bash
# Install Ollama
curl -fsSL https://ollama.com/install.sh | sh

# Download model (7GB)
ollama pull llama3.1:70b
```

## Step 3: Create Telegram Bot (1 minute)

1. Open Telegram
2. Search for `@BotFather`
3. Send: `/newbot`
4. Choose name: `My AI Assistant`
5. Choose username: `my_ai_assistant_bot`
6. Copy the **bot token** (looks like `1234567890:ABCdef...`)

## Step 4: Get Your Chat ID (30 seconds)

```bash
# Send any message to your bot first, then run:
curl "https://api.telegram.org/bot<YOUR_BOT_TOKEN>/getUpdates" | grep -o '"id":[0-9]*' | head -1 | cut -d: -f2
```

**Or manually:**
1. Send a message to your bot
2. Visit: `https://api.telegram.org/bot<YOUR_BOT_TOKEN>/getUpdates`
3. Find `"chat":{"id":123456789}`
4. Copy the number (your chat ID)

## Step 5: Configure Hermes (1 minute)

```bash
# Run setup wizard
hermes setup
```

**Answer the prompts:**

```
Model provider? [openrouter/anthropic/ollama]: openrouter
API key: sk-or-v1-YOUR_KEY_HERE
Model: anthropic/claude-sonnet-4

Enable Telegram? [y/n]: y
Bot token: 1234567890:ABCdef...
Your chat ID: 123456789

Agent name: MyAgent
```

**Or create config manually:**

```bash
mkdir -p ~/.hermes
cat > ~/.hermes/config.yaml << 'EOF'
agent:
  name: MyAgent

model:
  provider: openrouter
  model: anthropic/claude-sonnet-4
  api_key: sk-or-v1-YOUR_KEY_HERE

gateways:
  telegram:
    enabled: true
    bot_token: "1234567890:ABCdef..."
    home_chat_id: 123456789
    allowed_users:
      - 123456789

tools:
  web:
    enabled: true
  terminal:
    enabled: true
  file:
    enabled: true
  browser:
    enabled: true
EOF
```

## Step 6: Start Your Agent (10 seconds)

```bash
hermes start
```

**Expected output:**
```
✓ Configuration loaded
✓ Model connected (anthropic/claude-sonnet-4)
✓ Telegram gateway connected
✓ Listening for messages...
```

## Step 7: Test It! (30 seconds)

Open Telegram and send to your bot:

```
/start
```

Bot should reply with introduction.

**Try these commands:**

```
Hello, who are you?

Search for latest AI news

What files are in my home directory?

Create a Python script that prints "Hello World"

What's the weather like today?
```

## That's It! 🎉

Your AI agent is now running and connected to Telegram.

---

## What's Next?

### Make It Autonomous

Tell your agent:
```
Remember: You can execute commands without asking me first for safe operations
```

### Add More Features

```bash
# Enable GitHub integration
hermes config set integrations.github.enabled true
hermes config set integrations.github.token ghp_YOUR_GITHUB_TOKEN

# Enable Discord
hermes config set gateways.discord.enabled true
hermes config set gateways.discord.token YOUR_DISCORD_BOT_TOKEN
```

### Schedule Tasks

```
Run this command every day at 9 AM: echo "Good morning!"

Check Bitcoin price every hour and notify me if it's above $50k

Backup my documents folder every night at 2 AM
```

### Run in Background

**Option A: Using screen**
```bash
screen -S hermes
hermes start
# Press Ctrl+A then D to detach
```

**Option B: Using systemd (Linux)**
```bash
# Create service
sudo tee /etc/systemd/system/hermes.service << 'EOF'
[Unit]
Description=Hermes AI Agent
After=network.target

[Service]
Type=simple
User=$USER
ExecStart=/usr/local/bin/hermes start
Restart=always

[Install]
WantedBy=multi-user.target
EOF

# Start service
sudo systemctl daemon-reload
sudo systemctl enable hermes
sudo systemctl start hermes

# Check status
sudo systemctl status hermes
```

---

## Common Issues

### "API key invalid"
- Check your API key is correct
- Make sure you added credits (OpenRouter)
- Test: `hermes model test`

### "Bot not responding"
- Check bot token is correct
- Make sure you sent `/start` first
- Check your chat ID is in `allowed_users`
- Test: `hermes gateway test telegram`

### "Permission denied"
```bash
# Fix permissions
chmod 600 ~/.hermes/config.yaml
```

### "Command not found: hermes"
```bash
# Add to PATH
echo 'export PATH="$HOME/.local/bin:$PATH"' >> ~/.bashrc
source ~/.bashrc
```

---

## Cost Estimate

**OpenRouter (Claude Sonnet 4):**
- Light use (10 messages/day): ~$0.50/day
- Moderate use (50 messages/day): ~$2-3/day
- Heavy use (200 messages/day): ~$10-15/day

**Tips to reduce costs:**
- Use Claude Haiku for simple tasks ($0.25/$1.25 per 1M tokens)
- Enable caching
- Use local model (Ollama) when possible

---

## Example Workflows

### Personal Assistant
```
Check my email and summarize important messages

What's on my calendar today?

Remind me to call John at 3 PM

Create a todo list for this week
```

### Developer Assistant
```
Create a new GitHub repo called "my-project"

Write a Python script to scrape this website

Run tests in my project directory

Review this code and suggest improvements
```

### Research Assistant
```
Search for papers about "transformer architectures"

Summarize this article: [URL]

Compare these 3 products and make a recommendation

Track mentions of "AI" on Twitter and notify me of viral tweets
```

### Crypto Assistant
```
Check my wallet balance on all chains

What's the current ETH price?

Swap 0.1 ETH to USDC on Uniswap

Alert me when BTC drops below $40k
```

---

## Full Documentation

For advanced features, see:
- **Setup Guide**: `/root/hermes-telegram-setup-guide.md`
- **Advanced Features**: `/root/hermes-advanced-features.md`
- **Use Cases**: `/root/hermes-use-cases-examples.md`
- **Troubleshooting**: `/root/hermes-troubleshooting-faq.md`
- **Production Deployment**: `/root/hermes-production-deployment.md`

**Official Docs**: https://hermes-agent.nousresearch.com/docs

---

## Support

- **Discord**: Nous Research Discord
- **GitHub**: https://github.com/NousResearch/hermes-agent
- **Issues**: Report bugs on GitHub Issues

---

**Congratulations! You now have your own AI agent. Start chatting and explore what it can do!** 🚀
