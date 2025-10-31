# 🌳 HomeLab MCP Gateway with MCPJungle# 🧠 MCP Gateway Stack (Komodo Deployment)



[![MCPJungle](https://img.shields.io/badge/MCPJungle-Self--Hosted-blue.svg)](https://github.com/mcpjungle/MCPJungle)[![Docker Compose](https://img.shields.io/badge/docker--compose-v3.9-blue.svg)](https://docs.docker.com/compose/)

[![Docker Compose](https://img.shields.io/badge/docker--compose-ready-blue.svg)](https://docs.docker.com/compose/)[![Tailscale](https://img.shields.io/badge/Tailscale-Enabled-blue.svg)](https://tailscale.com)

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

[![PRs Welcome](https://img.shields.io/badge/PRs-welcome-brightgreen.svg)](CONTRIBUTING.md)

Self-hosted MCP Gateway using [MCPJungle](https://github.com/mcpjungle/MCPJungle) for managing and accessing Model Context Protocol (MCP) servers in your HomeLab environment.[![Maintenance](https://img.shields.io/badge/Maintained%3F-yes-green.svg)](https://github.com/YOUR-USERNAME/HL_DockerMCPGateway/graphs/commit-activity)



---This repository contains the Docker Compose configuration for running the **Docker MCP Gateway** and the **markitdown-mcp** server inside a **homelab environment**, managed by **Komodo** and exposed as **Tailscale Services** for secure, easy access.



## 📦 What is MCPJungle?---



MCPJungle is a production-ready, self-hosted MCP Gateway and Registry that provides:## 📦 Overview



- **🎯 Centralized Management**: Single source-of-truth for all MCP serversThe [**Docker MCP Gateway**](https://github.com/docker/mcp-gateway) provides a unified endpoint for Model Context Protocol (MCP) clients such as Claude Desktop, Cursor, and GitHub Copilot MCP.  

- **🖥️ Web UI + CLI**: Easy server registration and managementIt aggregates multiple MCP servers (tools) and exposes them to clients through a single interface.

- **🔍 Tool Discovery**: Unified gateway endpoint for all your MCP tools

- **🔒 Access Control**: Enterprise features for multi-user environments### Key Features

- **📡 Both Transports**: Supports STDIO and HTTP MCP servers

- **🎛️ Tool Groups**: Organize tools for different MCP clients- 🔐 **Tailscale Integration**: HTTPS access via tsdproxy or Tailscale Serve

- **📊 Observability**: OpenTelemetry metrics support- 🎯 **Flexible Deployment**: Use existing tsdproxy or dedicated sidecar

- 🔒 **Zero Trust Security**: Access controlled via Tailscale ACLs

---- 📡 **No Port Forwarding**: No firewall configuration needed

- 🚀 **Auto-provisioned TLS**: Certificates managed by Tailscale

## 🏗️ Architecture- 🎛️ **Komodo Managed**: Easy deployment and monitoring

- 🔍 **Auto-Discovery**: Finds MCP servers on Docker network

```

┌─────────────────────────────────────────────────────────┐### Architecture

│  MCP Clients (Claude, Cursor, etc.)                     │

│  Connect to: http://localhost:8080/mcp                  │```

└────────────────────────┬────────────────────────────────┘┌─────────────────────────────────────────────────────────────┐

                         ││              Your Tailscale Network                          │

                         │ MCP Protocol (HTTP)│                                                              │

                         ▼│  ┌──────────────┐      ┌────────────────────────────┐      │

┌─────────────────────────────────────────────────────────┐│  │ MCP Clients  │─────▶│ Docker MCP Gateway (CLI)   │      │

│  MCPJungle Gateway                                       ││  │ (Claude,etc) │ sock │ /var/run/mcp-gateway.sock  │      │

│  - Unified MCP endpoint                                  ││  └──────────────┘      └──────────┬─────────────────┘      │

│  - Tool discovery & routing                              ││                                    │                        │

│  - PostgreSQL backend                                    ││                         ┌──────────▼───────────┐            │

│  - Web UI on port 8080                                   ││                         │   Docker Socket      │            │

└────────────────────────┬────────────────────────────────┘│                         │   Spawns servers:    │            │

                         ││                         └──────────┬───────────┘            │

        ┌────────────────┼────────────────┐│                                    │                        │

        │                │                ││                         ┌──────────▼───────────┐            │

        ▼                ▼                ▼│                         │   MCP Servers        │            │

    HTTP Servers    STDIO Servers    Custom Servers│                         │   + markitdown       │            │

    (context7)      (filesystem)     (Your own)│                         │   + proxmox-mcp      │            │

```│                         │   + tailscale-mcp    │            │

│                         └──────────────────────┘            │

---│                                                              │

└─────────────────────────────────────────────────────────────┘

## 🚀 Quick Start                 Managed by Komodo

```

### 1. Deploy the Stack

In this setup:

```bash

# Clone the repository- **Docker MCP Gateway (CLI Tool)**  

git clone https://github.com/YOUR-USERNAME/HL_DockerMCPGateway.git  NOT a long-running container on port 3000. It's a CLI process that:

cd HL_DockerMCPGateway  - Reads `mcp-servers.json` to know which servers to manage

  - Spawns MCP server containers on demand via Docker socket

# Create environment file  - Listens on a UNIX socket (default: `/var/run/mcp-gateway.sock`)

cp .env.example .env  - MCP clients connect to this socket, not HTTP

# Edit .env with your passwords and credentials

- **MCP Server Containers**  

# Start MCPJungle and PostgreSQL  - **Markitdown**: Converts files (PDF, DOCX, HTML, etc.) into Markdown

docker compose up -d  - **Proxmox MCP**: Manages VMs, containers, and nodes in your hypervisor

  - **Tailscale MCP**: Automates your Tailscale network configuration

# Check that services are running  - Many more available via profiles

docker compose ps  - Started on-demand by the gateway, not manually



# View logs- **Komodo** orchestrates the server container definitions.

docker compose logs -f mcpjungle

```---



The gateway will be available at:## 🗂 Directory Structure

- **MCP Endpoint**: http://localhost:8080/mcp

- **Health Check**: http://localhost:8080/health```

.

### 2. Install MCPJungle CLI├── compose.yaml             # MCP Gateway + MCP servers

├── tailscale/

The CLI is used to register and manage MCP servers:│   └── serve-config.json    # Tailscale Serve config (if using sidecar)

├── README.md                # This file

**macOS/Linux (Homebrew):**├── LICENSE                  # MIT License

```bash├── .gitignore               # Git ignore rules

brew install mcpjungle/mcpjungle/mcpjungle├── .env.example             # Environment template

```├── docs/

│   ├── KOMODO_SETUP.md      # Komodo deployment guide

**Windows/Manual Install:**│   ├── TAILSCALE_SETUP.md   # Basic Tailscale configuration

Download the binary from [MCPJungle Releases](https://github.com/mcpjungle/MCPJungle/releases)│   ├── TSDPROXY_SETUP.md    # tsdproxy integration guide (RECOMMENDED!)

│   ├── TAILSCALE_SERVICES.md # Alternative: Tailscale Services with sidecar

**Verify Installation:**│   ├── TROUBLESHOOTING.md   # Common issues and solutions

```bash│   └── WORKSPACE_SETUP.md   # Workspace configuration

mcpjungle version└── workspace/               # Shared folder mounted read-only

``````



### 3. Register MCP Servers- **`workspace/`**:  

  Place any files here that you want to make accessible to MCP tools for conversion or processing.

#### HTTP-based MCP Server (e.g., context7)

- **`docs/TSDPROXY_SETUP.md`**:  

```bash  **RECOMMENDED**: Guide for integrating with your existing tsdproxy setup.

mcpjungle register \

  --name context7 \- **`tailscale/serve-config.json`** (optional):  

  --description "Up-to-date library documentation" \  If using Tailscale sidecar instead of tsdproxy, configures Tailscale Serve.

  --url https://mcp.context7.com/mcp

```---



#### STDIO-based MCP Server (e.g., filesystem)## ⚙️ Prerequisites



```bash- **Komodo** installed and configured on the host — See [Komodo Setup Guide](docs/KOMODO_SETUP.md)

# Create configuration file- **Docker** and **Docker Compose v2.20+** available

cat > filesystem.json <<EOF- **Tailscale** running on the host with tsdproxy configured

{  - **OR** a Tailscale Account with auth key for sidecar deployment

  "name": "filesystem",- Sign up at https://tailscale.com if you don't have an account

  "transport": "stdio",- **Tailscale Auth Key** — Generate at https://login.tailscale.com/admin/settings/keys

  "description": "Filesystem access for MCP clients",- **Services Enabled** — Enable at https://login.tailscale.com/admin/services

  "command": "npx",- Optional: a **Tailscale ACL** restricting access to TCP ports 6274–6277

  "args": ["-y", "@modelcontextprotocol/server-filesystem", "/host"]

}## 📚 Documentation

EOF

- **[tsdproxy Integration Guide](docs/TSDPROXY_SETUP.md)** - **⭐ RECOMMENDED** - Use your existing tsdproxy setup

# Register the server- **[Tailscale Services Guide](docs/TAILSCALE_SERVICES.md)** - Alternative: Deploy with dedicated Tailscale sidecar

mcpjungle register -c filesystem.json- **[Komodo Setup Guide](docs/KOMODO_SETUP.md)** - Detailed instructions for deploying with Komodo

```- **[Adding MCP Servers](docs/ADDING_MCP_SERVERS.md)** - How to add and configure additional MCP servers

- **[Troubleshooting Guide](docs/TROUBLESHOOTING.md)** - Common issues and solutions

#### Your HomeLab Servers (Proxmox, Tailscale, etc.)- **[Workspace Setup](docs/WORKSPACE_SETUP.md)** - Configure the workspace directory



If you're running these as separate HTTP services:---



```bash## 🚀 Quick Start (Komodo Automatic Deployment)

# Example: Proxmox MCP Server (HTTP)

mcpjungle register \### 1️⃣ Clone This Repository

  --name proxmox \```bash

  --description "Proxmox hypervisor management" \git clone https://github.com/mdlmarkham/HL_DockerMCPGateway.git

  --url http://your-proxmox-mcp-server:8080/mcp \cd HL_DockerMCPGateway

  --bearer-token "${PROXMOX_API_TOKEN}"```



# Example: Tailscale MCP Server (HTTP)### 2️⃣ Configure MCP Servers (Optional)

mcpjungle register \

  --name tailscale \Edit `mcp-servers.json` to add more servers beyond markitdown:

  --description "Tailscale network management" \

  --url http://your-tailscale-mcp-server:8080/mcp \```json

  --bearer-token "${TAILSCALE_API_KEY}"{

```  "mcpServers": {

    "markitdown": {

### 4. Verify Registration      "command": "docker",

      "args": [

```bash        "run", "--rm", "-i",

# List all registered servers        "--network=hl_dockermcpgateway_mcp",

mcpjungle list servers        "--name=mcp-markitdown",

        "-v", "/srv/mcp/workspace:/workspace:ro",

# List all available tools        "--security-opt=no-new-privileges:true",

mcpjungle list tools        "--read-only",

        "--tmpfs=/tmp",

# Get details about a specific server        "mcp/markitdown:latest"

mcpjungle get server context7      ]

    }

# Test a tool  }

mcpjungle invoke context7__get-library-docs \}

  --input '{"context7CompatibleLibraryID": "/lodash/lodash"}'```

```

See [docs/GATEWAY_CONFIG.md](docs/GATEWAY_CONFIG.md) for more server examples.

### 5. Connect Your MCP Clients

### 3️⃣ Configure Server Credentials (Optional)

#### Claude Desktop

If using Proxmox, Tailscale, or other authenticated servers:

Edit your Claude config (`~/Library/Application Support/Claude/claude_desktop_config.json` on macOS):

```bash

```jsoncp .env.example .env

{nano .env

  "mcpServers": {# Add your credentials

    "mcpjungle": {```

      "command": "npx",

      "args": [### 4️⃣ Deploy the Stack via Komodo

        "mcp-remote",

        "http://localhost:8080/mcp",The first deployment will build the gateway image from source:

        "--allow-http"

      ]```bash

    }# Build and deploy the entire stack

  }docker compose up -d --build

}

```# The gateway-runner service will:

# - Build the gateway from GitHub source

#### Cursor# - Start automatically

# - Spawn MCP server containers on demand

Edit your Cursor MCP config:```



```json**Note**: The first deployment takes a few minutes to build the gateway. Subsequent deployments are instant.

{

  "mcpServers": {### 5️⃣ Verify the Gateway is Running

    "mcpjungle": {

      "url": "http://localhost:8080/mcp"```bash

    }# Check gateway logs

  }docker logs mcp-gateway

}

```# Check if socket was created

ls -la /var/run/mcp-gateway.sock

---

# Test the gateway

## 🎛️ Managementdocker logs mcp-gateway | grep -i "listening"

```

### List Tools & Servers

### 6️⃣ Connect Your MCP Client

```bash

# List all serversConfigure your MCP client (Claude Desktop, Cursor, etc.) to connect to the gateway socket:

mcpjungle list servers

```json

# List all tools{

mcpjungle list tools  "mcpServers": {

    "docker-gateway": {

# List tools from specific server      "command": "docker",

mcpjungle list tools --server filesystem      "args": ["exec", "-i", "mcp-gateway", "mcp-gateway", "connect"],

      "transport": "stdio"

# View tool details    }

mcpjungle get tool filesystem__read_file  }

}

# Check tool usage statistics```

mcpjungle usage filesystem__read_file

```Or if the gateway socket is accessible from your client machine, connect directly to `/var/run/mcp-gateway.sock`.



### Enable/Disable Tools### 7️⃣ Add More Servers (Optional)



```bashEdit `mcp-servers.json` and restart the gateway:

# Disable a specific tool (won't be available to clients)

mcpjungle disable tool filesystem__read_file```bash

# Edit configuration

# Re-enable itnano mcp-servers.json

mcpjungle enable tool filesystem__read_file

# Restart gateway to pick up changes

# Disable entire server (all its tools)docker compose restart mcp-gateway

mcpjungle disable server filesystem

# Gateway will now manage the new servers

# Re-enable server```

mcpjungle enable server filesystem

```### 6️⃣ Configure Your MCP Client



### Remove ServersAccess the gateway at: `https://mcp-gateway.YOUR-TAILNET.ts.net`



```bashSee **[Tailscale Services Guide](docs/TAILSCALE_SERVICES.md)** for detailed client configuration.

# Deregister a server (removes all its tools)

mcpjungle deregister filesystem---

```

## 🔧 Detailed Setup

---

For comprehensive setup instructions including:

## 🎯 Tool Groups- Tailscale ACL configuration

- Service discovery setup

Create subsets of tools for different MCP clients to avoid overwhelming them with too many tools:- Multiple MCP server deployment

- Access control configuration

### Create a Tool Group

👉 **See the [Tailscale Services Guide](docs/TAILSCALE_SERVICES.md)**

```bash

# Create a tool group config---

cat > claude-tools.json <<EOF

{## 🔐 Network and Security

  "name": "claude-tools",

  "description": "Tools for Claude Desktop",| Component              | Access Method          | Security                      |

  "included_tools": [| ---------------------- | ---------------------- | ----------------------------- |

    "filesystem__read_file",| **Gateway (HTTPS)**    | Tailscale MagicDNS     | Zero Trust, Tailscale ACLs    |

    "filesystem__write_file",| **Inspector**          | /inspector path        | Same as gateway               |

    "context7__get-library-docs"| **Markitdown**         | Internal only          | Via Gateway                   |

  ]| **Tailscale Network**  | Mesh VPN               | WireGuard® encrypted          |

}| **TLS Certificates**   | Auto-provisioned       | Let's Encrypt via Tailscale   |

EOF

> 🔒 **Security**: All traffic is encrypted end-to-end. No ports are exposed to the public internet. Access is controlled via Tailscale ACLs.

# Create the groupdocker compose up -d

mcpjungle create group -c claude-tools.json```



# The group is now available at:### 4️⃣ Verify Containers

# http://localhost:8080/v0/groups/claude-tools/mcp

``````bash

docker ps

### Use Tool Group in MCP Client```



Configure your MCP client to use the group-specific endpoint:You should see:



```json```

{mcp-gateway

  "mcpServers": {mcp-markitdown

    "claude-tools": {```

      "command": "npx",

      "args": [---

        "mcp-remote",

        "http://localhost:8080/v0/groups/claude-tools/mcp",## 🔐 Network and Security

        "--allow-http"

      ]| Component          | Access                     | Notes                              |

    }| ------------------ | -------------------------- | ---------------------------------- |

  }| **Gateway (6277)** | Local LAN or via Tailscale | Main MCP endpoint                  |

}| **Gateway (6274)** | Local LAN or via Tailscale | Optional Inspector/UI endpoint     |

```| **Markitdown**     | Internal only              | Accessed through the Gateway       |

| **Tailscale**      | Mesh VPN                   | Used to securely reach Komodo host |

### Manage Tool Groups

> 🧱 Tip: In pfSense or Komodo network policies, restrict inbound access to **Tailscale subnet only** for ports `6274` and `6277`.

```bash

# List all tool groups---

mcpjungle list groups

## 🧩 Connecting Clients

# View group details

mcpjungle get group claude-toolsOnce the stack is running, point your MCP-compatible client to the Gateway:



# Delete a group### Example – Claude Desktop

mcpjungle delete group claude-tools

Edit your MCP config file (typically `~/.mcp/config.json`):

# List tools in a specific group

mcpjungle list tools --group claude-tools```json

```{

  "mcpServers": {

---    "gateway": {

      "url": "http://<TAILSCALE-IP>:6277"

## ⚙️ Configuration    }

  }

### Environment Variables}

```

Create a `.env` file in the project directory:

Now Claude can automatically discover tools (like `markitdown`) via the Gateway.

```bash

# PostgreSQL Configuration---

POSTGRES_PASSWORD=your_secure_password_here

## 🧠 Enabling Additional MCP Servers

# Server Mode

# - development: No authentication, ideal for personal use**See the complete guide:** [docs/ADDING_MCP_SERVERS.md](docs/ADDING_MCP_SERVERS.md)

# - enterprise: Authentication + access control for multi-user

SERVER_MODE=developmentYou have **two options** for adding MCP servers:



# Optional: OpenTelemetry Metrics### Option 1: Declarative (compose.yaml) - Recommended for Production

OTEL_ENABLED=false

OTEL_RESOURCE_ATTRIBUTES=deployment.environment.name=homelabAdd servers directly to `compose.yaml` for automatic deployment:



# Your MCP server credentials (if needed for registration)```yaml

PROXMOX_API_TOKEN=your_proxmox_tokenatlassian:

TAILSCALE_API_KEY=your_tailscale_key  image: mcp/atlassian:latest

CONFLUENCE_API_TOKEN=your_confluence_token  container_name: mcp-atlassian

JIRA_API_TOKEN=your_jira_token  stdin_open: true

```  tty: true

  environment:

### Enterprise Mode (Multi-User)    - CONFLUENCE_URL=${CONFLUENCE_URL}

    - JIRA_URL=${JIRA_URL}

For environments with multiple users who need different access levels:    # ... more config

  networks: [mcp]

```bash```

# Set enterprise mode in .env

SERVER_MODE=enterprise**Benefits:**

- ✅ Servers start automatically with the stack

# Restart the stack- ✅ Configuration via `.env` file

docker compose up -d- ✅ Version controlled and reproducible



# Initialize admin user (run once after first startup)See `compose.yaml` for examples:

mcpjungle init-server- **Atlassian** (Jira + Confluence) - Requires API tokens

- **OpenAPI** (API spec tools) - No configuration needed

# This creates an admin user and stores the access token in ~/.mcpjungle.conf

### Option 2: Runtime (MCP Catalog) - Good for Testing

# Create MCP clients with specific access permissions

mcpjungle create mcp-client cursor-user1 --allow "filesystem,context7"### When Running with Docker Compose (Komodo):

mcpjungle create mcp-client claude-user2 --allow "proxmox,tailscale"

Since the Gateway is running in a container, you need to execute the MCP commands **inside the Gateway container**:

# Each client gets a unique access token

# Configure the client to send: Authorization: Bearer <token>```bash

```# List available servers in the catalog

docker exec mcp-gateway docker mcp server catalog

**Claude Config with Authentication:**

```json# Enable a server (e.g., OpenAPI for API development)

{docker exec mcp-gateway docker mcp server enable openapi

  "mcpServers": {

    "mcpjungle": {# Or Atlassian for Jira/Confluence

      "command": "npx",docker exec mcp-gateway docker mcp server enable atlassian

      "args": [

        "mcp-remote",# Configure the server (interactive, if needed)

        "http://localhost:8080/mcp",docker exec -it mcp-gateway docker mcp server configure atlassian

        "--allow-http",

        "--header", "Authorization: Bearer YOUR_TOKEN_HERE"# List enabled servers

      ]docker exec mcp-gateway docker mcp server list

    }

  }# View server status

}docker exec mcp-gateway docker mcp server status openapi

``````



---### Common Servers to Enable:



## 📊 Monitoring & Troubleshooting```bash

# OpenAPI Toolkit - 5 tools (validate specs, generate code/cURL)

### Health Checksdocker exec mcp-gateway docker mcp server enable openapi



```bash# Atlassian (Jira + Confluence) - 37 tools

# Check gateway healthdocker exec mcp-gateway docker mcp server enable atlassian

curl http://localhost:8080/health

# Obsidian vault management - 12 tools (requires REST API plugin)

# Check PostgreSQLdocker exec mcp-gateway docker mcp server enable obsidian

docker compose exec postgres pg_isready -U mcpjungle

# Reddit integration - 6 tools (fetch posts, comments, search)

# View all service statusdocker exec mcp-gateway docker mcp server enable mcp-reddit

docker compose ps

```# Context7 library documentation - 2 tools (up-to-date docs)

docker exec mcp-gateway docker mcp server enable context7

### View Logs

# OpenAPI Schema analysis - 10 tools (spec analysis)

```bashdocker exec mcp-gateway docker mcp server enable openapi-schema

# Gateway logs

docker compose logs -f mcpjungle# Wikipedia knowledge - 11 tools (search, articles, summaries)

docker exec mcp-gateway docker mcp server enable wikipedia-mcp

# PostgreSQL logs

docker compose logs -f postgres# Komodo container management - 15 tools (manage THIS Komodo instance!)

docker exec mcp-gateway docker mcp server enable komodo-mcp

# All services

docker compose logs -f# Proxmox hypervisor management - 6 tools (manage VMs, nodes, clusters)

docker exec mcp-gateway docker mcp server enable proxmox-mcp

# Filter for errors

docker compose logs mcpjungle | grep -i error# Tailscale network management - 20+ tools (manage devices, ACLs, status)

```docker exec mcp-gateway docker mcp server enable tailscale-mcp



### Common Issues# GitHub integration

docker exec mcp-gateway docker mcp server enable github

#### STDIO Server Failures

# Web browsing capabilities

If STDIO servers fail or throw errors, check the MCPJungle logs for `stderr` output:docker exec mcp-gateway docker mcp server enable web-browse



```bash# PostgreSQL database tools

docker compose logs mcpjungle | grep -i "stderr"docker exec mcp-gateway docker mcp server enable postgres

```

# Filesystem operations

#### Filesystem Accessdocker exec mcp-gateway docker mcp server enable filesystem

```

The gateway container has `/srv/mcp/workspace` on your host mounted as `/host` inside the container.

Each new server is added to the same `mcp` network defined in the Compose file.

When registering the filesystem MCP server, use `/host` as the path:The Gateway will automatically discover and route to them.



```json### From Komodo UI:

{

  "name": "filesystem",Komodo also provides a terminal for your stack/container. You can:

  "transport": "stdio",

  "command": "npx",1. Navigate to your **HL_DockerMCPGateway** stack

  "args": ["-y", "@modelcontextprotocol/server-filesystem", "/host"]2. Click on the **mcp-gateway** container

}3. Open the **Terminal** tab

```4. Run commands directly: `docker mcp server enable atlassian`



#### Database Connection Issues---



```bash## 🧰 Troubleshooting

# Check if PostgreSQL is ready

docker compose exec postgres pg_isready -U mcpjungleFor common issues and detailed solutions, see the **[Troubleshooting Guide](docs/TROUBLESHOOTING.md)**.



# Restart the database**Quick Fixes**:

docker compose restart postgres

| Issue                      | Quick Fix                                             |

# Check database logs| -------------------------- | ----------------------------------------------------- |

docker compose logs postgres| Client can't see any tools | Check Tailscale IP and port 6277                      |

```| Conversion fails           | Verify `/srv/mcp/workspace` mapping                   |

| Markitdown not starting    | Run `docker compose pull`                             |

### Metrics (if enabled)| Gateway state missing      | Recreate with `docker volume create mcp-gateway-data` |

| High CPU/Memory usage      | Add resource limits in compose.yaml                   |

```bash

# View Prometheus metrics---

curl http://localhost:8080/metrics

```## 🧱 System Diagram



---```

        ┌────────────────────────┐

## 🔐 Security Considerations        │        Clients         │

        │ (Claude, Cursor, etc.) │

### Development Mode (Default)        └──────────┬─────────────┘

- No authentication required                   │  HTTP/SSE (Tailscale)

- All MCP clients have full access to all servers                   ▼

- Ideal for personal homelab use        ┌────────────────────────┐

- Gateway not exposed to internet        │     MCP Gateway        │  (port 6277)

        │ docker/mcp-gateway     │

### Enterprise Mode        └──────────┬─────────────┘

- Authentication required for all requests                   │ Docker Network (mcp)

- Per-client access control lists                   ▼

- Audit logging enabled        ┌────────────────────────┐

- Recommended for multi-user environments        │   Markitdown Server    │

        │   mcp/markitdown       │

### Docker Security        └────────────────────────┘

- PostgreSQL data in named volume (persistent)```

- Gateway has Docker socket access (to manage STDIO servers)

- Use strong PostgreSQL password---

- Consider running behind reverse proxy with TLS

## 🧩 Maintenance Commands

---

Stop all services:

## 📚 Resources

```bash

- [MCPJungle GitHub](https://github.com/mcpjungle/MCPJungle)docker compose down

- [MCPJungle Documentation](https://github.com/mcpjungle/MCPJungle#readme)```

- [Model Context Protocol](https://modelcontextprotocol.io/)

- [MCP Server Registry](https://github.com/modelcontextprotocol/servers)Update images:

- [MCPJungle Discord](https://discord.gg/CapV4Z3krk)

```bash

---docker compose pull && docker compose up -d

```

## 🛠️ Advanced Usage

View logs:

### Prompts

```bash

MCPJungle supports MCP Prompts (templates for common tasks):docker compose logs -f mcp-gateway

```

```bash

# List prompts from a server---

mcpjungle list prompts --server huggingface

## 🪪 License

# Get a specific prompt

mcpjungle get prompt "huggingface__Model Details" \MIT License — See [LICENSE](LICENSE) file for details.

  --arg model_id="openai/gpt-oss-120b"

```---



### Custom STDIO Servers## 👤 Author



If your STDIO servers need additional dependencies beyond `npx` or `uvx`, create a custom Docker image:**Matt Markham**  

Titan America Digital Transformation — Homelab / AI Integration Stack

```dockerfile

FROM mcpjungle/mcpjungle:latest-stdio---



# Add your custom dependencies## 🤝 Contributing

RUN apk add --no-cache python3 py3-pip

RUN pip3 install your-mcp-server-packageContributions, issues, and feature requests are welcome! Feel free to check the [issues page](https://github.com/YOUR-USERNAME/HL_DockerMCPGateway/issues).



# Use this image in docker-compose.yaml---

```

## ⭐ Support

### Backup & Restore

If you find this project helpful, please give it a ⭐ on GitHub!

```bash
# Backup PostgreSQL data
docker compose exec postgres pg_dump -U mcpjungle mcpjungle > backup.sql

# Restore from backup
docker compose exec -T postgres psql -U mcpjungle mcpjungle < backup.sql
```

---

## 🤝 Contributing

Contributions are welcome! Please feel free to submit a Pull Request.

---

## 📄 License

MIT License - See LICENSE file for details

---

## 🙏 Acknowledgments

- [MCPJungle Team](https://github.com/mcpjungle) for the excellent MCP gateway
- [Anthropic](https://www.anthropic.com/) for the Model Context Protocol specification
- All MCP server developers in the community
