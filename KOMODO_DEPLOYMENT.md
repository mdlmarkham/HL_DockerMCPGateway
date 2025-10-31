# Komodo Deployment Guide

This guide covers deploying MCPJungle to Komodo for centralized HomeLab management.

## Prerequisites

- Komodo instance running
- Git repository access from Komodo server
- `/srv/mcp/workspace` directory on Komodo host (or update path in compose.yaml)

## Method 1: Deploy from Git Repository (Recommended)

### Step 1: Add Stack in Komodo

1. Navigate to **Stacks** in Komodo UI
2. Click **Create Stack**
3. Configure:
   - **Name**: `mcpjungle`
   - **Type**: Git Repository
   - **Git URL**: `https://github.com/mdlmarkham/HL_DockerMCPGateway.git`
   - **Branch**: `main`
   - **Compose File**: `compose.yaml`

### Step 2: Configure Environment Variables

In the stack settings, add environment variables:

```bash
# Required: Set a secure password
POSTGRES_PASSWORD=your_very_secure_password_here

# Optional: Customize server mode
SERVER_MODE=development  # or 'enterprise' for multi-user

# Optional: Change default port
HOST_PORT=8080

# Optional: Use specific image tag
MCPJUNGLE_IMAGE_TAG=latest-stdio

# Optional: Enable metrics
OTEL_ENABLED=false
```

### Step 3: Deploy Stack

1. Click **Deploy** in Komodo
2. Wait for PostgreSQL healthcheck to pass
3. MCPJungle will start automatically after DB is healthy
4. Check logs in Komodo to verify startup

### Step 4: Verify Deployment

```bash
# From Komodo terminal or SSH to host
curl http://localhost:8080/health

# Should return healthy status
```

## Method 2: Deploy from Local Compose File

### Step 1: Upload Repository

1. Clone repository to Komodo server:
   ```bash
   cd /opt/stacks
   git clone https://github.com/mdlmarkham/HL_DockerMCPGateway.git
   cd HL_DockerMCPGateway
   ```

2. Create `.env` file:
   ```bash
   cp .env.example .env
   nano .env  # Edit with your secure password
   ```

### Step 2: Add Stack in Komodo

1. Navigate to **Stacks** in Komodo UI
2. Click **Create Stack**
3. Configure:
   - **Name**: `mcpjungle`
   - **Type**: Local Compose File
   - **File Path**: `/opt/stacks/HL_DockerMCPGateway/compose.yaml`

### Step 3: Deploy

1. Click **Deploy**
2. Monitor logs in Komodo

## Post-Deployment Setup

### 1. Install MCPJungle CLI (on your local machine)

```bash
# macOS/Linux
brew install mcpjungle/mcpjungle/mcpjungle

# Windows - download from releases
# https://github.com/mcpjungle/MCPJungle/releases
```

### 2. Register Your First MCP Server

```bash
# Example: Register context7 documentation server
mcpjungle register \
  --name context7 \
  --description "Up-to-date library documentation" \
  --url https://mcp.context7.com/mcp

# Verify registration
mcpjungle list servers
```

### 3. Configure MCP Client

**For Claude Desktop:**

Edit `claude_desktop_config.json`:

```json
{
  "mcpServers": {
    "mcpjungle": {
      "command": "npx",
      "args": [
        "mcp-remote",
        "http://your-komodo-host:8080/mcp",
        "--allow-http"
      ]
    }
  }
}
```

**For Cursor:**

```json
{
  "mcpServers": {
    "mcpjungle": {
      "url": "http://your-komodo-host:8080/mcp"
    }
  }
}
```

## Accessing MCPJungle

### Local Network Access

- **Gateway Endpoint**: `http://komodo-host:8080/mcp`
- **Health Check**: `http://komodo-host:8080/health`
- **Metrics** (if enabled): `http://komodo-host:8080/metrics`

### Secure Remote Access (via Tailscale)

If your Komodo host is on Tailscale:

```bash
# Get Tailscale hostname
tailscale status

# Access via Tailscale
http://your-komodo-host.tailnet-name.ts.net:8080/mcp
```

Or set up Tailscale Serve for HTTPS:

```bash
# On Komodo host
tailscale serve --bg 8080
```

Then access via: `https://your-komodo-host.tailnet-name.ts.net`

## Managing the Stack in Komodo

### View Logs

1. Navigate to **Stacks** → **mcpjungle**
2. Click **Logs**
3. Select service:
   - `db` - PostgreSQL logs
   - `mcpjungle` - Gateway logs

