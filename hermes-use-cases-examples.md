# Hermes Agent - Real-World Use Cases & Examples

Practical examples and complete workflows for common use cases.

## Use Case 1: Crypto Trading Bot

### Setup

**1. Configure wallet and exchanges:**

```bash
# Save wallet credentials
cat > ~/.agent/credentials/wallet.env << 'EOF'
SOLANA_ADDRESS=YourSolanaAddress
SOLANA_PRIVATE_KEY=your_private_key
EVM_ADDRESS=0xYourEthAddress
EVM_PRIVATE_KEY=0xyour_private_key

# RPC endpoints
SOLANA_RPC=https://api.mainnet-beta.solana.com
ETHEREUM_RPC=https://eth-mainnet.g.alchemy.com/v2/YOUR_KEY
BASE_RPC=https://mainnet.base.org
EOF

chmod 600 ~/.agent/credentials/wallet.env
```

**2. Create trading strategy script:**

```bash
cat > ~/.hermes/scripts/trading_strategy.py << 'EOF'
#!/usr/bin/env python3
import requests
import json
from datetime import datetime

def check_opportunities():
    """Check for trading opportunities"""
    
    # Example: Monitor ETH/USDC price on Uniswap
    url = "https://api.dexscreener.com/latest/dex/tokens/0xC02aaA39b223FE8D0A0e5C4F27eAD9083C756Cc2"
    response = requests.get(url)
    data = response.json()
    
    if data.get('pairs'):
        pair = data['pairs'][0]
        price = float(pair['priceUsd'])
        volume_24h = float(pair['volume']['h24'])
        
        # Simple strategy: Buy if price drops 5% and volume is high
        if price < 3000 and volume_24h > 1000000000:
            print(f"BUY_SIGNAL: ETH at ${price:.2f}, Volume: ${volume_24h/1e9:.2f}B")
            return True
    
    return False

if __name__ == "__main__":
    if check_opportunities():
        print("Trading opportunity detected!")
    else:
        print("No opportunities at this time.")
EOF

chmod +x ~/.hermes/scripts/trading_strategy.py
```

**3. Set up monitoring via Telegram:**

Tell your agent:
```
"Run trading_strategy.py every 5 minutes. If it prints BUY_SIGNAL, notify me immediately on Telegram with the details."
```

**4. Manual trading commands:**

```
"Check ETH price on Uniswap"
"Swap 0.1 ETH to USDC on Uniswap"
"Check my wallet balance on Ethereum and Base"
"What's the gas price on Ethereum right now?"
```

**5. Automated trading (with confirmation):**

```
"If ETH drops below $2900, swap 0.5 ETH to USDC automatically"
"Set a stop-loss: if my ETH position drops 10%, sell to USDC"
"DCA strategy: buy $100 of ETH every Monday at 9 AM"
```

### Advanced: MEV Bot

```python
# ~/.hermes/scripts/mev_monitor.py
#!/usr/bin/env python3
from web3 import Web3
import asyncio

async def monitor_mempool():
    """Monitor mempool for MEV opportunities"""
    w3 = Web3(Web3.HTTPProvider('YOUR_RPC_WITH_MEMPOOL_ACCESS'))
    
    # Subscribe to pending transactions
    pending_filter = w3.eth.filter('pending')
    
    while True:
        for tx_hash in pending_filter.get_new_entries():
            tx = w3.eth.get_transaction(tx_hash)
            
            # Analyze transaction for MEV opportunity
            if is_mev_opportunity(tx):
                print(f"MEV_OPPORTUNITY: {tx_hash.hex()}")
        
        await asyncio.sleep(0.1)

def is_mev_opportunity(tx):
    # Your MEV detection logic
    return False

if __name__ == "__main__":
    asyncio.run(monitor_mempool())
```

## Use Case 2: Social Media Manager

### Setup Multi-Account Posting

**1. Configure accounts:**

```bash
# Twitter/X account 1
cat > ~/.agent/credentials/x-main-cookies.json << 'EOF'
[
  {"name": "auth_token", "value": "...", "domain": ".x.com"},
  {"name": "ct0", "value": "...", "domain": ".x.com"}
]
EOF

# Twitter/X account 2
cat > ~/.agent/credentials/x-alt-cookies.json << 'EOF'
[
  {"name": "auth_token", "value": "...", "domain": ".x.com"},
  {"name": "ct0", "value": "...", "domain": ".x.com"}
]
EOF

chmod 600 ~/.agent/credentials/x-*-cookies.json
```

