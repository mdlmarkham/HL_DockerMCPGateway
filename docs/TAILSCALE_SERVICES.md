# Tailscale Services Configuration Guide

This guide explains how to expose your MCP Gateway Stack as Tailscale Services, making them easily discoverable and accessible within your tailnet.

## What are Tailscale Services?

Tailscale Services is a feature that automatically discovers and displays network-exposed ports running on your Tailscale machines. When properly configured, your MCP Gateway will appear in the Tailscale admin console with clickable links and proper categorization.

## Overview

This deployment uses **Tailscale Serve** to expose the MCP Gateway securely over HTTPS within your tailnet. Benefits include:

- ✅ **Automatic HTTPS**: TLS certificates provisioned automatically
- ✅ **MagicDNS**: Access via friendly hostnames (e.g., `https://mcp-gateway.tail1234.ts.net`)
- ✅ **No Port Forwarding**: No need to expose ports on your firewall
- ✅ **Service Discovery**: Appears in Tailscale Services list
- ✅ **Access Control**: Use Tailscale ACLs to control who can access the service

## Architecture

```
┌─────────────────────────────────────────────────────┐
│                  Tailscale Network                   │
│                                                      │
│  ┌────────────────┐         ┌──────────────────┐   │
│  │  MCP Clients   │────────▶│  mcp-gateway     │   │
│  │ (Claude, etc.) │  HTTPS  │  .tail1234.ts.net│   │
│  └────────────────┘         └──────────────────┘   │
│                                     │               │
│                              ┌──────▼────────┐      │
│                              │   Tailscale   │      │
│                              │    Sidecar    │      │
│                              └──────┬────────┘      │
│                                     │               │
│                              ┌──────▼────────┐      │
│                              │  MCP Gateway  │      │
│                              │  Container    │      │
│                              └───────────────┘      │
└─────────────────────────────────────────────────────┘
```

## Prerequisites

1. **Tailscale Account**: Sign up at https://tailscale.com
2. **Tailnet Created**: Your private network
3. **Auth Key Generated**: From https://login.tailscale.com/admin/settings/keys
4. **Services Enabled**: Enable at https://login.tailscale.com/admin/services

## Setup Instructions

### Step 1: Enable Tailscale Services

1. Go to https://login.tailscale.com/admin/services
2. Click **Enable services collection**
3. This allows your devices to share their exposed services

### Step 2: Create a Tailscale Auth Key

1. Navigate to https://login.tailscale.com/admin/settings/keys
2. Click **Generate auth key**
3. Configure the key:
   - ✅ **Reusable**: Check this box (allows multiple containers)
   - ✅ **Ephemeral**: Uncheck (for persistent services)
   - ✅ **Pre-approved**: Check (automatically authorizes)
   - **Tags**: Add `tag:mcp-server` (create if needed)
   - **Expiration**: Set to a long duration or never

4. Copy the generated key (starts with `tskey-auth-`)

### Step 3: Create Access Control Tags

Add tags to your Tailscale ACL policy at https://login.tailscale.com/admin/acls:

```json
{
  "tagOwners": {
    "tag:mcp-server": ["autogroup:admin"]
  },
  "acls": [
    {
      "action": "accept",
      "src": ["autogroup:member"],
      "dst": ["tag:mcp-server:*"]
    }
  ]
}
```

This allows all tailnet members to access MCP servers.

### Step 4: Configure Environment Variables

1. Copy the example environment file:
   ```bash
   cp .env.example .env
   ```

2. Edit `.env` and add your Tailscale auth key:
   ```bash
   TS_AUTHKEY=tskey-auth-xxxxxxxxxxxxxxxxxxxxxxxxxxxxx
   TS_CERT_DOMAIN=tail1234.ts.net  # Your tailnet domain
   MCP_WORKSPACE_PATH=/srv/mcp/workspace
   ```

3. Get your tailnet domain:
   ```bash
   # It's shown in your Tailscale admin console URL
   # Example: tail1234.ts.net
   ```

### Step 5: Configure Tailscale Serve

The `tailscale/serve-config.json` file defines how services are exposed. Default configuration:

```json
{
  "TCP": {
    "443": {
      "HTTPS": true
    }
  },
  "Web": {
    "mcp-gateway.${TS_CERT_DOMAIN}:443": {
      "Handlers": {
        "/": {
          "Proxy": "http://127.0.0.1:6277"
        },
        "/inspector": {
          "Proxy": "http://127.0.0.1:6274"
        }
      }
    }
  }
}
```

This configuration:
- Exposes HTTPS on port 443
- Routes traffic to MCP Gateway on port 6277
- Provides an inspector endpoint at `/inspector`

