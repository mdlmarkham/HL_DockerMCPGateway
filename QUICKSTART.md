# MCPJungle Quick Start Guide

This guide will get you up and running with MCPJungle in minutes.

## Prerequisites

- Docker and Docker Compose installed
- MCPJungle CLI installed ([installation guide](https://github.com/mcpjungle/MCPJungle#installation))

## Step 1: Deploy MCPJungle

```bash
# Clone or navigate to the repository
cd HL_DockerMCPGateway

# Create environment file
cp .env.example .env

# Edit .env and set a secure PostgreSQL password
nano .env  # or use your preferred editor

# Start the services
docker compose up -d

# Verify services are running
docker compose ps

# Check health
curl http://localhost:8080/health
```

You should see:
- `mcpjungle` - Running on port 8080
- `postgres` - Running (internal only)

## Step 2: Register Your First MCP Server

### Option A: HTTP-based Server (context7)

Context7 provides up-to-date library documentation:

```bash
mcpjungle register \
  --name context7 \
  --description "Up-to-date library documentation" \
  --url https://mcp.context7.com/mcp

# Verify registration
mcpjungle list servers
mcpjungle list tools --server context7
```

### Option B: STDIO-based Server (filesystem)

Filesystem server allows reading/writing files:

```bash
# Create config file
cat > filesystem.json <<'EOF'
{
  "name": "filesystem",
  "transport": "stdio",
  "description": "Filesystem access for MCP clients",
  "command": "npx",
  "args": ["-y", "@modelcontextprotocol/server-filesystem", "/host"]
}
EOF

# Register the server
mcpjungle register -c filesystem.json

# Verify
mcpjungle list tools --server filesystem
```

## Step 3: Test the Gateway

```bash
# List all available tools
mcpjungle list tools

# Invoke a tool
mcpjungle invoke context7__get-library-docs \
  --input '{"context7CompatibleLibraryID": "/lodash/lodash"}'
```

## Step 4: Connect Claude Desktop

Edit Claude's config file:

**macOS:** `~/Library/Application Support/Claude/claude_desktop_config.json`
**Windows:** `%APPDATA%\Claude\claude_desktop_config.json`
**Linux:** `~/.config/Claude/claude_desktop_config.json`

Add this configuration:

```json
{
  "mcpServers": {
    "mcpjungle": {
      "command": "npx",
      "args": [
        "mcp-remote",
        "http://localhost:8080/mcp",
        "--allow-http"
      ]
    }
  }
}
```

Restart Claude Desktop and ask:

> "What tools are available to you?"

Claude should list all the tools from your registered MCP servers!

## Step 5: Connect Cursor

Edit Cursor's MCP settings:

**All platforms:** Settings → Features → MCP → Edit Config

```json
{
  "mcpServers": {
    "mcpjungle": {
      "url": "http://localhost:8080/mcp"
    }
  }
}
```

Restart Cursor and test the connection.

## Common First Servers to Register

### Time Server (STDIO)

```bash
cat > time.json <<'EOF'
{
  "name": "time",
  "transport": "stdio",
  "description": "Get current time and convert timezones",
  "command": "npx",
  "args": ["-y", "@modelcontextprotocol/server-time"]
}
EOF

mcpjungle register -c time.json
```

### Brave Search (HTTP)

```bash
# Get API key from: https://brave.com/search/api/
mcpjungle register \
  --name brave-search \
  --description "Web and local search using Brave" \
  --url https://mcp.brave.com/search \
  --bearer-token "YOUR_BRAVE_API_KEY"
```

### GitHub MCP (STDIO)

```bash
cat > github.json <<'EOF'
{
  "name": "github",
  "transport": "stdio",
  "description": "GitHub repository management",
  "command": "npx",
  "args": ["-y", "@modelcontextprotocol/server-github"],
  "env": {
    "GITHUB_PERSONAL_ACCESS_TOKEN": "your_github_token_here"
  }
}
EOF

mcpjungle register -c github.json
```

## Next Steps

- **Explore more servers**: Check the [MCP Server Registry](https://github.com/modelcontextprotocol/servers)
- **Create tool groups**: Organize tools for different clients
- **Enable enterprise mode**: Add authentication for multi-user setups
- **Monitor metrics**: Enable OpenTelemetry for observability

## Troubleshooting

### Gateway not responding

```bash
# Check logs
docker compose logs mcpjungle

# Restart services
docker compose restart
```

### CLI can't connect

```bash
# Verify gateway is running
curl http://localhost:8080/health

# Check network connectivity
docker compose ps
```

### STDIO server fails

```bash
# Check gateway logs for stderr output
docker compose logs mcpjungle | grep -A 5 "server_name"

# Verify npx is available in container
docker compose exec mcpjungle which npx
```

## Getting Help

- [MCPJungle Documentation](https://github.com/mcpjungle/MCPJungle)
- [MCPJungle Discord](https://discord.gg/CapV4Z3krk)
- [MCP Specification](https://modelcontextprotocol.io/)

---

🎉 **Congratulations!** You now have a working MCP Gateway with MCPJungle!
