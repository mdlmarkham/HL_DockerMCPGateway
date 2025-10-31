# Komodo Setup Guide

This guide will help you deploy the MCP Gateway Stack using Komodo.

## Prerequisites

- Komodo installed and running
- Docker and Docker Compose available on the host
- Network access configured

## Deployment Steps

### Method 1: Using Komodo UI (Stack)

1. **Navigate to Stacks**
   - Open Komodo dashboard
   - Go to **Stacks** section

2. **Create New Stack**
   - Click **New Stack**
   - Name: `mcp-gateway`
   - Description: `MCP Gateway with Markitdown server`

3. **Configure Stack**
   - **Repository**: Point to your Git repository (or use local file)
   - **Compose File**: `compose.yaml`
   - **Working Directory**: `/opt/komodo/stacks/mcp-gateway`

4. **Set Environment Variables**
   - Add environment variables from `.env.example`
   - Update `MCP_WORKSPACE_PATH` to your preferred location

5. **Deploy**
   - Click **Deploy Stack**
   - Monitor logs for successful startup

### Method 2: Using Komodo CLI

```bash
# Clone the repository
git clone https://github.com/YOUR-USERNAME/HL_DockerMCPGateway.git
cd HL_DockerMCPGateway

# Create environment file
cp .env.example .env
# Edit .env with your configuration

# Deploy using Komodo
komodo stack deploy --name mcp-gateway --file compose.yaml
```

### Method 3: Direct Docker Compose

If you prefer to manage it manually outside Komodo's UI:

```bash
cd /opt/komodo/stacks/mcp-gateway
docker compose up -d
```

## Network Configuration

### Port Exposure

By default, the stack exposes:
- **6277**: MCP Gateway endpoint
- **6274**: Inspector/UI endpoint (optional)

### Firewall Rules

If using pfSense or another firewall with Komodo:

1. Create firewall rules to allow access from Tailscale network
2. Block external access to these ports
3. Allow internal Docker network communication

Example pfSense rule:
```
Interface: LAN
Protocol: TCP
Source: Tailscale subnet (e.g., 100.64.0.0/10)
Destination: Komodo host
Destination Port: 6274, 6277
Action: Allow
```

## Monitoring

### Check Stack Status

In Komodo:
1. Go to **Stacks** → **mcp-gateway**
2. View running services
3. Check resource usage

### View Logs

```bash
# Via Komodo UI
Stack → mcp-gateway → Logs

# Via CLI
komodo stack logs mcp-gateway

# Via Docker
docker compose logs -f
```

## Updates

### Update Stack

1. **Via Komodo UI**:
   - Stacks → mcp-gateway → **Update Stack**
   - Pull latest images
   - Redeploy

2. **Via CLI**:
   ```bash
   cd /path/to/stack
   git pull
   docker compose pull
   docker compose up -d
   ```

## Managing MCP Servers

### Adding Servers from the Catalog

Since the Gateway runs in a container, you need to execute MCP commands **inside the container**:

#### Via Komodo UI:

1. Navigate to **Stacks** → **mcp-gateway**
2. Click on the **mcp-gateway** container
3. Open the **Terminal** tab
4. Run commands:
   ```bash
   # List available servers
   docker mcp server catalog
   
   # Enable a server
   docker mcp server enable atlassian
   
   # Configure it (interactive)
   docker mcp server configure atlassian
   
   # List enabled servers
   docker mcp server list
   ```

#### Via Command Line:

```bash
# From your Komodo host, execute commands in the Gateway container
docker exec mcp-gateway docker mcp server catalog
docker exec mcp-gateway docker mcp server enable atlassian
docker exec -it mcp-gateway docker mcp server configure atlassian
docker exec mcp-gateway docker mcp server list
```

#### Popular Servers to Enable:

```bash
# Atlassian (Jira + Confluence) - 37 tools
docker exec mcp-gateway docker mcp server enable atlassian

# Obsidian vault management - 12 tools
docker exec mcp-gateway docker mcp server enable obsidian

# Reddit integration - 6 tools
docker exec mcp-gateway docker mcp server enable mcp-reddit

# Context7 library docs - 2 tools
docker exec mcp-gateway docker mcp server enable context7

# OpenAPI Schema analysis - 10 tools
docker exec mcp-gateway docker mcp server enable openapi-schema

# Wikipedia knowledge - 11 tools
docker exec mcp-gateway docker mcp server enable wikipedia-mcp

# Komodo management - 15 tools (manage THIS Komodo!)
docker exec mcp-gateway docker mcp server enable komodo-mcp

# Proxmox hypervisor - 6 tools
docker exec mcp-gateway docker mcp server enable proxmox-mcp

# Tailscale network - 20+ tools
docker exec mcp-gateway docker mcp server enable tailscale-mcp

# GitHub integration
docker exec mcp-gateway docker mcp server enable github

# Web browsing
docker exec mcp-gateway docker mcp server enable web-browse

# PostgreSQL
docker exec mcp-gateway docker mcp server enable postgres
```

### Server Configuration

Some servers require credentials (API tokens, URLs, etc.). You can:

1. **Interactive Configuration**: Use `docker mcp server configure <server>`
2. **Environment Variables**: Add to `.env` file and update stack
3. **Secrets Management**: Use Docker secrets or Komodo's secret management

Example for Atlassian in `.env`:
```bash
CONFLUENCE_URL=https://your-company.atlassian.net/wiki
CONFLUENCE_USERNAME=your.email@company.com
CONFLUENCE_API_TOKEN=your_token_here
JIRA_URL=https://your-company.atlassian.net
JIRA_USERNAME=your.email@company.com
JIRA_API_TOKEN=your_token_here
```

Then pass these to the server via Komodo's environment variable management.

## Auto-Start on Boot

Komodo should automatically restart stacks on system reboot. Verify:

1. Check restart policy in `compose.yaml`: `restart: unless-stopped`
2. Ensure Komodo service starts on boot
3. Test by rebooting the host

## Troubleshooting

### Stack Won't Start

1. Check Komodo logs: `/var/log/komodo/`
2. Verify Docker is running: `systemctl status docker`
3. Check compose file syntax: `docker compose config`

### Ports Already in Use

```bash
# Check what's using the ports
sudo lsof -i :6277
sudo lsof -i :6274

# Stop conflicting services or change ports in compose.yaml
```

### Permission Issues

```bash
# Ensure Komodo has access to Docker socket
sudo usermod -aG docker komodo
sudo systemctl restart komodo
```

## Best Practices

1. **Use Git**: Keep your compose.yaml in version control
2. **Environment Variables**: Never commit `.env` files
3. **Backups**: Backup the `mcp-gateway-data` volume regularly
4. **Updates**: Keep images updated for security patches
5. **Monitoring**: Set up alerts for container failures in Komodo

## Additional Resources

- [Komodo Documentation](https://github.com/mbecker20/komodo)
- [Docker Compose Reference](https://docs.docker.com/compose/)
- [MCP Gateway Documentation](https://github.com/docker/mcp-gateway)
