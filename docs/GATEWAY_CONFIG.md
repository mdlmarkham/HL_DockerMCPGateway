# MCP Gateway Configuration Guide

This guide explains how to configure MCP servers for the Docker MCP Gateway.

## Configuration Methods

The MCP Gateway supports multiple ways to configure servers:

### Method 1: Manual Configuration File (Recommended for Docker)

Create or edit `mcp-config.json` in the project root with your server definitions.

**Example: Basic markitdown server**

```json
{
  "mcpServers": {
    "markitdown": {
      "command": "docker",
      "args": [
        "exec",
        "-i",
        "mcp-markitdown",
        "markitdown-mcp"
      ],
      "transport": "stdio"
    }
  }
}
```

**Example: Multiple servers**

```json
{
  "mcpServers": {
    "markitdown": {
      "command": "docker",
      "args": ["exec", "-i", "mcp-markitdown", "markitdown-mcp"],
      "transport": "stdio"
    },
    "proxmox": {
      "command": "docker",
      "args": ["exec", "-i", "mcp-proxmox", "proxmox-mcp"],
      "transport": "stdio",
      "env": {
        "PROXMOX_HOST": "${PROXMOX_HOST}",
        "PROXMOX_TOKEN_NAME": "${PROXMOX_TOKEN_NAME}",
        "PROXMOX_TOKEN_VALUE": "${PROXMOX_TOKEN_VALUE}"
      }
    },
    "tailscale": {
      "command": "docker",
      "args": ["exec", "-i", "mcp-tailscale", "node", "/app/dist/index.js"],
      "transport": "stdio",
      "env": {
        "TAILSCALE_API_KEY": "${TAILSCALE_MCP_API_KEY}",
        "TAILSCALE_TAILNET": "${TAILSCALE_MCP_TAILNET}"
      }
    }
  }
}
```

### Method 2: Gateway CLI (Interactive)

If you need to configure servers interactively:

```bash
# Enter the gateway container
docker exec -it mcp-gateway sh

# Use the gateway CLI (if available)
mcp-gateway server add markitdown --command "docker exec -i mcp-markitdown markitdown-mcp"
```

### Method 3: Docker Compose Override

For complex setups, you can create a `compose.override.yaml`:

```yaml
services:
  mcp-gateway:
    environment:
      - MCP_SERVERS=markitdown,proxmox,tailscale
```

## Configuration File Location

The `mcp-config.json` file should be placed in the project root and is automatically mounted into the gateway container at `/app/mcp-config.json`.

## Server Definition Format

Each server in the configuration must specify:

- **name**: Unique identifier for the server
- **command**: The executable command (typically `docker`)
- **args**: Array of arguments to pass to the command
- **transport**: Communication protocol (`stdio` for most MCP servers)
- **env** (optional): Environment variables for the server

## Common Server Configurations

### MarkItDown (Document Conversion)

```json
{
  "markitdown": {
    "command": "docker",
    "args": ["exec", "-i", "mcp-markitdown", "markitdown-mcp"],
    "transport": "stdio"
  }
}
```

### Proxmox MCP (Hypervisor Management)

```json
{
  "proxmox": {
    "command": "docker",
    "args": ["exec", "-i", "mcp-proxmox", "proxmox-mcp"],
    "transport": "stdio",
    "env": {
      "PROXMOX_HOST": "${PROXMOX_HOST}",
      "PROXMOX_USER": "${PROXMOX_USER}",
      "PROXMOX_TOKEN_NAME": "${PROXMOX_TOKEN_NAME}",
      "PROXMOX_TOKEN_VALUE": "${PROXMOX_TOKEN_VALUE}"
    }
  }
}
```

### Tailscale MCP (Network Management)

```json
{
  "tailscale": {
    "command": "docker",
    "args": ["exec", "-i", "mcp-tailscale", "node", "/app/dist/index.js"],
    "transport": "stdio",
    "env": {
      "TAILSCALE_API_KEY": "${TAILSCALE_MCP_API_KEY}",
      "TAILSCALE_TAILNET": "${TAILSCALE_MCP_TAILNET}"
    }
  }
}
```

### Atlassian (Jira + Confluence)

```json
{
  "atlassian": {
    "command": "docker",
    "args": ["exec", "-i", "mcp-atlassian", "atlassian-mcp"],
    "transport": "stdio",
    "env": {
      "JIRA_URL": "${JIRA_URL}",
      "JIRA_USERNAME": "${JIRA_USERNAME}",
      "JIRA_API_TOKEN": "${JIRA_API_TOKEN}",
      "CONFLUENCE_URL": "${CONFLUENCE_URL}",
      "CONFLUENCE_USERNAME": "${CONFLUENCE_USERNAME}",
      "CONFLUENCE_API_TOKEN": "${CONFLUENCE_API_TOKEN}"
    }
  }
}
```

## Applying Configuration Changes

After editing `mcp-config.json`:

```bash
# Restart the gateway to pick up changes
docker compose restart mcp-gateway

# Verify servers are loaded
docker logs mcp-gateway | grep "tools listed"
```

You should see output like:
```
> 6 tools listed in 45.2ms
```

## Troubleshooting

### Gateway shows "No server is enabled"

**Cause**: No servers defined in `mcp-config.json` or file not mounted correctly.

**Solution**:
1. Verify `mcp-config.json` exists in the project root
2. Check the file is valid JSON
3. Restart the gateway: `docker compose restart mcp-gateway`

### Gateway shows "0 tools listed"

**Cause**: MCP server containers are not running or not accessible.

**Solution**:
1. Check server containers are running: `docker ps | grep mcp-`
2. Start missing servers: `docker compose up -d markitdown`
3. Check server logs: `docker logs mcp-markitdown`

### Server fails to start

**Cause**: Missing environment variables or incorrect container name.

**Solution**:
1. Verify container name matches: `docker ps --format "{{.Names}}"`
2. Check environment variables are set in `.env`
3. Test manually: `docker exec -i mcp-markitdown markitdown-mcp`

### Tools not appearing in client

**Cause**: Client not configured to connect to gateway.

**Solution**:
1. Verify gateway is accessible: `curl http://localhost:3000/health`
2. Configure client (Claude Desktop, Cursor) with gateway URL
3. Check client logs for connection errors

## Environment Variables

The gateway supports environment variable substitution in `mcp-config.json`. Define variables in `.env`:

```bash
# .env
PROXMOX_HOST=192.168.1.100
PROXMOX_TOKEN_NAME=mytoken
PROXMOX_TOKEN_VALUE=secret-token-value
```

Reference them in the config:

```json
{
  "env": {
    "PROXMOX_HOST": "${PROXMOX_HOST}"
  }
}
```

## Best Practices

1. **Version Control**: Add `mcp-config.json` to git, exclude `.env`
2. **Validation**: Use a JSON validator before applying changes
3. **Documentation**: Comment server purposes in commit messages
4. **Testing**: Test each server individually before adding to production
5. **Monitoring**: Check gateway logs after configuration changes

## Next Steps

- [Add more MCP servers](./ADDING_MCP_SERVERS.md)
- [Configure tsdproxy](./TSDPROXY_SETUP.md)
- [Troubleshooting guide](./TROUBLESHOOTING.md)