### Step 6: Deploy the Stack

```bash
# Start the services
docker compose up -d

# Check logs
docker compose logs -f tailscale-gateway

# Verify Tailscale connection
docker exec tailscale-mcp-gateway tailscale status
```

### Step 7: Verify Service Exposure

1. **Check Tailscale Serve Status**:
   ```bash
   docker exec tailscale-mcp-gateway tailscale serve status
   ```

   Output should show:
   ```
   https://mcp-gateway.tail1234.ts.net (tailnet only)
   |-- / http://127.0.0.1:6277
   |-- /inspector http://127.0.0.1:6274
   ```

2. **View in Admin Console**:
   - Go to https://login.tailscale.com/admin/machines
   - Click on your `mcp-gateway` machine
   - Scroll to the **Services** section
   - You should see HTTPS service on port 443

3. **Test Access**:
   ```bash
   # From any device on your tailnet
   curl https://mcp-gateway.tail1234.ts.net
   ```

## Client Configuration

### Claude Desktop

Edit your MCP configuration file (typically `~/Library/Application Support/Claude/claude_desktop_config.json` on macOS):

```json
{
  "mcpServers": {
    "gateway": {
      "url": "https://mcp-gateway.tail1234.ts.net",
      "timeout": 30000
    }
  }
}
```

### Cursor / VS Code

Add to your settings:

```json
{
  "mcp.servers": {
    "gateway": {
      "command": "mcp-client",
      "args": ["--url", "https://mcp-gateway.tail1234.ts.net"]
    }
  }
}
```

### GitHub Copilot

```json
{
  "github.copilot.mcp.gateway": "https://mcp-gateway.tail1234.ts.net"
}
```

## Advanced Configuration

### Using Tailscale CLI for Dynamic Configuration

Instead of using a serve config file, you can configure services dynamically:

```bash
# Enter the Tailscale container
docker exec -it tailscale-mcp-gateway sh

# Configure serve
tailscale serve https:443 / http://127.0.0.1:6277

# Check status
tailscale serve status

# Add additional path
tailscale serve https:443 /inspector http://127.0.0.1:6274
```

### Multiple MCP Servers

To expose multiple MCP servers as separate services:

```yaml
services:
  # Add another Tailscale sidecar for a different service
  tailscale-markitdown:
    image: tailscale/tailscale:latest
    container_name: tailscale-mcp-markitdown
    hostname: mcp-markitdown
    environment:
      - TS_AUTHKEY=${TS_AUTHKEY}
      - TS_EXTRA_ARGS=--advertise-tags=tag:mcp-server
    # ... rest of config

  markitdown-exposed:
    image: mcp/markitdown:latest
    network_mode: service:tailscale-markitdown
    # ... rest of config
```

### Using Tailscale Funnel (Public Internet Access)

⚠️ **Warning**: This exposes your service to the public internet. Only use if absolutely necessary.

1. Enable Funnel in admin console: https://login.tailscale.com/admin/settings/funnel
2. Modify the serve config or use:
   ```bash
   docker exec tailscale-mcp-gateway tailscale funnel 443
   ```

## Access Control

### Restrict Access to Specific Users

Update your ACL policy:

```json
{
  "acls": [
    {
      "action": "accept",
      "src": ["user1@example.com", "user2@example.com"],
      "dst": ["tag:mcp-server:443"]
    }
  ]
}
```

### Restrict Access to Specific Groups

```json
{
  "groups": {
    "group:developers": ["user1@example.com", "user2@example.com"]
  },
  "acls": [
    {
      "action": "accept",
      "src": ["group:developers"],
      "dst": ["tag:mcp-server:443"]
    }
  ]
}
```

### Service-Specific ACLs

```json
{
  "acls": [
    {
      "action": "accept",
      "src": ["group:developers"],
      "dst": ["tag:mcp-server:443"],
      "proto": "tcp"
    }
  ]
}
```

## Monitoring and Debugging

### Check Tailscale Connection

```bash
docker exec tailscale-mcp-gateway tailscale status
```

### View Tailscale Logs

```bash
docker compose logs -f tailscale-gateway
```

### Test Service Accessibility

```bash
# From another device on your tailnet
tailscale ping mcp-gateway

# Test HTTPS
curl -v https://mcp-gateway.tail1234.ts.net
```

### Check Service Discovery

1. Go to https://login.tailscale.com/admin/services
2. Filter by type: HTTPS
3. Look for your `mcp-gateway` service

### Debug Serve Configuration

```bash
docker exec tailscale-mcp-gateway tailscale serve status
```