**2. Create posting script:**

```python
# ~/.hermes/scripts/social_media_post.py
#!/usr/bin/env python3
import sys
import json
import requests
from pathlib import Path

def post_to_twitter(text, account="main"):
    """Post to Twitter/X"""
    cookies_file = Path.home() / f".agent/credentials/x-{account}-cookies.json"
    
    with open(cookies_file) as f:
        cookies = json.load(f)
    
    # Convert to dict
    cookie_dict = {c['name']: c['value'] for c in cookies}
    
    # GraphQL API call
    url = "https://api.x.com/graphql/5CdvsV_zjv4L64XFifAglw/CreateTweet"
    headers = {
        "authorization": "Bearer AAAAAAAAAAAAAAAAAAAAANRILgAAAAAAnNwIzUejRCOuH5E6I8xnZz4puTs%3D1Zv7ttfk8LF81IUq16cHjhLTvJu4FA33AGWWjCpTnA",
        "x-csrf-token": cookie_dict.get('ct0', ''),
        "content-type": "application/json",
    }
    
    payload = {
        "variables": {
            "tweet_text": text,
            "dark_request": False,
            "media": {"media_entities": [], "possibly_sensitive": False},
            "semantic_annotation_ids": []
        },
        "features": {},
        "queryId": "5CdvsV_zjv4L64XFifAglw"
    }
    
    response = requests.post(url, headers=headers, cookies=cookie_dict, json=payload)
    
    if response.status_code == 200:
        data = response.json()
        tweet_id = data['data']['create_tweet']['tweet_results']['result']['rest_id']
        print(f"SUCCESS: https://x.com/{account}/status/{tweet_id}")
        return True
    else:
        print(f"ERROR: {response.status_code} - {response.text}")
        return False

if __name__ == "__main__":
    if len(sys.argv) < 2:
        print("Usage: social_media_post.py <text> [account]")
        sys.exit(1)
    
    text = sys.argv[1]
    account = sys.argv[2] if len(sys.argv) > 2 else "main"
    
    post_to_twitter(text, account)
EOF

chmod +x ~/.hermes/scripts/social_media_post.py
```

**3. Content generation with AI:**

```
"Generate 5 engaging tweets about AI and crypto, max 280 characters each"
"Create a Twitter thread about Hermes Agent features (10 tweets)"
"Write a professional LinkedIn post about my new project"
```

**4. Scheduled posting:**

```
"Post to Twitter every day at 10 AM with AI-generated content about tech news"
"Schedule these 5 tweets to post every 4 hours starting tomorrow"
"Post to both my Twitter accounts: main at 9 AM, alt at 3 PM daily"
```

**5. Engagement automation:**

```
"Monitor mentions of @myhandle and auto-reply to questions"
"Like and retweet posts from @influencer1, @influencer2, @influencer3"
"Track hashtag #AI and notify me of viral tweets (>1000 likes)"
```

### Cross-Platform Posting

```python
# ~/.hermes/scripts/cross_post.py
#!/usr/bin/env python3
import sys

def cross_post(content, platforms):
    """Post to multiple platforms"""
    results = {}
    
    if 'twitter' in platforms:
        # Post to Twitter
        results['twitter'] = post_to_twitter(content)
    
    if 'discord' in platforms:
        # Post to Discord
        results['discord'] = post_to_discord(content)
    
    if 'telegram' in platforms:
        # Post to Telegram channel
        results['telegram'] = post_to_telegram(content)
    
    return results

if __name__ == "__main__":
    content = sys.argv[1]
    platforms = sys.argv[2].split(',') if len(sys.argv) > 2 else ['twitter']
    
    results = cross_post(content, platforms)
    print(f"Posted to: {', '.join([p for p, success in results.items() if success])}")
EOF
```

**Usage:**

```
"Post this to Twitter, Discord, and my Telegram channel: 'New blog post is live!'"
"Cross-post my latest GitHub release to all my social media"
```

## Use Case 3: DevOps Automation

### CI/CD Pipeline Integration

**1. GitHub Actions monitoring:**

```
"Monitor GitHub Actions in my-org/my-repo and notify me of failed builds"
"When a PR is opened, automatically run tests and post results as a comment"
"Deploy to production when a release is tagged"
```

**2. Server monitoring:**