### Update Stack

When you push changes to the Git repository:

1. Navigate to stack in Komodo
2. Click **Pull** to fetch latest changes
3. Click **Deploy** to apply updates

### Restart Services

```bash
# Restart entire stack
# In Komodo: Click "Restart" on stack

# Or via CLI on Komodo host
cd /path/to/stack
docker compose restart

# Restart specific service
docker compose restart mcpjungle
```

### Scale or Modify

1. Edit `compose.yaml` in repository
2. Commit and push changes
3. Pull and redeploy in Komodo

## Backup and Restore

### Backup PostgreSQL Data

```bash
# Create backup
docker exec mcpjungle-db pg_dump -U mcpjungle mcpjungle > mcpjungle-backup-$(date +%Y%m%d).sql

# Or backup volume
docker run --rm -v hl_dockermcpgateway_db_data:/data -v $(pwd):/backup alpine tar czf /backup/db-backup.tar.gz -C /data .
```

### Restore from Backup

```bash
# Restore from SQL dump
docker exec -i mcpjungle-db psql -U mcpjungle mcpjungle < mcpjungle-backup.sql

# Or restore volume
docker run --rm -v hl_dockermcpgateway_db_data:/data -v $(pwd):/backup alpine tar xzf /backup/db-backup.tar.gz -C /data
```

## Monitoring in Komodo

### Health Checks

Komodo will automatically monitor container health based on healthcheck definitions in compose.yaml.

### Alerts

Configure Komodo alerts for:
- Container restart events
- Health check failures
- Resource usage thresholds

### Resource Usage

Monitor in Komodo dashboard:
- CPU usage
- Memory usage
- Network traffic
- Disk I/O

## Troubleshooting

### Stack Won't Start

**Check logs in Komodo:**
1. Navigate to stack
2. View logs for each service
3. Look for error messages

**Common issues:**

```bash
# Database not ready
# Solution: Wait for healthcheck, or restart stack

# Port 8080 already in use
# Solution: Change HOST_PORT in environment variables

# Docker socket permission denied
# Solution: Ensure Komodo has Docker socket access
```

### Can't Connect to Gateway

```bash
# Test from Komodo host
docker exec mcpjungle wget -qO- http://localhost:8080/health

# Check if port is exposed
docker ps | grep mcpjungle

# Check firewall rules
sudo ufw status
```

### Database Connection Issues

```bash
# Check database health
docker exec mcpjungle-db pg_isready -U mcpjungle

# View database logs
docker logs mcpjungle-db

# Restart database
docker compose restart db
```

## Security Considerations

### 1. Use Strong PostgreSQL Password

Always set `POSTGRES_PASSWORD` to a strong, unique value:

```bash
# Generate secure password
openssl rand -base64 32
```

### 2. Restrict Network Access

If not using Tailscale, restrict access via firewall:

```bash
# Only allow from specific IPs
sudo ufw allow from 192.168.1.0/24 to any port 8080
```

### 3. Enable Enterprise Mode

For production or multi-user environments:

```bash
# Set in Komodo environment variables
SERVER_MODE=enterprise
```

Then initialize admin user:

```bash
# From your local machine with CLI
mcpjungle init-server
```

### 4. Regular Backups

Schedule regular PostgreSQL backups in Komodo or via cron:

```bash
# Add to Komodo host crontab
0 2 * * * docker exec mcpjungle-db pg_dump -U mcpjungle mcpjungle > /backup/mcpjungle-$(date +\%Y\%m\%d).sql
```

## Advanced Configuration

### Custom Workspace Path

If you want to use a different workspace path:

1. Edit `compose.yaml` volume mount:
   ```yaml
   - /your/custom/path:/host:rw
   ```

2. Update and redeploy in Komodo

### Resource Limits

Add resource constraints in `compose.yaml`:

```yaml
services:
  mcpjungle:
    # ... existing config ...
    deploy:
      resources:
        limits:
          cpus: '2'
          memory: 2G
        reservations:
          cpus: '0.5'
          memory: 512M
```

## Support

- **MCPJungle Issues**: https://github.com/mcpjungle/MCPJungle/issues
- **MCPJungle Discord**: https://discord.gg/CapV4Z3krk
- **Komodo Documentation**: https://komo.do/docs

---

**Ready to deploy!** Follow the steps above to get MCPJungle running on your Komodo instance.
