# Hermes Agent - Production Deployment & Best Practices

Complete guide for deploying Hermes Agent in production environments with security, reliability, and scalability.

## Part 1: Production Architecture

### 1.1 Recommended Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                     Load Balancer (nginx)                    │
│                    SSL/TLS Termination                       │
└─────────────────────┬───────────────────────────────────────┘
                      │
        ┌─────────────┼─────────────┐
        │             │             │
┌───────▼──────┐ ┌───▼──────┐ ┌───▼──────┐
│  Hermes      │ │  Hermes  │ │  Hermes  │
│  Instance 1  │ │  Inst. 2 │ │  Inst. 3 │
└───────┬──────┘ └────┬─────┘ └────┬─────┘
        │             │             │
        └─────────────┼─────────────┘
                      │
        ┌─────────────▼─────────────┐
        │    Shared Services        │
        │  - Redis (cache)          │
        │  - PostgreSQL (state)     │
        │  - S3 (file storage)      │
        └───────────────────────────┘
```

### 1.2 Infrastructure Requirements

**Minimum (Single Instance):**
- CPU: 2 cores
- RAM: 4GB
- Disk: 20GB SSD
- Network: 100 Mbps

**Recommended (Production):**
- CPU: 4-8 cores
- RAM: 8-16GB
- Disk: 50GB+ SSD
- Network: 1 Gbps

**High Availability:**
- 3+ instances across availability zones
- Load balancer with health checks
- Shared cache (Redis)
- Shared database (PostgreSQL)
- Object storage (S3/MinIO)

## Part 2: Server Setup

### 2.1 Ubuntu Server Setup

```bash
#!/bin/bash
# Production server setup script

# Update system
sudo apt-get update
sudo apt-get upgrade -y

# Install dependencies
sudo apt-get install -y \
    python3.11 \
    python3-pip \
    git \
    curl \
    wget \
    build-essential \
    libssl-dev \
    libffi-dev \
    python3-dev \
    redis-server \
    postgresql \
    nginx \
    certbot \
    python3-certbot-nginx

# Install Docker (optional)
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh
sudo usermod -aG docker $USER

# Install Docker Compose
sudo curl -L "https://github.com/docker/compose/releases/latest/download/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose

# Create hermes user
sudo useradd -m -s /bin/bash hermes
sudo usermod -aG sudo hermes

# Setup directories
sudo mkdir -p /opt/hermes
sudo chown hermes:hermes /opt/hermes

# Install Hermes Agent
sudo -u hermes pip3 install hermes-agent

echo "Server setup complete!"
```

### 2.2 Security Hardening

```bash
#!/bin/bash
# Security hardening script

# Configure firewall
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw allow 22/tcp    # SSH
sudo ufw allow 80/tcp    # HTTP
sudo ufw allow 443/tcp   # HTTPS
sudo ufw enable

# Disable root login
sudo sed -i 's/PermitRootLogin yes/PermitRootLogin no/' /etc/ssh/sshd_config

# Enable SSH key authentication only
sudo sed -i 's/#PasswordAuthentication yes/PasswordAuthentication no/' /etc/ssh/sshd_config

# Restart SSH
sudo systemctl restart sshd

# Install fail2ban
sudo apt-get install -y fail2ban
sudo systemctl enable fail2ban
sudo systemctl start fail2ban

# Configure fail2ban for SSH
cat > /etc/fail2ban/jail.local << 'EOF'
[sshd]
enabled = true
port = 22
filter = sshd
logpath = /var/log/auth.log
maxretry = 3
bantime = 3600
EOF

sudo systemctl restart fail2ban

# Setup automatic security updates
sudo apt-get install -y unattended-upgrades
sudo dpkg-reconfigure -plow unattended-upgrades

echo "Security hardening complete!"
```

### 2.3 SSL/TLS Setup

```bash
#!/bin/bash
# SSL certificate setup with Let's Encrypt

DOMAIN="agent.yourdomain.com"
EMAIL="your-email@example.com"

# Install certbot
sudo apt-get install -y certbot python3-certbot-nginx

# Obtain certificate
sudo certbot --nginx -d $DOMAIN --email $EMAIL --agree-tos --non-interactive

# Auto-renewal
sudo systemctl enable certbot.timer
sudo systemctl start certbot.timer

