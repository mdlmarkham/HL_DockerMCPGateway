# 🌳 HomeLab MCP Gateway with MCPJungle

Self-hosted MCP Gateway using [MCPJungle](https://github.com/mcpjungle/MCPJungle) for managing Model Context Protocol (MCP) servers in your HomeLab.

## Deployment Options

- **Local Docker Compose**: Quick start below
- **[Komodo Deployment](KOMODO_DEPLOYMENT.md)**: Centralized HomeLab management (recommended)

## Quick Start

```bash
# 1. Create environment file
cp .env.example .env
# Edit .env and set POSTGRES_PASSWORD

# 2. Start services
docker compose up -d

# 3. Install MCPJungle CLI
brew install mcpjungle/mcpjungle/mcpjungle

# 4. Register an MCP server
mcpjungle register --name context7 --url https://mcp.context7.com/mcp

# 5. Connect Claude Desktop
# Add to claude_desktop_config.json:
{
  "mcpServers": {
    "mcpjungle": {
      "command": "npx",
      "args": ["mcp-remote", "http://localhost:8080/mcp", "--allow-http"]
    }
  }
}
```

## What is MCPJungle?

MCPJungle is a production-ready MCP Gateway providing:

- 🖥️ **Web UI + CLI** - Easy management
- 🔍 **Unified Endpoint** - Single gateway for all MCP servers
- 📡 **Both Transports** - HTTP and STDIO support
- 🔒 **Access Control** - Enterprise authentication
- 🎯 **Tool Groups** - Organize tools per client

## Documentation

- **[Komodo Deployment](KOMODO_DEPLOYMENT.md)** - Deploy to Komodo (recommended)
- **[Quick Start Guide](QUICKSTART.md)** - Detailed setup instructions
- **[Migration Guide](MIGRATION.md)** - Migrating from Docker MCP Gateway
- **[MCPJungle Docs](https://github.com/mcpjungle/MCPJungle)** - Official documentation

## Architecture

```
MCP Clients → http://localhost:8080/mcp → MCPJungle Gateway
                                              ↓
                                        PostgreSQL DB
                                              ↓
                                   HTTP/STDIO MCP Servers
```

## Management

```bash
# List servers
mcpjungle list servers

# List tools
mcpjungle list tools

# Disable a tool
mcpjungle disable tool server__tool_name

# Remove a server
mcpjungle deregister server_name
```

## Configuration

Edit .env file:

```bash
POSTGRES_PASSWORD=your_secure_password
SERVER_MODE=development  # or enterprise for auth
```

## Enterprise Mode

For multi-user environments:

```bash
# Enable enterprise mode
SERVER_MODE=enterprise docker compose up -d

# Initialize admin
mcpjungle init-server

# Create client with limited access
mcpjungle create mcp-client my-client --allow "server1,server2"
```

## Resources

- [MCPJungle GitHub](https://github.com/mcpjungle/MCPJungle)
- [Model Context Protocol](https://modelcontextprotocol.io/)
- [MCP Server Registry](https://github.com/modelcontextprotocol/servers)

## License

MIT