## Troubleshooting

### Service Not Appearing in Admin Console

1. **Verify services collection is enabled**:
   - https://login.tailscale.com/admin/services

2. **Check Tailscale is running**:
   ```bash
   docker ps | grep tailscale
   ```

3. **Verify serve is active**:
   ```bash
   docker exec tailscale-mcp-gateway tailscale serve status
   ```

4. **Check ACLs**:
   - Ensure the device isn't blocking incoming connections
   - Verify ACL allows traffic to tag:mcp-server

### Cannot Access Service from Client

1. **Verify tailnet connectivity**:
   ```bash
   tailscale ping mcp-gateway
   ```

2. **Check DNS resolution**:
   ```bash
   nslookup mcp-gateway.tail1234.ts.net
   ```

3. **Test without HTTPS**:
   ```bash
   curl http://100.x.y.z:6277  # Use Tailscale IP directly
   ```

4. **Check firewall rules**:
   - Ensure Docker allows container network access
   - Verify Tailscale isn't blocked

### Certificate Issues

Tailscale automatically provisions certificates. If you see cert errors:

1. **Verify MagicDNS is enabled**:
   - https://login.tailscale.com/admin/dns

2. **Check certificate domain**:
   ```bash
   docker exec tailscale-mcp-gateway tailscale cert status
   ```

3. **Regenerate certificates**:
   ```bash
   docker exec tailscale-mcp-gateway tailscale serve reset
   docker exec tailscale-mcp-gateway tailscale serve https:443 / http://127.0.0.1:6277
   ```

### High Latency or Slow Connections

1. **Check if using DERP relay**:
   ```bash
   docker exec tailscale-mcp-gateway tailscale netcheck
   ```

2. **Enable direct connections**:
   - Ensure UDP is not blocked
   - Check firewall rules for UDP 41641

3. **Optimize DERP settings**:
   - Use nearest DERP server
   - Consider running a custom DERP server

## Performance Optimization

### Resource Limits

Add resource limits to prevent excessive usage:

```yaml
services:
  tailscale-gateway:
    deploy:
      resources:
        limits:
          cpus: '0.5'
          memory: 256M
        reservations:
          cpus: '0.25'
          memory: 128M
```

### Connection Pooling

For high-traffic deployments, consider using connection pooling in your MCP clients.

### Caching

Enable caching for static content:

```bash
tailscale serve https:443 --set-header "Cache-Control: public, max-age=3600" / http://127.0.0.1:6277
```

## Security Best Practices

1. ✅ **Use ephemeral auth keys** for temporary deployments
2. ✅ **Rotate auth keys regularly** (every 90 days)
3. ✅ **Use tags** instead of user-based authentication for services
4. ✅ **Enable audit logging** in Tailscale admin console
5. ✅ **Review ACLs regularly** to ensure least privilege
6. ✅ **Monitor service access logs**
7. ✅ **Never use Funnel** unless absolutely necessary
8. ✅ **Keep Tailscale updated** to the latest stable version

## Migration from Direct Port Exposure

If you're migrating from the previous setup with direct port exposure:

### Old Configuration (compose.yaml)
```yaml
services:
  mcp-gateway:
    ports:
      - "6277:6277"
      - "6274:6274"
```

### New Configuration (with Tailscale)
```yaml
services:
  tailscale-gateway:
    # Tailscale sidecar configuration
  
  mcp-gateway:
    network_mode: service:tailscale-gateway
    # No ports section needed!
```

### Client Migration

**Before** (direct IP):
```json
{
  "url": "http://100.x.y.z:6277"
}
```

**After** (Tailscale Serve):
```json
{
  "url": "https://mcp-gateway.tail1234.ts.net"
}
```

Benefits:
- ✅ HTTPS instead of HTTP
- ✅ Friendly hostname instead of IP
- ✅ Automatic certificate management
- ✅ Better access control
- ✅ Service discovery in admin console

## Additional Resources

- [Tailscale Serve Documentation](https://tailscale.com/kb/1312/serve)
- [Tailscale Services Feature](https://tailscale.com/kb/1100/services)
- [Tailscale Docker Guide](https://tailscale.com/kb/1282/docker)
- [Tailscale ACL Documentation](https://tailscale.com/kb/1337/policy-syntax)
- [MagicDNS Setup](https://tailscale.com/kb/1081/magicdns)

## Support

For issues specific to:
- **Tailscale**: https://tailscale.com/contact/support
- **MCP Gateway**: https://github.com/docker/mcp-gateway/issues
- **This Setup**: https://github.com/YOUR-USERNAME/HL_DockerMCPGateway/issues