# Test renewal
sudo certbot renew --dry-run

echo "SSL setup complete for $DOMAIN"
```

## Part 3: Application Deployment

### 3.1 Systemd Service

```bash
# Create systemd service file
sudo tee /etc/systemd/system/hermes-agent.service << 'EOF'
[Unit]
Description=Hermes AI Agent
After=network.target redis.service postgresql.service
Wants=redis.service postgresql.service

[Service]
Type=simple
User=hermes
Group=hermes
WorkingDirectory=/opt/hermes
Environment="PATH=/home/hermes/.local/bin:/usr/local/bin:/usr/bin:/bin"
Environment="PYTHONUNBUFFERED=1"
ExecStart=/home/hermes/.local/bin/hermes start
Restart=always
RestartSec=10
StandardOutput=journal
StandardError=journal
SyslogIdentifier=hermes-agent

# Security settings
NoNewPrivileges=true
PrivateTmp=true
ProtectSystem=strict
ProtectHome=true
ReadWritePaths=/opt/hermes /home/hermes/.hermes /home/hermes/.agent

# Resource limits
LimitNOFILE=65536
MemoryMax=4G
CPUQuota=200%

[Install]
WantedBy=multi-user.target
EOF

# Enable and start service
sudo systemctl daemon-reload
sudo systemctl enable hermes-agent
sudo systemctl start hermes-agent

# Check status
sudo systemctl status hermes-agent
```

### 3.2 Nginx Reverse Proxy

```nginx
# /etc/nginx/sites-available/hermes-agent
upstream hermes_backend {
    least_conn;
    server 127.0.0.1:8080 max_fails=3 fail_timeout=30s;
    # Add more instances for HA
    # server 10.0.1.2:8080 max_fails=3 fail_timeout=30s;
    # server 10.0.1.3:8080 max_fails=3 fail_timeout=30s;
}

# Rate limiting
limit_req_zone $binary_remote_addr zone=hermes_limit:10m rate=10r/s;
limit_conn_zone $binary_remote_addr zone=hermes_conn:10m;

server {
    listen 80;
    server_name agent.yourdomain.com;
    
    # Redirect to HTTPS
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name agent.yourdomain.com;
    
    # SSL configuration
    ssl_certificate /etc/letsencrypt/live/agent.yourdomain.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/agent.yourdomain.com/privkey.pem;
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers HIGH:!aNULL:!MD5;
    ssl_prefer_server_ciphers on;
    ssl_session_cache shared:SSL:10m;
    ssl_session_timeout 10m;
    
    # Security headers
    add_header Strict-Transport-Security "max-age=31536000; includeSubDomains" always;
    add_header X-Frame-Options "SAMEORIGIN" always;
    add_header X-Content-Type-Options "nosniff" always;
    add_header X-XSS-Protection "1; mode=block" always;
    
    # Logging
    access_log /var/log/nginx/hermes-access.log;
    error_log /var/log/nginx/hermes-error.log;
    
    # Rate limiting
    limit_req zone=hermes_limit burst=20 nodelay;
    limit_conn hermes_conn 10;
    
    # Webhook endpoint
    location /webhook {
        proxy_pass http://hermes_backend;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        
        # Timeouts
        proxy_connect_timeout 60s;
        proxy_send_timeout 60s;
        proxy_read_timeout 60s;
        
        # Buffer settings
        proxy_buffering off;
        proxy_request_buffering off;
    }
    
    # Health check endpoint
    location /health {
        proxy_pass http://hermes_backend;
        access_log off;
    }
    
    # Metrics endpoint (restrict access)
    location /metrics {
        allow 10.0.0.0/8;  # Internal network only
        deny all;
        proxy_pass http://hermes_backend;
    }
}
```

```bash
# Enable site
sudo ln -s /etc/nginx/sites-available/hermes-agent /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl reload nginx
```

### 3.3 Docker Deployment

**Dockerfile:**

```dockerfile
FROM python:3.11-slim

