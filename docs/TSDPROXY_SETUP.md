# tsdproxy Integration Guide

This guide explains how to expose the MCP Gateway through your existing `tsdproxy` setup instead of using a Tailscale sidecar container.

## Architecture Overview

```
┌─────────────────────────────────────────────────────────────┐
│ Komodo Host (Already on Tailscale)                          │
│                                                              │
│  ┌──────────────┐         ┌─────────────────────────────┐   │
│  │   tsdproxy   │────────▶│   MCP Gateway Container     │   │
│  │ (Host Process│         │   Port: 3000                │   │
│  │  on TS Net)  │         │   Network: mcp (bridge)     │   │
│  └──────────────┘         └─────────────────────────────┘   │
│         │                              │                     │
│         │                              │                     │
│         │                              ▼                     │
│         │                  ┌────────────────────────┐        │
│         │                  │  MCP Servers           │        │
│         │                  │  - markitdown          │        │
│         │                  │  - atlassian           │        │
│         │                  │  - proxmox-mcp         │        │
│         │                  │  - tailscale-mcp       │        │
│         │                  └────────────────────────┘        │
│         │                                                    │
│  Tailscale Magic DNS:                                        │
│  https://mcp-gateway.your-tailnet.ts.net                     │
└─────────────────────────────────────────────────────────────┘
```

## Benefits of tsdproxy

✅ **Simpler Architecture**: No sidecar containers needed  
✅ **Resource Efficient**: Leverages host's existing Tailscale connection  
✅ **Native Integration**: Works seamlessly with your Tailscale setup  
✅ **Automatic TLS**: tsdproxy handles HTTPS certificates  
✅ **No Auth Keys**: No need for TS_AUTHKEY since host is already authenticated  

## Prerequisites

- Komodo host already connected to your Tailscale network
- `tsdproxy` installed and running on the host
- Docker and Docker Compose installed
- MCP Gateway repository cloned

## Configuration Steps

### 1. Configure tsdproxy

Add the MCP Gateway to your tsdproxy configuration. The exact method depends on how you're running tsdproxy:

#### Option A: tsdproxy with systemd service

Edit your tsdproxy configuration (typically `/etc/tsdproxy/config.yaml` or similar):

```yaml
backends:
  mcp-gateway:
    url: http://localhost:3000
    hostname: mcp-gateway.your-tailnet.ts.net
```

Then restart tsdproxy:

```bash
sudo systemctl restart tsdproxy
```

#### Option B: tsdproxy command line

If running tsdproxy directly, add the backend:

```bash
tsdproxy \
  --backend mcp-gateway=http://localhost:3000 \
  --hostname mcp-gateway.your-tailnet.ts.net
```

#### Option C: tsdproxy via Docker

If you're running tsdproxy in Docker (separate from MCP stack):

```yaml
services:
  tsdproxy:
    image: tailscale/tsdproxy:latest
    container_name: tsdproxy
    restart: unless-stopped
    network_mode: host
    environment:
      - TS_AUTHKEY=${TS_AUTHKEY}
      - TS_HOSTNAME=mcp-gateway
      - TS_EXTRA_ARGS=--advertise-tags=tag:proxy
    command:
      - /usr/local/bin/tsdproxy
      - --backend=mcp-gateway=http://localhost:3000
    volumes:
      - tsdproxy-state:/var/lib/tailscale
    cap_add:
      - NET_ADMIN
      - SYS_MODULE
```

### 2. Update Environment Variables (Optional)

Create `.env` file if you haven't already:

```bash
cp .env.example .env
```

Edit `.env` and set the gateway port (default is 3000):

```bash
# MCP Gateway Configuration
MCP_GATEWAY_PORT=3000
```

### 3. Start the Stack

```bash
# Start the gateway and base services
docker compose up -d

# Start optional services with profiles
docker compose --profile proxmox --profile tailscale-mcp up -d
```

### 4. Verify the Setup

Check that the gateway is running:

```bash
docker compose ps
docker compose logs mcp-gateway
```

Test the gateway locally:

```bash
curl http://localhost:3000/health
```

Test via Tailscale:

```bash
curl https://mcp-gateway.your-tailnet.ts.net/health
```

## Troubleshooting

### Gateway not accessible via Tailscale

1. **Check tsdproxy logs**:
   ```bash
   sudo journalctl -u tsdproxy -f
   # or
   docker logs tsdproxy
   ```

2. **Verify hostname**:
   ```bash
   tailscale status | grep mcp-gateway
   ```

3. **Test local connectivity**:
   ```bash
   curl http://localhost:3000/health
   ```

### Port conflicts

If port 3000 is already in use, change it in `.env`:

```bash
MCP_GATEWAY_PORT=3001
```

Then recreate the container:

```bash
docker compose up -d --force-recreate mcp-gateway
```

And update your tsdproxy backend URL to match.

### MCP Servers not discovered

Check the gateway can access Docker:

```bash
docker compose exec mcp-gateway ls -la /var/run/docker.sock
```

Check the gateway logs:

```bash
docker compose logs mcp-gateway | grep -i discover
```

## Tailscale ACLs

If using Tailscale ACLs, ensure the MCP Gateway host can be accessed:

```json
{
  "acls": [
    {
      "action": "accept",
      "src": ["group:developers"],
      "dst": ["tag:proxy:*"]
    }
  ],
  "tagOwners": {
    "tag:proxy": ["your-admin@example.com"]
  }
}
```

## Migrating from Sidecar

If you previously had the Tailscale sidecar setup:

1. **Stop the old stack**:
   ```bash
   docker compose down
   ```

2. **The compose.yaml has already been updated** to remove:
   - `tailscale-gateway` service
   - `tailscale-mcp-gateway` and `tailscale-mcp-markitdown` volumes
   - `network_mode: service:tailscale-gateway` from mcp-gateway
   - `depends_on: tailscale-gateway` from mcp-gateway

3. **Remove Tailscale-related environment variables** from `.env`:
   - `TS_AUTHKEY` (no longer needed)
   - `TS_EXTRA_ARGS` (no longer needed)

4. **Configure tsdproxy** (see steps above)

5. **Start the new stack**:
   ```bash
   docker compose up -d
   ```

## Next Steps

- [Add MCP servers](./ADDING_MCP_SERVERS.md) from the catalog
- [Configure Proxmox MCP](./ADDING_MCP_SERVERS.md#proxmox-mcp) for hypervisor management
- [Configure Tailscale MCP](./ADDING_MCP_SERVERS.md#tailscale-mcp) for network automation
- [Set up monitoring](./MONITORING.md) for your MCP stack

## Additional Resources

- [tsdproxy Documentation](https://github.com/tailscale/tsdproxy)
- [Tailscale Docker Guide](https://tailscale.com/kb/1282/docker/)
- [MCP Gateway Documentation](https://modelcontextprotocol.io/docs/gateway)
