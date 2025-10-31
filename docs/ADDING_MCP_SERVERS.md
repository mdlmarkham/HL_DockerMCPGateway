# Adding MCP Servers Guide

This guide explains how to add additional MCP servers to your deployment.

## Table of Contents

- [Overview](#overview)
- [Method 1: Declarative (compose.yaml)](#method-1-declarative-composeyaml)
- [Method 2: Runtime (MCP Catalog)](#method-2-runtime-mcp-catalog)
- [Common MCP Servers](#common-mcp-servers)
- [Server Configuration](#server-configuration)
- [Troubleshooting](#troubleshooting)

---

## Overview

There are **two approaches** to adding MCP servers:

| Method | Best For | Pros | Cons |
|--------|----------|------|------|
| **Declarative** (compose.yaml) | Production, repeatable deployments | Version controlled, consistent | Requires compose file editing |
| **Runtime** (MCP Catalog) | Testing, quick additions | Fast, interactive | Not version controlled |

Both methods work together - you can use declarative for core servers and runtime for testing.

---

## Method 1: Declarative (compose.yaml)

### Advantages

✅ **Reproducible** - Servers start automatically with the stack  
✅ **Version Controlled** - Changes tracked in Git  
✅ **Environment Variables** - Configuration via `.env` file  
✅ **Docker Compose Profiles** - Enable/disable server sets  

### Adding a Server

1. **Edit compose.yaml**

Add a new service following this template:

```yaml
services:
  # ... existing services ...

  your-server-name:
    image: mcp/server-name:latest
    container_name: mcp-your-server
    restart: "no"           # Started on-demand by gateway
    stdin_open: true        # Required for MCP protocol
    tty: true              # Required for MCP protocol
    
    environment:
      # Server-specific environment variables
      - SERVER_API_KEY=${SERVER_API_KEY:-}
      - SERVER_URL=${SERVER_URL:-}
    
    # Security hardening
    security_opt:
      - no-new-privileges:true
    read_only: true        # Set to false if server needs write access
    tmpfs:
      - /tmp
    
    # Workspace access
    volumes:
      - /srv/mcp/workspace:/workspace:ro
    
    networks: [mcp]
    
    # Optional: Use profiles to enable conditionally
    profiles:
      - optional-servers
```

2. **Update .env file**

Add configuration variables:

```bash
# Your Server Configuration
SERVER_API_KEY=your_api_key_here
SERVER_URL=https://your-server.com
```

3. **Deploy**

```bash
# Standard deployment (only servers without profiles)
docker compose up -d

# Include optional servers
docker compose --profile optional-servers up -d

# Or remove the profiles section from compose.yaml for always-on
```

### Example: Atlassian Server

The repository includes Atlassian as an example (Jira + Confluence integration):

```yaml
atlassian:
  image: mcp/atlassian:latest
  container_name: mcp-atlassian
  restart: "no"
  stdin_open: true
  tty: true
  
  environment:
    - CONFLUENCE_URL=${CONFLUENCE_URL:-}
    - CONFLUENCE_USERNAME=${CONFLUENCE_USERNAME:-}
    - CONFLUENCE_API_TOKEN=${CONFLUENCE_API_TOKEN:-}
    - JIRA_URL=${JIRA_URL:-}
    - JIRA_USERNAME=${JIRA_USERNAME:-}
    - JIRA_API_TOKEN=${JIRA_API_TOKEN:-}
  
  security_opt:
    - no-new-privileges:true
  read_only: true
  tmpfs:
    - /tmp
  
  volumes:
    - /srv/mcp/workspace:/workspace:rw
  
  networks: [mcp]
  
  profiles:
    - atlassian  # Remove this to always enable
```

**To enable:**

1. Edit `.env` and set Atlassian credentials
2. Either:
   - Remove `profiles: [atlassian]` from compose.yaml, OR
   - Deploy with: `docker compose --profile atlassian up -d`

---

## Method 2: Runtime (MCP Catalog)

### Advantages

✅ **Quick** - Enable servers in seconds  
✅ **Interactive** - Guided configuration  
✅ **No file editing** - Command-line only  
✅ **Testing** - Try before committing to compose.yaml  

### Commands

When running via Docker Compose (Komodo), execute commands **inside the Gateway container**:

#### Via Komodo UI:

1. Navigate to **Stacks** → **HL_DockerMCPGateway**
2. Click **mcp-gateway** container
3. Open **Terminal** tab
4. Run:

```bash
# List available servers
docker mcp server catalog

# Enable a server
docker mcp server enable github

# Configure (interactive prompts)
docker mcp server configure github

# List enabled servers
docker mcp server list

# Check status
docker mcp server status github

# Disable a server
docker mcp server disable github
```

#### Via SSH/CLI:

```bash
# Execute commands in the Gateway container
docker exec mcp-gateway docker mcp server catalog
docker exec mcp-gateway docker mcp server enable github
docker exec -it mcp-gateway docker mcp server configure github
docker exec mcp-gateway docker mcp server list
```

### Limitations

⚠️ **Not Persistent** - Servers enabled via catalog may not survive container restarts  
⚠️ **Not Version Controlled** - Changes not tracked in Git  
⚠️ **Manual Configuration** - Need to reconfigure after recreating containers  

**Recommendation**: Use this for testing, then move to declarative once confirmed working.

---

## Common MCP Servers

Here are popular MCP servers you can add:

### Atlassian (Jira + Confluence)

**Image**: `mcp/atlassian:latest`  
**Tools**: 37 (Jira + Confluence operations)  
**Configuration**: API tokens from Atlassian  
**Use Cases**: Project management, documentation, issue tracking  

```yaml
atlassian:
  image: mcp/atlassian:latest
  environment:
    - CONFLUENCE_URL=${CONFLUENCE_URL}
    - CONFLUENCE_USERNAME=${CONFLUENCE_USERNAME}
    - CONFLUENCE_API_TOKEN=${CONFLUENCE_API_TOKEN}
    - JIRA_URL=${JIRA_URL}
    - JIRA_USERNAME=${JIRA_USERNAME}
    - JIRA_API_TOKEN=${JIRA_API_TOKEN}
```

### OpenAPI Toolkit

**Image**: `mcp/openapi:latest`  
**Tools**: 5 (OpenAPI/Swagger operations)  
**Configuration**: None required (works with URLs or file contents)  
**Use Cases**: API development, spec validation, code generation, cURL command generation  

```yaml
openapi:
  image: mcp/openapi:latest
  environment:
    - MODE=Stdio
  volumes:
    - /srv/mcp/workspace:/workspace:ro  # Optional: for loading specs from files
```

**Tools provided:**
- `create_csharp_snippet` - Generate C# code from OpenAPI specs
- `generate_curl_command` - Generate cURL commands for API operations
- `get_known_responses` - List possible HTTP responses for operations
- `get_list_of_operations` - Get all endpoints from a spec
- `validate_document` - Validate and analyze OpenAPI/Swagger specs

### GitHub

**Image**: `mcp/github:latest`  
**Tools**: Repository management, issues, PRs  
**Configuration**: GitHub token  
**Use Cases**: Code management, CI/CD integration  

```yaml
github:
  image: mcp/github:latest
  environment:
    - GITHUB_TOKEN=${GITHUB_TOKEN}
```

### PostgreSQL

**Image**: `mcp/postgres:latest`  
**Tools**: Database queries, schema inspection  
**Configuration**: Database connection string  
**Use Cases**: Database operations, data analysis  

```yaml
postgres:
  image: mcp/postgres:latest
  environment:
    - POSTGRES_CONNECTION_STRING=${POSTGRES_CONNECTION_STRING}
```

### Filesystem

**Image**: `mcp/filesystem:latest`  
**Tools**: File operations, directory management  
**Configuration**: Base path  
**Use Cases**: File management, local operations  

```yaml
filesystem:
  image: mcp/filesystem:latest
  volumes:
    - /srv/mcp/workspace:/workspace:rw
  environment:
    - FILESYSTEM_BASE_PATH=/workspace
```

### Web Browse

**Image**: `mcp/web-browse:latest`  
**Tools**: Web scraping, page fetching  
**Configuration**: None (or proxy settings)  
**Use Cases**: Web research, data extraction  

```yaml
web-browse:
  image: mcp/web-browse:latest
  # Optional proxy configuration
  environment:
    - HTTP_PROXY=${HTTP_PROXY:-}
    - HTTPS_PROXY=${HTTPS_PROXY:-}
```

### Brave Search

**Image**: `mcp/brave-search:latest`  
**Tools**: Web search, AI search  
**Configuration**: Brave API key  
**Use Cases**: Information retrieval, research  

```yaml
brave-search:
  image: mcp/brave-search:latest
  environment:
    - BRAVE_API_KEY=${BRAVE_API_KEY}
```

### Obsidian

**Image**: `mcp/obsidian:latest`  
**Tools**: 12 (vault management, search, file operations)  
- `obsidian_append_content` - Append content to files
- `obsidian_batch_get_file_contents` - Read multiple files
- `obsidian_complex_search` - JsonLogic queries
- `obsidian_delete_file` - Delete files/directories
- `obsidian_get_file_contents` - Read single file
- `obsidian_get_periodic_note` - Daily/weekly/monthly notes
- `obsidian_get_recent_changes` - Recently modified files
- `obsidian_get_recent_periodic_notes` - Recent periodic notes
- `obsidian_list_files_in_dir` - List directory contents
- `obsidian_list_files_in_vault` - List vault root
- `obsidian_patch_content` - Patch notes (headings/blocks)
- `obsidian_simple_search` - Text search

**Configuration**: Requires Obsidian REST API plugin  
**Use Cases**: Personal knowledge management, note taking  

```yaml
obsidian:
  image: mcp/obsidian:latest
  environment:
    - OBSIDIAN_HOST=${OBSIDIAN_HOST:-host.docker.internal}
    - OBSIDIAN_API_KEY=${OBSIDIAN_API_KEY}
  profiles:
    - obsidian
```

### Reddit

**Image**: `mcp/reddit-mcp:latest`  
**Tools**: 6 (Reddit operations)  
- `fetchPosts` - Fetch hot posts from subreddit
- `getComments` - Get post comments
- `getSubredditInfo` - Get subreddit information
- `postComment` - Post a comment
- `postToSubreddit` - Create new post
- `searchPosts` - Search within subreddit

**Configuration**: Reddit API credentials  
**Use Cases**: Social media monitoring, content posting  

```yaml
reddit:
  image: mcp/reddit-mcp:latest
  environment:
    - USERNAME=${REDDIT_USERNAME}
    - REDDIT_CLIENT_ID=${REDDIT_CLIENT_ID}
    - REDDIT_CLIENT_SECRET=${REDDIT_CLIENT_SECRET}
    - REDDIT_PASSWORD=${REDDIT_PASSWORD}
  profiles:
    - reddit
```

### Context7

**Image**: `mcp/context7:latest`  
**Tools**: 2 (library documentation)  
- `get-library-docs` - Fetch library documentation
- `resolve-library-id` - Resolve library ID from name

**Configuration**: None required  
**Use Cases**: Up-to-date library documentation for AI coding  

```yaml
context7:
  image: mcp/context7:latest
  # No configuration needed
  profiles:
    - context7
```

### OpenAPI Schema

**Image**: `mcp/openapi-schema:latest`  
**Tools**: 10 (OpenAPI schema analysis)  
- `get-component` - Get component definition
- `get-endpoint` - Get endpoint details
- `get-examples` - Get examples
- `get-path-parameters` - Get path parameters
- `get-request-body` - Get request body schema
- `get-response-schema` - Get response schema
- `list-components` - List all components
- `list-endpoints` - List API endpoints
- `list-security-schemes` - List security schemes
- `search-schema` - Search across schema

**Configuration**: Requires volume mount for schema files  
**Use Cases**: OpenAPI spec analysis, API documentation  

```yaml
openapi-schema:
  image: mcp/openapi-schema:latest
  volumes:
    - /srv/mcp/workspace:/workspace:ro
  profiles:
    - openapi-schema
```

### Wikipedia

**Image**: `mcp/wikipedia-mcp:latest`  
**Tools**: 11 (Wikipedia knowledge access)  
- `extract_key_facts` - Extract key facts
- `get_article` - Get full article content
- `get_coordinates` - Get article coordinates
- `get_links` - Get article links
- `get_related_topics` - Get related topics
- `get_sections` - Get article sections
- `get_summary` - Get article summary
- `search_wikipedia` - Search Wikipedia
- `summarize_article_for_query` - Query-specific summary
- `summarize_article_section` - Section summary
- `test_wikipedia_connectivity` - Test connectivity

**Configuration**: None required  
**Use Cases**: Knowledge retrieval, research, fact-checking  

```yaml
wikipedia:
  image: mcp/wikipedia-mcp:latest
  # No configuration needed
  profiles:
    - wikipedia
```

### Komodo

**Image**: `ghcr.io/mp-tool/komodo-mcp-server:latest`  
**Tools**: 15 (Komodo container management)  
- `komodo_configure` - Configure Komodo server connection
- `komodo_health_check` - Check Komodo connectivity
- `komodo_list_servers` - List all available servers
- `komodo_get_server_stats` - Get server statistics
- `komodo_list_containers` - List Docker containers
- `komodo_start_container` - Start a container
- `komodo_stop_container` - Stop a container
- `komodo_restart_container` - Restart a container
- `komodo_pause_container` - Pause a container
- `komodo_unpause_container` - Unpause a container
- `komodo_list_deployments` - List all deployments
- `komodo_deploy_container` - Deploy a container
- `komodo_list_stacks` - List Docker Compose stacks
- `komodo_deploy_stack` - Deploy a stack
- `komodo_stop_stack` - Stop a stack

**Configuration**: Komodo server URL and credentials  
**Use Cases**: Container orchestration, deployment automation, HomeLab management  
**Special**: This is meta - managing Komodo from within Komodo!  

```yaml
komodo-mcp:
  image: ghcr.io/mp-tool/komodo-mcp-server:latest
  environment:
    - KOMODO_URL=${KOMODO_URL}
    - KOMODO_USERNAME=${KOMODO_USERNAME}
    - KOMODO_PASSWORD=${KOMODO_PASSWORD}
    - NODE_ENV=production
  profiles:
    - komodo
```

### Proxmox MCP Server

**Tools**: 6 (get_nodes, get_node_status, get_vms, execute_vm_command, get_storage, get_cluster_status)  
**Repository**: [canvrno/ProxmoxMCP](https://github.com/canvrno/ProxmoxMCP)  
**License**: MIT  
**Use Cases**: HomeLab hypervisor management, VM monitoring, infrastructure automation

This server manages Proxmox Virtual Environment (VE/PVE) hypervisors:

- **Node Management**: List nodes, check status, monitor resources
- **VM Operations**: List VMs, execute commands via QEMU guest agent
- **Storage Management**: List storage pools, check usage and capacity
- **Cluster Tools**: Monitor cluster health, quorum status

**Configuration**:
```yaml
proxmox-mcp:
  image: ghcr.io/canvrno/proxmoxmcp:latest
  container_name: mcp-proxmox
  stdin_open: true
  tty: true
  environment:
    - PROXMOX_HOST=${PROXMOX_HOST}
    - PROXMOX_PORT=${PROXMOX_PORT:-8006}
    - PROXMOX_USER=${PROXMOX_USER}
    - PROXMOX_TOKEN_NAME=${PROXMOX_TOKEN_NAME}
    - PROXMOX_TOKEN_VALUE=${PROXMOX_TOKEN_VALUE}
    - PROXMOX_VERIFY_SSL=${PROXMOX_VERIFY_SSL:-false}
    - PROXMOX_SERVICE=${PROXMOX_SERVICE:-PVE}
    - LOG_LEVEL=${PROXMOX_LOG_LEVEL:-INFO}
  networks:
    - mcp
  security_opt:
    - no-new-privileges:true
  read_only: true
  tmpfs:
    - /tmp
  profiles:
    - proxmox
```

**Setup Requirements**:
1. Create API token in Proxmox UI (Datacenter → Permissions → API Tokens)
2. Use format `username@realm` (e.g., `root@pam` or `user@pve`)
3. Set `PROXMOX_VERIFY_SSL=false` for self-signed certificates
4. Ensure QEMU guest agent is installed in VMs for `execute_vm_command`

**Special Note**: Perfect for HomeLab environments! Manage the same Proxmox hypervisor running your containers through conversational AI.

---

### Tailscale MCP Server

**Tools**: 20+ (device management, network status, ACL operations, admin tools)  
**Repository**: [HexSleeves/tailscale-mcp](https://github.com/HexSleeves/tailscale-mcp)  
**License**: MIT  
**Use Cases**: Tailscale network management, device monitoring, ACL automation, network status

This server enables programmatic control of your Tailscale network:

- **Network Tools**: Get version, network status, ping peers
- **Device Management**: List devices, monitor status, manage configurations
- **ACL Operations**: View and manage Access Control Lists
- **Admin Tools**: Manage tailnet settings, users, and permissions

**Configuration**:
```yaml
tailscale-mcp:
  image: ghcr.io/hexsleeves/tailscale-mcp-server:latest
  container_name: mcp-tailscale
  stdin_open: true
  tty: true
  environment:
    - TAILSCALE_API_KEY=${TAILSCALE_MCP_API_KEY}
    - TAILSCALE_TAILNET=${TAILSCALE_MCP_TAILNET}
    - TAILSCALE_API_BASE_URL=${TAILSCALE_API_BASE_URL:-https://api.tailscale.com}
    - LOG_LEVEL=${TAILSCALE_MCP_LOG_LEVEL:-1}
    - NODE_ENV=${NODE_ENV:-production}
  networks:
    - mcp
  security_opt:
    - no-new-privileges:true
  read_only: true
  tmpfs:
    - /tmp
    - /app/logs
  profiles:
    - tailscale-mcp
```

**Setup Requirements**:
1. Generate API key at https://login.tailscale.com/admin/settings/keys
2. Use a **different** API key than the `TS_AUTHKEY` (that's for auth, this is for management)
3. Set `TAILSCALE_MCP_TAILNET` to your tailnet name (e.g., alice@gmail.com or mycompany.com)
4. Log levels: 0=DEBUG, 1=INFO (default), 2=WARN, 3=ERROR

**Special Note**: This MCP server manages your Tailscale network programmatically! It's **separate** from the `tailscale-gateway` service which provides network connectivity. This server enables AI assistants to automate Tailscale device management, ACL updates, and network monitoring.

---

## Server Configuration

### Environment Variables

All servers use environment variables for configuration. Best practices:

1. **Never commit secrets** - Use `.env` file (already in `.gitignore`)
2. **Use defaults** - `${VAR:-default}` syntax provides fallbacks
3. **Document requirements** - Update `.env.example` when adding servers
4. **Komodo integration** - Use Komodo's secret management for sensitive values

### Volumes

Servers may need access to the workspace:

```yaml
volumes:
  # Read-only for security
  - /srv/mcp/workspace:/workspace:ro
  
  # Read-write if server creates files
  - /srv/mcp/workspace:/workspace:rw
```

### Security Hardening

All servers should include:

```yaml
security_opt:
  - no-new-privileges:true
read_only: true  # If server doesn't write files
tmpfs:
  - /tmp  # Writable temporary storage
```

### Network

All servers MUST be on the `mcp` network for Gateway discovery:

```yaml
networks: [mcp]
```

---

## Troubleshooting

### Server Not Discovered by Gateway

**Symptoms**: Server running but not visible to clients

**Solutions**:
1. Verify server is on `mcp` network:
   ```bash
   docker network inspect mcp
   ```

2. Check Gateway can reach server:
   ```bash
   docker exec mcp-gateway ping mcp-servername
   ```

3. Restart Gateway to force discovery:
   ```bash
   docker compose restart mcp-gateway
   ```

### Configuration Errors

**Symptoms**: Server starts but tools don't work

**Solutions**:
1. Check environment variables:
   ```bash
   docker exec mcp-servername env | grep -i api
   ```

2. View server logs:
   ```bash
   docker logs mcp-servername
   # Or via catalog:
   docker exec mcp-gateway docker mcp server logs servername
   ```

3. Validate credentials externally before configuring

### Permission Issues

**Symptoms**: Server can't access workspace files

**Solutions**:
1. Check volume mount:
   ```bash
   docker exec mcp-servername ls -la /workspace
   ```

2. Verify host permissions:
   ```bash
   ls -la /srv/mcp/workspace
   ```

3. Adjust volume to `rw` if needed (from `ro`)

### Server Won't Start

**Symptoms**: Container immediately exits

**Solutions**:
1. Check container logs:
   ```bash
   docker logs mcp-servername
   ```

2. Verify `stdin_open: true` and `tty: true` are set

3. Check image exists and is up to date:
   ```bash
   docker pull mcp/servername:latest
   ```

### Profile Not Working

**Symptoms**: Server doesn't start with stack

**Solutions**:
1. Deploy with profile explicitly:
   ```bash
   docker compose --profile your-profile up -d
   ```

2. Or remove `profiles:` section from service definition

3. Verify profile name matches in compose and command

---

## Best Practices

### Development Workflow

1. **Test with Runtime Method** first
   ```bash
   docker exec mcp-gateway docker mcp server enable newserver
   ```

2. **Verify it works** with your use case

3. **Add to compose.yaml** for production:
   - Copy container definition
   - Add environment variables to `.env.example`
   - Document in README
   - Commit changes

4. **Use profiles for optional servers**:
   - Core servers: No profile (always on)
   - Optional/testing: Use profiles

### Production Deployment

1. **Pin image versions** for stability:
   ```yaml
   image: mcp/atlassian:v1.2.3  # Not :latest
   ```

2. **Document all servers** in README

3. **Backup configurations** regularly

4. **Monitor server health** via Komodo

5. **Update regularly** for security patches

### Security

1. **Least privilege** - Read-only volumes when possible
2. **No secrets in Git** - Always use `.env` file
3. **Network isolation** - Only `mcp` network, no external exposure
4. **Regular updates** - `docker compose pull` weekly
5. **Audit logs** - Review server logs periodically

---

## Additional Resources

- [MCP Server Catalog](https://github.com/modelcontextprotocol/servers)
- [Docker MCP Gateway Docs](https://github.com/docker/mcp-gateway)
- [Docker Compose Documentation](https://docs.docker.com/compose/)
- [Komodo Documentation](https://github.com/mbecker20/komodo)

---

**Next Steps:**

1. ✅ Choose your method (declarative vs runtime)
2. ✅ Select servers from the catalog
3. ✅ Configure credentials in `.env`
4. ✅ Deploy and test
5. ✅ Verify with `docker logs` and client testing