# Install system dependencies
RUN apt-get update && apt-get install -y \
    git \
    curl \
    build-essential \
    && rm -rf /var/lib/apt/lists/*

# Create app user
RUN useradd -m -u 1000 hermes

# Install Hermes Agent
RUN pip install --no-cache-dir hermes-agent

# Create directories
RUN mkdir -p /home/hermes/.hermes /home/hermes/.agent/credentials
RUN chown -R hermes:hermes /home/hermes

# Switch to app user
USER hermes
WORKDIR /home/hermes

# Health check
HEALTHCHECK --interval=30s --timeout=10s --start-period=40s --retries=3 \
    CMD curl -f http://localhost:8080/health || exit 1

# Expose webhook port
EXPOSE 8080

# Start agent
CMD ["hermes", "start"]
```

**docker-compose.yml:**

```yaml
version: '3.8'

services:
  hermes:
    build: .
    image: hermes-agent:latest
    container_name: hermes-agent
    restart: unless-stopped
    
    volumes:
      - ./config:/home/hermes/.hermes:ro
      - ./credentials:/home/hermes/.agent/credentials:ro
      - hermes-data:/home/hermes/.hermes/data
      - hermes-cache:/home/hermes/.hermes/cache
    
    environment:
      - TZ=UTC
      - PYTHONUNBUFFERED=1
    
    ports:
      - "127.0.0.1:8080:8080"
    
    depends_on:
      - redis
      - postgres
    
    networks:
      - hermes-network
    
    logging:
      driver: "json-file"
      options:
        max-size: "10m"
        max-file: "3"
    
    deploy:
      resources:
        limits:
          cpus: '2'
          memory: 4G
        reservations:
          cpus: '1'
          memory: 2G

  redis:
    image: redis:7-alpine
    container_name: hermes-redis
    restart: unless-stopped
    
    command: redis-server --appendonly yes --requirepass ${REDIS_PASSWORD}
    
    volumes:
      - redis-data:/data
    
    networks:
      - hermes-network
    
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 10s
      timeout: 3s
      retries: 3

  postgres:
    image: postgres:15-alpine
    container_name: hermes-postgres
    restart: unless-stopped
    
    environment:
      - POSTGRES_DB=hermes
      - POSTGRES_USER=hermes
      - POSTGRES_PASSWORD=${POSTGRES_PASSWORD}
    
    volumes:
      - postgres-data:/var/lib/postgresql/data
    
    networks:
      - hermes-network
    
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U hermes"]
      interval: 10s
      timeout: 3s
      retries: 3

  nginx:
    image: nginx:alpine
    container_name: hermes-nginx
    restart: unless-stopped
    
    ports:
      - "80:80"
      - "443:443"
    
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf:ro
      - ./ssl:/etc/nginx/ssl:ro
      - nginx-logs:/var/log/nginx
    
    depends_on:
      - hermes
    
    networks:
      - hermes-network

volumes:
  hermes-data:
  hermes-cache:
  redis-data:
  postgres-data:
  nginx-logs:

networks:
  hermes-network:
    driver: bridge
```

**Deploy:**

```bash
# Create .env file
cat > .env << 'EOF'
REDIS_PASSWORD=your_secure_redis_password
POSTGRES_PASSWORD=your_secure_postgres_password
EOF

# Build and start
docker-compose up -d

# View logs
docker-compose logs -f hermes

# Scale (if using multiple instances)
docker-compose up -d --scale hermes=3
```

## Part 4: Monitoring & Observability

### 4.1 Prometheus Metrics

**prometheus.yml:**

```yaml
global:
  scrape_interval: 15s
  evaluation_interval: 15s

scrape_configs:
  - job_name: 'hermes-agent'
    static_configs:
      - targets: ['localhost:9090']
    metrics_path: '/metrics'
```

**Hermes config for metrics:**

```yaml
monitoring:
  enabled: true
  prometheus:
    enabled: true
    port: 9090
    metrics:
      - requests_total
      - requests_duration_seconds
      - errors_total
      - active_sessions
      - model_tokens_used
      - model_api_calls
      - cache_hits
      - cache_misses
```

### 4.2 Grafana Dashboard

```json
{
  "dashboard": {
    "title": "Hermes Agent Monitoring",
    "panels": [
      {
        "title": "Request Rate",
        "targets": [
          {
            "expr": "rate(hermes_requests_total[5m])"
          }
        ]
      },
      {
        "title": "Error Rate",
        "targets": [
          {
            "expr": "rate(hermes_errors_total[5m])"
          }
        ]
      },
      {
        "title": "Response Time (p95)",
        "targets": [
          {
            "expr": "histogram_quantile(0.95, hermes_requests_duration_seconds_bucket)"
          }
        ]
      },
      {
        "title": "Active Sessions",
        "targets": [
          {
            "expr": "hermes_active_sessions"
          }
        ]
      },
      {
        "title": "Model Token Usage",
        "targets": [
          {
            "expr": "rate(hermes_model_tokens_used[1h])"
          }
        ]
      },
      {
        "title": "Cache Hit Rate",
        "targets": [
          {
            "expr": "hermes_cache_hits / (hermes_cache_hits + hermes_cache_misses)"
          }
        ]
      }
    ]
  }
}
```

### 4.3 Logging Stack (ELK)

**filebeat.yml:**

```yaml
filebeat.inputs:
  - type: log
    enabled: true
    paths:
      - /var/log/hermes/*.log
    fields:
      service: hermes-agent
    json.keys_under_root: true
    json.add_error_key: true

output.elasticsearch:
  hosts: ["localhost:9200"]
  index: "hermes-logs-%{+yyyy.MM.dd}"

setup.kibana:
  host: "localhost:5601"
```

### 4.4 Alerting

**alertmanager.yml:**

```yaml
route:
  receiver: 'telegram'
  group_by: ['alertname', 'severity']
  group_wait: 10s
  group_interval: 10s
  repeat_interval: 1h

receivers:
  - name: 'telegram'
    telegram_configs:
      - bot_token: 'YOUR_BOT_TOKEN'
        chat_id: YOUR_CHAT_ID
        message: |
          🚨 Alert: {{ .GroupLabels.alertname }}
          Severity: {{ .GroupLabels.severity }}
          {{ range .Alerts }}
          - {{ .Annotations.description }}
          {{ end }}
```

**alert_rules.yml:**

```yaml
groups:
  - name: hermes_alerts
    interval: 30s
    rules:
      - alert: HighErrorRate
        expr: rate(hermes_errors_total[5m]) > 0.1
        for: 5m
        labels:
          severity: critical
        annotations:
          description: "Error rate is {{ $value }} errors/sec"
      
      - alert: HighResponseTime
        expr: histogram_quantile(0.95, hermes_requests_duration_seconds_bucket) > 5
        for: 5m
        labels:
          severity: warning
        annotations:
          description: "P95 response time is {{ $value }}s"
      
      - alert: ServiceDown
        expr: up{job="hermes-agent"} == 0
        for: 1m
        labels:
          severity: critical
        annotations:
          description: "Hermes agent is down"
      
      - alert: HighMemoryUsage
        expr: process_resident_memory_bytes > 4e9
        for: 5m
        labels:
          severity: warning
        annotations:
          description: "Memory usage is {{ $value | humanize }}B"
```

## Part 5: Backup & Disaster Recovery

### 5.1 Automated Backup Script

```bash
#!/bin/bash
# /opt/hermes/backup.sh

set -e

BACKUP_DIR="/backup/hermes"
DATE=$(date +%Y%m%d_%H%M%S)
BACKUP_PATH="$BACKUP_DIR/$DATE"
RETENTION_DAYS=30

# Create backup directory
mkdir -p "$BACKUP_PATH"

# Backup configuration
echo "Backing up configuration..."
cp -r /home/hermes/.hermes/config.yaml "$BACKUP_PATH/"
cp -r /home/hermes/.hermes/SOUL.md "$BACKUP_PATH/"

# Backup credentials (encrypted)
echo "Backing up credentials..."
tar -czf - /home/hermes/.agent/credentials | \
    openssl enc -aes-256-cbc -salt -pbkdf2 \
    -pass file:/opt/hermes/.backup_password \
    -out "$BACKUP_PATH/credentials.tar.gz.enc"

# Backup database
echo "Backing up database..."
if [ -f /home/hermes/.hermes/memory.db ]; then
    sqlite3 /home/hermes/.hermes/memory.db ".backup '$BACKUP_PATH/memory.db'"
fi

# Backup PostgreSQL (if using)
if command -v pg_dump &> /dev/null; then
    pg_dump -U hermes hermes | gzip > "$BACKUP_PATH/postgres.sql.gz"
fi

# Backup skills
echo "Backing up skills..."
tar -czf "$BACKUP_PATH/skills.tar.gz" /home/hermes/.hermes/skills/

# Create manifest
cat > "$BACKUP_PATH/manifest.txt" << EOF
Backup Date: $DATE
Hostname: $(hostname)
Hermes Version: $(hermes --version)
Files:
$(ls -lh "$BACKUP_PATH")
EOF

# Upload to S3 (optional)
if command -v aws &> /dev/null; then
    echo "Uploading to S3..."
    aws s3 sync "$BACKUP_PATH" "s3://your-backup-bucket/hermes/$DATE/"
fi

# Cleanup old backups
echo "Cleaning up old backups..."
find "$BACKUP_DIR" -type d -mtime +$RETENTION_DAYS -exec rm -rf {} +

echo "Backup completed: $BACKUP_PATH"
```

**Schedule backup:**

```bash
# Add to crontab
sudo crontab -e

# Daily backup at 3 AM
0 3 * * * /opt/hermes/backup.sh >> /var/log/hermes-backup.log 2>&1
```

### 5.2 Disaster Recovery Procedure

```bash
#!/bin/bash
# /opt/hermes/restore.sh

set -e

BACKUP_PATH=$1

if [ -z "$BACKUP_PATH" ]; then
    echo "Usage: $0 <backup_path>"
    exit 1
fi

echo "Restoring from: $BACKUP_PATH"

# Stop service
sudo systemctl stop hermes-agent

# Restore configuration
echo "Restoring configuration..."
cp "$BACKUP_PATH/config.yaml" /home/hermes/.hermes/
cp "$BACKUP_PATH/SOUL.md" /home/hermes/.hermes/

# Restore credentials
echo "Restoring credentials..."
openssl enc -aes-256-cbc -d -pbkdf2 \
    -pass file:/opt/hermes/.backup_password \
    -in "$BACKUP_PATH/credentials.tar.gz.enc" | \
    tar -xzf - -C /

# Restore database
echo "Restoring database..."
if [ -f "$BACKUP_PATH/memory.db" ]; then
    cp "$BACKUP_PATH/memory.db" /home/hermes/.hermes/memory.db
fi

# Restore PostgreSQL
if [ -f "$BACKUP_PATH/postgres.sql.gz" ]; then
    gunzip < "$BACKUP_PATH/postgres.sql.gz" | psql -U hermes hermes
fi

# Restore skills
echo "Restoring skills..."
tar -xzf "$BACKUP_PATH/skills.tar.gz" -C /

# Fix permissions
chown -R hermes:hermes /home/hermes/.hermes
chown -R hermes:hermes /home/hermes/.agent
chmod 600 /home/hermes/.hermes/config.yaml
chmod 700 /home/hermes/.agent/credentials
chmod 600 /home/hermes/.agent/credentials/*

# Start service
sudo systemctl start hermes-agent

# Verify
sleep 5
sudo systemctl status hermes-agent

echo "Restore completed!"
```

## Part 6: Best Practices

### 6.1 Configuration Management

**Use environment-specific configs:**

```bash
# Development
~/.hermes/config.dev.yaml

# Staging
~/.hermes/config.staging.yaml

# Production
~/.hermes/config.prod.yaml
```

**Start with specific config:**

```bash
hermes start --config ~/.hermes/config.prod.yaml
```

### 6.2 Secrets Management

**Use HashiCorp Vault:**

```bash
# Store secrets in Vault
vault kv put secret/hermes/prod \
    telegram_token="..." \
    anthropic_key="..." \
    github_token="..."

# Retrieve at runtime
vault kv get -format=json secret/hermes/prod | \
    jq -r '.data.data' > ~/.agent/credentials/secrets.json
```

**Or use AWS Secrets Manager:**

```bash
# Store secret
aws secretsmanager create-secret \
    --name hermes/prod/credentials \
    --secret-string file://credentials.json

# Retrieve at runtime
aws secretsmanager get-secret-value \
    --secret-id hermes/prod/credentials \
    --query SecretString \
    --output text > ~/.agent/credentials/secrets.json
```

### 6.3 Cost Optimization

**1. Use cheaper models for simple tasks:**

```yaml
model:
  # Default: fast and cheap
  provider: anthropic
  model: claude-haiku-4
  
  # Fallback to powerful model for complex tasks
  fallback:
    provider: anthropic
    model: claude-sonnet-4
    trigger_on:
      - context_length > 50000
      - task_complexity: high
```

**2. Enable aggressive caching:**

```yaml
cache:
  enabled: true
  backend: redis
  ttl:
    web_search: 86400  # 24 hours
    file_content: 3600  # 1 hour
    api_response: 1800  # 30 minutes
```

**3. Rate limiting:**

```yaml
model:
  rate_limit:
    requests_per_minute: 30
    tokens_per_minute: 100000
```

**4. Monitor costs:**

```bash
# Daily cost report
hermes costs report --period daily

# Set budget alerts
hermes costs alert --daily-limit 10 --notify telegram
```

### 6.4 Performance Tuning

**1. Connection pooling:**

```yaml
database:
  pool_size: 20
  max_overflow: 10
  pool_timeout: 30
```

**2. Async operations:**

```yaml
execution:
  async: true
  max_concurrent: 10
```

**3. Resource limits:**

```yaml
resources:
  max_memory_mb: 4096
  max_cpu_percent: 80
  max_disk_mb: 10240
```

### 6.5 Security Checklist

- [ ] All credentials stored securely (encrypted at rest)
- [ ] SSL/TLS enabled for all external connections
- [ ] Firewall configured (only necessary ports open)
- [ ] SSH key authentication only (no passwords)
- [ ] Regular security updates enabled
- [ ] Fail2ban configured
- [ ] Rate limiting enabled
- [ ] IP whitelist for admin endpoints
- [ ] Audit logging enabled
- [ ] Regular backups tested
- [ ] Secrets rotation policy in place
- [ ] Monitoring and alerting configured

### 6.6 Operational Checklist

- [ ] Health checks configured
- [ ] Monitoring dashboards set up
- [ ] Alerting rules defined
- [ ] Backup automation working
- [ ] Disaster recovery tested
- [ ] Documentation up to date
- [ ] Runbooks for common issues
- [ ] On-call rotation defined
- [ ] Incident response plan
- [ ] Change management process

## Part 7: Scaling Strategies

### 7.1 Horizontal Scaling

**Load balancer configuration:**

```nginx
upstream hermes_backend {
    least_conn;
    
    # Instance 1
    server 10.0.1.10:8080 max_fails=3 fail_timeout=30s weight=1;
    
    # Instance 2
    server 10.0.1.11:8080 max_fails=3 fail_timeout=30s weight=1;
    
    # Instance 3
    server 10.0.1.12:8080 max_fails=3 fail_timeout=30s weight=1;
    
    # Health check
    check interval=3000 rise=2 fall=3 timeout=1000;
}
```

**Session affinity (if needed):**

```nginx
upstream hermes_backend {
    ip_hash;  # Route same IP to same instance
    # ... servers ...
}
```

### 7.2 Vertical Scaling

**Increase resources:**

```yaml
# systemd service
[Service]
MemoryMax=8G
CPUQuota=400%

# Docker
deploy:
  resources:
    limits:
      cpus: '4'
      memory: 8G
```

### 7.3 Auto-Scaling (Kubernetes)

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: hermes-agent-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: hermes-agent
  minReplicas: 2
  maxReplicas: 10
  metrics:
    - type: Resource
      resource:
        name: cpu
        target:
          type: Utilization
          averageUtilization: 70
    - type: Resource
      resource:
        name: memory
        target:
          type: Utilization
          averageUtilization: 80
```

## Conclusion

Production deployment requires careful planning and ongoing maintenance. Key takeaways:

1. **Security first**: Encrypt credentials, use SSL, restrict access
2. **Monitor everything**: Metrics, logs, alerts
3. **Automate backups**: Test recovery procedures regularly
4. **Plan for scale**: Start simple, scale as needed
5. **Document everything**: Runbooks, procedures, architecture

**Next steps:**
1. Set up monitoring and alerting
2. Test disaster recovery
3. Optimize costs
4. Scale as needed
5. Keep documentation updated

For production support, join the Nous Research Discord or open a GitHub issue.