```bash
# ~/.hermes/scripts/server_health.sh
#!/bin/bash

# Check multiple servers
SERVERS=("prod-1" "prod-2" "prod-3")

for server in "${SERVERS[@]}"; do
    echo "=== $server ==="
    
    # SSH and check health
    ssh $server << 'ENDSSH'
        # Disk usage
        df -h / | awk 'NR==2 {print "Disk: " $5}'
        
        # Memory
        free -h | awk 'NR==2 {print "Memory: " $3 "/" $2}'
        
        # CPU load
        uptime | awk -F'load average:' '{print "Load:" $2}'
        
        # Docker containers
        docker ps --format "{{.Names}}: {{.Status}}" | head -5
        
        # Check critical services
        systemctl is-active nginx postgresql redis
ENDSSH
    
    echo ""
done
EOF

chmod +x ~/.hermes/scripts/server_health.sh
```

**Schedule monitoring:**

```
"Run server_health.sh every 10 minutes and alert me if any service is down"
```

**3. Automated deployments:**

```
"When I push to main branch in my-app repo:
1. Run tests
2. Build Docker image
3. Push to registry
4. Deploy to staging
5. Run smoke tests
6. If all pass, ask me to deploy to production"
```

**4. Log analysis:**

```bash
# ~/.hermes/scripts/analyze_logs.py
#!/usr/bin/env python3
import re
from collections import Counter
from datetime import datetime, timedelta

def analyze_nginx_logs(log_file):
    """Analyze nginx access logs"""
    
    errors = []
    status_codes = Counter()
    ips = Counter()
    
    with open(log_file) as f:
        for line in f:
            # Parse log line
            match = re.search(r'(\d+\.\d+\.\d+\.\d+).*"(GET|POST).*" (\d+)', line)
            if match:
                ip, method, status = match.groups()
                status_codes[status] += 1
                
                if status.startswith('5'):
                    errors.append(line.strip())
                
                ips[ip] += 1
    
    # Report
    print(f"Total requests: {sum(status_codes.values())}")
    print(f"\nStatus codes:")
    for code, count in status_codes.most_common():
        print(f"  {code}: {count}")
    
    print(f"\nTop IPs:")
    for ip, count in ips.most_common(10):
        print(f"  {ip}: {count}")
    
    if errors:
        print(f"\n⚠️ {len(errors)} server errors detected!")
        print("Recent errors:")
        for error in errors[-5:]:
            print(f"  {error}")

if __name__ == "__main__":
    analyze_nginx_logs("/var/log/nginx/access.log")
EOF

chmod +x ~/.hermes/scripts/analyze_logs.py
```

**Usage:**

```
"Analyze nginx logs and report any anomalies"
"Check application logs for errors in the last hour"
"Summarize database slow query log"
```

### Infrastructure as Code

**5. Terraform automation:**

```
"Run terraform plan in /infra/aws and show me the changes"
"If the plan looks good, apply it"
"Monitor AWS costs and alert if daily spend > $100"
```

**6. Kubernetes management:**

```
"List all pods in production namespace"
"Restart the api-server deployment"
"Check logs of the failing pod"
"Scale web-app deployment to 5 replicas"
```

## Use Case 4: Research Assistant

### Academic Research

**1. Paper discovery:**

```
"Search arXiv for papers about 'transformer architectures' published in 2026"
"Find papers by author 'Yann LeCun' in the last 6 months"
"What are the most cited papers on 'reinforcement learning' this year?"
```

**2. Paper analysis:**

```
"Download and summarize this paper: https://arxiv.org/abs/2401.12345"
"Compare these 3 papers and highlight key differences"
"Extract methodology from this paper and create a step-by-step guide"
```

**3. Literature review automation:**

```
"Create a literature review on 'large language models' with papers from 2024-2026"
"Track new papers on 'computer vision' and notify me weekly with summaries"
"Build a citation graph for papers related to 'neural architecture search'"
```

### Market Research

**4. Competitor analysis:**

```
"Monitor competitor websites and alert me of changes"
"Track pricing changes for products in this category"
"Analyze competitor social media activity and engagement"
```

**5. Trend monitoring:**

```
"What's trending on Twitter in #AI and #crypto?"
"Monitor Google Trends for 'artificial intelligence' and report weekly"
"Track mentions of 'my-product' across social media"
```

**6. Data collection:**

