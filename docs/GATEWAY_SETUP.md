# Docker MCP Gateway Setup Guide

This guide explains how to properly set up the Docker MCP Gateway with your MCP server containers.

## Understanding the Gateway

**IMPORTANT**: The Docker MCP Gateway is **NOT** a long-running HTTP service on port 3000.

### What it IS:
- A CLI tool (`docker mcp`) that manages MCP server containers
- Spawns server containers on-demand via the Docker socket
- Listens on a UNIX socket (default: `/var/run/mcp-gateway.sock`)
- MCP clients connect via this socket, not HTTP

### What it is NOT:
- Not a web server with REST API
- Not exposed on any TCP port
- Not accessed via `http://localhost:3000`
- Not a long-running container in your compose stack

## Installation

### Option 1: Docker Desktop (Easiest)

If you have Docker Desktop installed, the gateway CLI is already available:

```bash
docker mcp --help
```

### Option 2: Standalone Binary (for Servers)

Download and install the gateway binary on your Komodo host:

```bash
# Linux AMD64
wget https://github.com/docker/mcp-gateway/releases/latest/download/docker-mcp-linux-amd64 \
  -O /usr/local/bin/docker-mcp
chmod +x /usr/local/bin/docker-mcp

# Linux ARM64
wget https://github.com/docker/mcp-gateway/releases/latest/download/docker-mcp-linux-arm64 \
  -O /usr/local/bin/docker-mcp
chmod +x /usr/local/bin/docker-mcp

# macOS AMD64
wget https://github.com/docker/mcp-gateway/releases/latest/download/docker-mcp-darwin-amd64 \
  -O /usr/local/bin/docker-mcp
chmod +x /usr/local/bin/docker-mcp
```

### Option 3: Build from Source

```bash
git clone https://github.com/docker/mcp-gateway.git
cd mcp-gateway
make build
sudo cp bin/docker-mcp /usr/local/bin/
```

## Configuration

### 1. Deploy MCP Server Containers

Your `compose.yaml` defines the server containers. Deploy them (but don't start them):

```bash
# Navigate to your stack directory
cd /path/to/HL_DockerMCPGateway

# Create the containers without starting them
docker compose up -d --no-start

# Or if using Komodo, just deploy the stack
```

The gateway will start these containers on-demand.

### 2. Enable Servers

Tell the gateway which servers to manage:

```bash
# Enable markitdown (no credentials needed)
docker mcp server enable markitdown

# Enable with specific Docker image
docker mcp server enable markitdown --image mcp/markitdown:latest

# List enabled servers
docker mcp server list
```

### 3. Configure Server Credentials (Optional)

For servers that need credentials (Proxmox, Tailscale, Atlassian), either:

**A) Use environment variables in compose.yaml** (already done)

The compose file passes credentials from `.env` to containers.

**B) Use docker mcp CLI to configure**

```bash
docker mcp server configure proxmox \
  --env PROXMOX_HOST=192.168.1.100 \
  --env PROXMOX_TOKEN_NAME=mytoken \
  --env PROXMOX_TOKEN_VALUE=secret
```

### 4. Create mcp-servers.json (Optional)

For advanced configuration, create `~/.docker/mcp-servers.json`:

```json
{
  "mcpServers": {
    "markitdown": {
      "command": "docker",
      "args": [
        "run",
        "--rm",
        "-i",
        "--network=hl_dockermcpgateway_mcp",
        "--name=mcp-markitdown",
        "-v", "/srv/mcp/workspace:/workspace:ro",
        "--security-opt=no-new-privileges:true",
        "--read-only",
        "--tmpfs=/tmp",
        "mcp/markitdown:latest"
      ]
    },
    "proxmox": {
      "command": "docker",
      "args": [
        "run",
        "--rm",
        "-i",
        "--network=hl_dockermcpgateway_mcp",
        "--name=mcp-proxmox",
        "--env-file=/path/to/.env",
        "ghcr.io/canvrno/proxmoxmcp:latest"
      ]
    }
  }
}
```

