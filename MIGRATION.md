# Migration to MCPJungle - Summary

## What Changed

We've migrated from the Docker MCP Gateway to **MCPJungle**, a more mature and feature-rich MCP gateway solution.

## Key Differences

### Before (Docker MCP Gateway)

- ❌ CLI-only tool (no web UI)
- ❌ Configuration via JSON files
- ❌ UNIX socket communication only
- ❌ Pre-defined server containers in compose.yaml
- ❌ Manual configuration updates
- ❌ No built-in access control
- ❌ Limited observability

### After (MCPJungle)

- ✅ Web UI + CLI for management
- ✅ Dynamic server registration
- ✅ HTTP-based MCP gateway (easier client integration)
- ✅ PostgreSQL-backed persistence
- ✅ CLI-based server management
- ✅ Enterprise mode with authentication
- ✅ OpenTelemetry metrics support
- ✅ Tool groups for organizing tools
- ✅ Per-client access control

## Architecture Changes

### Old Architecture
```
MCP Client → UNIX Socket → Gateway Container → Docker Socket → MCP Servers
```

### New Architecture
```
MCP Client → HTTP (port 8080) → MCPJungle Gateway → PostgreSQL
                                        ↓
                                 Docker Socket → STDIO Servers
                                        ↓
                                 HTTP Requests → Remote Servers
```

## Files Changed

### Removed
- `Dockerfile.gateway` - No longer needed (using official MCPJungle image)
- `mcp-servers.json` - Server registration now via CLI, not config files
- `.dockerignore` - Not needed with official images

### Modified
- `compose.yaml` - Replaced gateway service with MCPJungle + PostgreSQL
- `README.md` - Complete rewrite for MCPJungle
- `.env.example` - Updated for MCPJungle configuration

### Added
- `QUICKSTART.md` - Quick start guide for new users

## Migration Steps

If you were using the old Docker MCP Gateway:

1. **Backup any important data**
   ```bash
   docker compose down
   ```

2. **Pull the new changes**
   ```bash
   git pull origin main
   ```

3. **Update environment file**
   ```bash
   cp .env.example .env
   # Edit .env with your passwords
   ```

4. **Start MCPJungle**
   ```bash
   docker compose up -d
   ```

5. **Install MCPJungle CLI**
   ```bash
   brew install mcpjungle/mcpjungle/mcpjungle
   # or download from releases
   ```

6. **Re-register your MCP servers**
   
   Instead of defining servers in `mcp-servers.json`, register them via CLI:
   
   ```bash
   # Example: Register a server
   mcpjungle register \
     --name my-server \
     --url http://my-server:8080/mcp
   ```

7. **Update MCP client configurations**
   
   Change from UNIX socket to HTTP endpoint:
   
   **Old (Docker Gateway):**
   ```json
   {
     "command": "docker",
     "args": ["exec", "-i", "mcp-gateway", "mcp-client", "connect"]
   }
   ```
   
   **New (MCPJungle):**
   ```json
   {
     "command": "npx",
     "args": ["mcp-remote", "http://localhost:8080/mcp", "--allow-http"]
   }
   ```

## Benefits of MCPJungle

### 1. **Easier Management**
- Web UI for viewing servers and tools
- CLI commands for registration/management
- No need to edit compose.yaml for new servers

### 2. **Better for Production**
- PostgreSQL backend (not just in-memory)
- Health checks and monitoring
- Proper state management

### 3. **Multi-User Support**
- Enterprise mode with authentication
- Per-client access control
- Audit logging

### 4. **Tool Organization**
- Create tool groups for different clients
- Enable/disable tools without removing servers
- Better control over what clients can access

### 5. **Better Documentation**
- Active community (Discord)
- Comprehensive documentation
- Regular updates and releases

## Example Workflows

### Register a new server
```bash
mcpjungle register --name myserver --url http://myserver:8080/mcp
```

### List all tools
```bash
mcpjungle list tools
```

### Create a tool group
```bash
cat > group.json <<EOF
{
  "name": "claude-tools",
  "included_tools": ["filesystem__read_file", "context7__get-library-docs"]
}
EOF
mcpjungle create group -c group.json
```

### Disable a tool temporarily
```bash
mcpjungle disable tool myserver__dangerous_tool
```

## Getting Help

- **Documentation**: https://github.com/mcpjungle/MCPJungle
- **Discord**: https://discord.gg/CapV4Z3krk
- **Issues**: https://github.com/mcpjungle/MCPJungle/issues

## FAQ

**Q: Can I still use STDIO servers?**  
A: Yes! MCPJungle supports both STDIO and HTTP transports. Use the `-stdio` tagged image.

**Q: Do I need to keep server containers in compose.yaml?**  
A: No. MCPJungle manages server lifecycle via registration, not pre-defined containers.

**Q: Is authentication required?**  
A: Only in enterprise mode. Development mode (default) has no authentication.

**Q: Can I use both gateways?**  
A: Not recommended, but technically possible on different ports. Choose one.

**Q: How do I upgrade MCPJungle?**  
A: `docker compose pull && docker compose up -d`

---

**Status**: ✅ Migration Complete

You're now using MCPJungle! Enjoy the improved experience.