```bash
# ~/.hermes/scripts/scrape_data.py
#!/usr/bin/env python3
import requests
from bs4 import BeautifulSoup
import json
from datetime import datetime

def scrape_product_prices(urls):
    """Scrape product prices from multiple sites"""
    
    results = []
    
    for url in urls:
        response = requests.get(url)
        soup = BeautifulSoup(response.content, 'html.parser')
        
        # Extract price (adjust selectors for your sites)
        price_elem = soup.select_one('.price, .product-price, [itemprop="price"]')
        if price_elem:
            price = price_elem.text.strip()
            results.append({
                'url': url,
                'price': price,
                'timestamp': datetime.now().isoformat()
            })
    
    return results

if __name__ == "__main__":
    urls = [
        "https://example.com/product1",
        "https://example.com/product2",
    ]
    
    data = scrape_product_prices(urls)
    print(json.dumps(data, indent=2))
EOF

chmod +x ~/.hermes/scripts/scrape_data.py
```

**Schedule data collection:**

```
"Run scrape_data.py daily at 8 AM and save results to /data/prices.json"
"If any price changes by more than 10%, notify me immediately"
```

## Use Case 5: Personal Productivity

### Email Management

**1. Auto-categorization:**

```
"Check my email every hour and:
- Archive newsletters
- Flag urgent emails from my boss
- Auto-reply to meeting requests with my availability
- Summarize important emails and send to Telegram"
```

**2. Email templates:**

```
"When I receive a job application email, reply with:
'Thank you for your application. We'll review it and get back to you within 5 business days.'"
```

**3. Follow-up reminders:**

```
"If I send an email and don't get a reply in 3 days, remind me to follow up"
```

### Calendar & Scheduling

**4. Meeting management:**

```
"Check my calendar for tomorrow and send me a summary"
"Find a 1-hour slot this week when both me and john@example.com are free"
"Remind me 15 minutes before each meeting"
"Block 2 hours every morning for deep work"
```

**5. Time tracking:**

```
"Track time I spend on different projects and report weekly"
"Log this task: 'Worked on feature X for 2 hours'"
"What did I work on last week?"
```

### Task Management

**6. Todo list integration:**

```
"Add to my todo list: 'Review PR #123'"
"What are my top 3 priorities today?"
"Mark 'Deploy to production' as done"
"Move all overdue tasks to today"
```

**7. Project tracking:**

```
"Create a project plan for 'New Website' with 10 tasks"
"What's the status of 'Mobile App' project?"
"Generate a progress report for this week"
```

## Use Case 6: Content Creation

### Blog Writing

**1. Content generation:**

```
"Write a 1000-word blog post about 'AI in Healthcare'"
"Create an outline for a tutorial on 'Building a REST API'"
"Generate 10 blog post ideas about crypto trading"
```

**2. SEO optimization:**

```
"Optimize this blog post for SEO with keyword 'machine learning'"
"Generate meta description and title for this article"
"Suggest internal links for this blog post"
```

**3. Publishing automation:**

```
"Publish this blog post to my WordPress site"
"Share this article on Twitter, LinkedIn, and Reddit"
"Schedule this post to publish next Monday at 9 AM"
```

### Video Content

**4. Script writing:**

```
"Write a YouTube video script about 'Getting Started with Hermes Agent' (10 minutes)"
"Create talking points for a podcast episode on AI ethics"
"Generate captions for this video transcript"
```

**5. Video editing automation:**

```bash
# ~/.hermes/scripts/auto_edit_video.sh
#!/bin/bash

INPUT=$1
OUTPUT=$2

# Auto-edit video with ffmpeg
ffmpeg -i "$INPUT" \
  -vf "scale=1920:1080,fps=30" \
  -c:v libx264 -preset fast -crf 22 \
  -c:a aac -b:a 192k \
  "$OUTPUT"

echo "Video edited: $OUTPUT"
EOF

chmod +x ~/.hermes/scripts/auto_edit_video.sh
```

**Usage:**

```
"Edit this video and optimize for YouTube"
"Extract audio from video and transcribe it"
"Generate thumbnail for this video"
```

## Use Case 7: Smart Home Integration

### Home Automation

**1. Device control:**

```
"Turn on living room lights"
"Set bedroom temperature to 22°C"
"Lock all doors"
"Start robot vacuum"
```

**2. Scenes and routines:**