## Running the Gateway

### Option 1: Foreground (for testing)

```bash
docker mcp gateway run
```

Press Ctrl+C to stop.

### Option 2: Background Daemon

```bash
# Start in background
docker mcp gateway run --daemon

# Check status
docker mcp gateway status

# Stop
docker mcp gateway stop
```

### Option 3: Systemd Service (Production)

Create `/etc/systemd/system/docker-mcp-gateway.service`:

```ini
[Unit]
Description=Docker MCP Gateway
After=docker.service
Requires=docker.service

[Service]
Type=simple
ExecStart=/usr/local/bin/docker-mcp gateway run
Restart=on-failure
RestartSec=5s
User=root
Group=docker

[Install]
WantedBy=multi-user.target
```

Enable and start:

```bash
sudo systemctl daemon-reload
sudo systemctl enable docker-mcp-gateway
sudo systemctl start docker-mcp-gateway
sudo systemctl status docker-mcp-gateway
```

## Connecting MCP Clients

### Claude Desktop

Edit `~/Library/Application Support/Claude/claude_desktop_config.json` (macOS) or
`%APPDATA%\Claude\claude_desktop_config.json` (Windows):

```json
{
  "mcpServers": {
    "docker-gateway": {
      "command": "docker",
      "args": ["mcp", "gateway", "connect"],
      "transport": "stdio"
    }
  }
}
```

### Cursor

Edit your Cursor MCP settings to connect to the gateway socket.

### Custom Client

Connect to the UNIX socket at `/var/run/mcp-gateway.sock` (or custom path).

## Troubleshooting

### Gateway shows "No servers enabled"

```bash
# Enable at least one server
docker mcp server enable markitdown

# List enabled servers
docker mcp server list
```

### Server container fails to start

```bash
# Check server logs
docker logs mcp-markitdown

# Test manually
docker run --rm -i mcp/markitdown:latest
```

### Gateway can't access Docker socket

```bash
# Ensure gateway has permission
sudo chmod 666 /var/run/docker.sock

# Or add user to docker group
sudo usermod -aG docker $USER
```

### Client can't connect to gateway

```bash
# Verify gateway is running
docker mcp gateway status

# Check socket exists
ls -la /var/run/mcp-gateway.sock

# Test connection
echo '{"jsonrpc":"2.0","method":"initialize","params":{},"id":1}' | \
  socat - UNIX-CONNECT:/var/run/mcp-gateway.sock
```

## Advanced: Gateway in Container (Not Recommended)

If you absolutely need to run the gateway in a container:

```yaml
services:
  gateway-runner:
    image: ghcr.io/docker/mcp-gateway:master
    container_name: mcp-gateway
    network_mode: host
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
      - ./mcp-servers.json:/etc/mcp-servers.json:ro
      - mcp-gateway-data:/var/lib/mcp-gateway
    command: >
      mcp-gateway server
        --config /etc/mcp-servers.json
        --socket /var/run/mcp-gateway.sock
    profiles: ["gateway"]
```

**Not recommended** because:
- Adds unnecessary containerization layer
- Complicates networking
- Harder to debug
- The CLI approach is simpler

## Next Steps

- [Add more MCP servers](./ADDING_MCP_SERVERS.md)
- [Configure Proxmox MCP](./ADDING_MCP_SERVERS.md#proxmox-mcp)
- [Configure Tailscale MCP](./ADDING_MCP_SERVERS.md#tailscale-mcp)
- [Troubleshooting guide](./TROUBLESHOOTING.md)

## Resources

- [Official Docker MCP Gateway Docs](https://docs.docker.com/ai/mcp-catalog-and-toolkit/mcp-gateway/)
- [Docker MCP Gateway GitHub](https://github.com/docker/mcp-gateway)
- [Model Context Protocol Specification](https://modelcontextprotocol.io/)