```
"Create a 'Good Morning' routine:
- Turn on bedroom lights gradually
- Start coffee maker
- Read today's weather and calendar
- Play morning playlist"
```

**3. Monitoring:**

```
"Alert me if front door opens while I'm away"
"Notify if temperature drops below 18°C"
"Check security cameras and send snapshot if motion detected"
```

### Energy Management

**4. Usage tracking:**

```
"What's my electricity usage today?"
"Compare energy consumption this month vs last month"
"Which devices are using the most power?"
```

**5. Optimization:**

```
"Turn off all lights when no one is home"
"Adjust thermostat based on electricity prices"
"Charge EV when electricity is cheapest"
```

## Use Case 8: Learning & Education

### Study Assistant

**1. Flashcard generation:**

```
"Create flashcards from this textbook chapter"
"Generate quiz questions about 'Python decorators'"
"Test me on 'Machine Learning algorithms'"
```

**2. Concept explanation:**

```
"Explain 'blockchain consensus mechanisms' like I'm 10 years old"
"What's the difference between supervised and unsupervised learning?"
"Give me 3 real-world examples of 'recursion'"
```

**3. Study schedule:**

```
"Create a study plan for learning React in 30 days"
"Remind me to review flashcards every evening"
"Track my study hours and report weekly"
```

### Code Learning

**4. Practice problems:**

```
"Give me 5 LeetCode-style problems on 'binary trees'"
"Generate a coding challenge about 'async/await in JavaScript'"
"Create a project idea to practice 'REST API development'"
```

**5. Code review for learning:**

```
"Review my code and suggest improvements"
"Explain what this code does line by line"
"Refactor this code to be more Pythonic"
```

## Use Case 9: Health & Fitness

### Workout Tracking

**1. Exercise logging:**

```
"Log workout: 30 min run, 5km"
"Add to fitness log: Bench press 3x10 @ 80kg"
"What did I do at the gym this week?"
```

**2. Progress tracking:**

```
"Show my running progress over the last month"
"Am I hitting my weekly workout goal?"
"Generate a fitness report for this month"
```

**3. Workout planning:**

```
"Create a 4-week strength training program"
"Suggest a HIIT workout for 20 minutes"
"What exercises target lower back?"
```

### Nutrition

**4. Meal tracking:**

```
"Log meal: Chicken breast 200g, rice 150g, broccoli 100g"
"Calculate calories for today"
"Am I meeting my protein goal?"
```

**5. Meal planning:**

```
"Generate a meal plan for 2000 calories, high protein"
"Suggest 5 healthy dinner recipes under 500 calories"
"Create a grocery list for this week's meal plan"
```

## Use Case 10: Finance Management

### Expense Tracking

**1. Transaction logging:**

```
"Log expense: $50 for groceries"
"Add income: $5000 salary"
"What did I spend on restaurants this month?"
```

**2. Budget management:**

```
"Set budget: $500/month for food"
"Am I over budget in any category?"
"Show spending breakdown for last month"
```

**3. Financial reports:**

```
"Generate monthly financial report"
"Compare spending this month vs last month"
"What's my net worth?"
```

### Investment Tracking

**4. Portfolio monitoring:**

```
"Check my crypto portfolio value"
"What's my ROI on ETH?"
"Alert me if BTC drops below $40k"
```

**5. Market analysis:**

```
"Analyze S&P 500 performance this year"
"Compare ETH vs BTC returns in 2026"
"What are the top performing altcoins this month?"
```

---

## Tips for Effective Agent Usage

### 1. Be Specific

❌ "Check my stuff"
✅ "Check my GitHub notifications and email, summarize urgent items"

### 2. Set Clear Boundaries

```
"Monitor this but only notify me if it's critical"
"Run this daily but don't spam me with logs"
```

### 3. Use Confirmation Wisely

```
"Swap 0.1 ETH to USDC (confirm first)"
"Delete old logs (no confirmation needed)"
```

### 4. Chain Commands

```
"Pull latest code, run tests, and if they pass, deploy to staging"
```

### 5. Leverage Memory

```
"Remember: I prefer TypeScript over JavaScript"
"Remember: My work hours are 9 AM - 5 PM EST"
```

### 6. Create Reusable Workflows

```
"Save this as a skill: 'Deploy to Production'"
"Create a workflow for 'Weekly Report Generation'"
```

---

**These use cases demonstrate the versatility of Hermes Agent. Mix and match patterns to create your own custom workflows!**
