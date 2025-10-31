# 🧠 MCP Gateway Stack (Komodo Deployment)

[![Docker Compose](https://img.shields.io/badge/docker--compose-v3.9-blue.svg)](https://docs.docker.com/compose/)
[![Tailscale](https://img.shields.io/badge/Tailscale-Enabled-blue.svg)](https://tailscale.com)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![PRs Welcome](https://img.shields.io/badge/PRs-welcome-brightgreen.svg)](CONTRIBUTING.md)
[![Maintenance](https://img.shields.io/badge/Maintained%3F-yes-green.svg)](https://github.com/YOUR-USERNAME/HL_DockerMCPGateway/graphs/commit-activity)

This repository contains the Docker Compose configuration for running the **Docker MCP Gateway** and the **markitdown-mcp** server inside a **homelab environment**, managed by **Komodo** and exposed as **Tailscale Services** for secure, easy access.

---

## 📦 Overview

The [**Docker MCP Gateway**](https://github.com/docker/mcp-gateway) provides a unified endpoint for Model Context Protocol (MCP) clients such as Claude Desktop, Cursor, and GitHub Copilot MCP.  
It aggregates multiple MCP servers (tools) and exposes them to clients through a single interface.

### Key Features

- 🔐 **Tailscale Integration**: HTTPS access via tsdproxy or Tailscale Serve
- 🎯 **Flexible Deployment**: Use existing tsdproxy or dedicated sidecar
- 🔒 **Zero Trust Security**: Access controlled via Tailscale ACLs
- 📡 **No Port Forwarding**: No firewall configuration needed
- 🚀 **Auto-provisioned TLS**: Certificates managed by Tailscale
- 🎛️ **Komodo Managed**: Easy deployment and monitoring
- 🔍 **Auto-Discovery**: Finds MCP servers on Docker network

### Architecture

```
┌─────────────────────────────────────────────────────┐
│              Your Tailscale Network                  │
│                                                      │
│  ┌──────────────┐      ┌──────────────────────┐    │
│  │ MCP Clients  │─────▶│ mcp-gateway          │    │
│  │ (Claude,etc) │ HTTPS│ .your-tailnet.ts.net │    │
│  └──────────────┘      └──────────┬───────────┘    │
│                                    │                │
│                         ┌──────────▼───────────┐    │
│                         │     tsdproxy         │    │
│                         │  (Host Process)      │    │
│                         └──────────┬───────────┘    │
│                                    │                │
│                         ┌──────────▼───────────┐    │
│                         │   MCP Gateway        │    │
│                         │   Container :3000    │    │
│                         └──────────┬───────────┘    │
│                                    │                │
│                         ┌──────────▼───────────┐    │
│                         │   MCP Servers        │    │
│                         │   + markitdown       │    │
│                         │   + proxmox-mcp      │    │
│                         │   + tailscale-mcp    │    │
│                         └──────────────────────┘    │
│                                                      │
└─────────────────────────────────────────────────────┘
                 Managed by Komodo
```

In this setup:

- **tsdproxy (Host Process)**  
  Provides secure networking and automatic HTTPS via Tailscale.  
  Uses your host's existing Tailscale connection.

- **Gateway Container (`mcp-gateway`)**  
  Hosts the MCP interface and manages tool discovery, configuration, and routing.  
  Exposed on port 3000 for tsdproxy to proxy.

- **MCP Servers**  
  - **Markitdown**: Converts files (PDF, DOCX, HTML, etc.) into Markdown
  - **Proxmox MCP**: Manages VMs, containers, and nodes in your hypervisor
  - **Tailscale MCP**: Automates your Tailscale network configuration
  - Many more available via profiles

- **Komodo** orchestrates containers as a managed stack.  
- **tsdproxy** makes the gateway accessible with HTTPS on your tailnet.

---

## 🗂 Directory Structure

```
.
├── compose.yaml             # MCP Gateway + MCP servers
├── tailscale/
│   └── serve-config.json    # Tailscale Serve config (if using sidecar)
├── README.md                # This file
├── LICENSE                  # MIT License
├── .gitignore               # Git ignore rules
├── .env.example             # Environment template
├── docs/
│   ├── KOMODO_SETUP.md      # Komodo deployment guide
│   ├── TAILSCALE_SETUP.md   # Basic Tailscale configuration
│   ├── TSDPROXY_SETUP.md    # tsdproxy integration guide (RECOMMENDED!)
│   ├── TAILSCALE_SERVICES.md # Alternative: Tailscale Services with sidecar
│   ├── TROUBLESHOOTING.md   # Common issues and solutions
│   └── WORKSPACE_SETUP.md   # Workspace configuration
└── workspace/               # Shared folder mounted read-only
```

- **`workspace/`**:  
  Place any files here that you want to make accessible to MCP tools for conversion or processing.

- **`docs/TSDPROXY_SETUP.md`**:  
  **RECOMMENDED**: Guide for integrating with your existing tsdproxy setup.

- **`tailscale/serve-config.json`** (optional):  
  If using Tailscale sidecar instead of tsdproxy, configures Tailscale Serve.

---

## ⚙️ Prerequisites

- **Komodo** installed and configured on the host — See [Komodo Setup Guide](docs/KOMODO_SETUP.md)
- **Docker** and **Docker Compose v2.20+** available
- **Tailscale** running on the host with tsdproxy configured
  - **OR** a Tailscale Account with auth key for sidecar deployment
- Sign up at https://tailscale.com if you don't have an account
- **Tailscale Auth Key** — Generate at https://login.tailscale.com/admin/settings/keys
- **Services Enabled** — Enable at https://login.tailscale.com/admin/services
- Optional: a **Tailscale ACL** restricting access to TCP ports 6274–6277

## 📚 Documentation

- **[tsdproxy Integration Guide](docs/TSDPROXY_SETUP.md)** - **⭐ RECOMMENDED** - Use your existing tsdproxy setup
- **[Tailscale Services Guide](docs/TAILSCALE_SERVICES.md)** - Alternative: Deploy with dedicated Tailscale sidecar
- **[Komodo Setup Guide](docs/KOMODO_SETUP.md)** - Detailed instructions for deploying with Komodo
- **[Adding MCP Servers](docs/ADDING_MCP_SERVERS.md)** - How to add and configure additional MCP servers
- **[Troubleshooting Guide](docs/TROUBLESHOOTING.md)** - Common issues and solutions
- **[Workspace Setup](docs/WORKSPACE_SETUP.md)** - Configure the workspace directory

---

## 🚀 Quick Start (with tsdproxy)

### 1️⃣ Clone This Repository
```bash
git clone https://github.com/mdlmarkham/HL_DockerMCPGateway.git
cd HL_DockerMCPGateway
```

### 2️⃣ Configure Environment Variables

```bash
# Copy the example environment file
cp .env.example .env

# Edit .env (optional - defaults work for most setups)
nano .env
```

Optional variables:
- `MCP_GATEWAY_PORT`: Gateway port (default: 3000)
- `MCP_WORKSPACE_PATH`: Path to your workspace directory

### 3️⃣ Configure tsdproxy

Add the MCP Gateway backend to your tsdproxy configuration:

```bash
# Example for tsdproxy systemd service
sudo tee -a /etc/tsdproxy/config.yaml << EOF
backends:
  mcp-gateway:
    url: http://localhost:3000
    hostname: mcp-gateway.your-tailnet.ts.net
EOF

sudo systemctl restart tsdproxy
```

**See [tsdproxy Integration Guide](docs/TSDPROXY_SETUP.md) for detailed instructions.**

### 4️⃣ Create Workspace Directory (Optional)

```bash
# Create the workspace directory
sudo mkdir -p /srv/mcp/workspace
sudo chmod 755 /srv/mcp/workspace
```

Or update `compose.yaml` to use your preferred path.

### 5️⃣ Launch the Stack

```bash
# Start the gateway and base services
docker compose up -d

# Check status
docker compose ps

# View logs
docker compose logs -f mcp-gateway
```

### 6️⃣ Verify the Setup

```bash
# Test locally
curl http://localhost:3000/health

# Test via Tailscale
curl https://mcp-gateway.your-tailnet.ts.net/health
```

### 7️⃣ Enable Optional MCP Servers

```bash
# Start Proxmox MCP server (after configuring .env)
docker compose --profile proxmox up -d

# Start Tailscale MCP server (after configuring .env)
docker compose --profile tailscale-mcp up -d

# View available MCP tools
docker compose logs mcp-gateway | grep -i tool
```

### 6️⃣ Configure Your MCP Client

Access the gateway at: `https://mcp-gateway.YOUR-TAILNET.ts.net`

See **[Tailscale Services Guide](docs/TAILSCALE_SERVICES.md)** for detailed client configuration.

---

## 🔧 Detailed Setup

For comprehensive setup instructions including:
- Tailscale ACL configuration
- Service discovery setup
- Multiple MCP server deployment
- Access control configuration

👉 **See the [Tailscale Services Guide](docs/TAILSCALE_SERVICES.md)**

---

## 🔐 Network and Security

| Component              | Access Method          | Security                      |
| ---------------------- | ---------------------- | ----------------------------- |
| **Gateway (HTTPS)**    | Tailscale MagicDNS     | Zero Trust, Tailscale ACLs    |
| **Inspector**          | /inspector path        | Same as gateway               |
| **Markitdown**         | Internal only          | Via Gateway                   |
| **Tailscale Network**  | Mesh VPN               | WireGuard® encrypted          |
| **TLS Certificates**   | Auto-provisioned       | Let's Encrypt via Tailscale   |

> 🔒 **Security**: All traffic is encrypted end-to-end. No ports are exposed to the public internet. Access is controlled via Tailscale ACLs.
docker compose up -d
```

### 4️⃣ Verify Containers

```bash
docker ps
```

You should see:

```
mcp-gateway
mcp-markitdown
```

---

## 🔐 Network and Security

| Component          | Access                     | Notes                              |
| ------------------ | -------------------------- | ---------------------------------- |
| **Gateway (6277)** | Local LAN or via Tailscale | Main MCP endpoint                  |
| **Gateway (6274)** | Local LAN or via Tailscale | Optional Inspector/UI endpoint     |
| **Markitdown**     | Internal only              | Accessed through the Gateway       |
| **Tailscale**      | Mesh VPN                   | Used to securely reach Komodo host |

> 🧱 Tip: In pfSense or Komodo network policies, restrict inbound access to **Tailscale subnet only** for ports `6274` and `6277`.

---

## 🧩 Connecting Clients

Once the stack is running, point your MCP-compatible client to the Gateway:

### Example – Claude Desktop

Edit your MCP config file (typically `~/.mcp/config.json`):

```json
{
  "mcpServers": {
    "gateway": {
      "url": "http://<TAILSCALE-IP>:6277"
    }
  }
}
```

Now Claude can automatically discover tools (like `markitdown`) via the Gateway.

---

## 🧠 Enabling Additional MCP Servers

**See the complete guide:** [docs/ADDING_MCP_SERVERS.md](docs/ADDING_MCP_SERVERS.md)

You have **two options** for adding MCP servers:

### Option 1: Declarative (compose.yaml) - Recommended for Production

Add servers directly to `compose.yaml` for automatic deployment:

```yaml
atlassian:
  image: mcp/atlassian:latest
  container_name: mcp-atlassian
  stdin_open: true
  tty: true
  environment:
    - CONFLUENCE_URL=${CONFLUENCE_URL}
    - JIRA_URL=${JIRA_URL}
    # ... more config
  networks: [mcp]
```

**Benefits:**
- ✅ Servers start automatically with the stack
- ✅ Configuration via `.env` file
- ✅ Version controlled and reproducible

See `compose.yaml` for examples:
- **Atlassian** (Jira + Confluence) - Requires API tokens
- **OpenAPI** (API spec tools) - No configuration needed

### Option 2: Runtime (MCP Catalog) - Good for Testing

### When Running with Docker Compose (Komodo):

Since the Gateway is running in a container, you need to execute the MCP commands **inside the Gateway container**:

```bash
# List available servers in the catalog
docker exec mcp-gateway docker mcp server catalog

# Enable a server (e.g., OpenAPI for API development)
docker exec mcp-gateway docker mcp server enable openapi

# Or Atlassian for Jira/Confluence
docker exec mcp-gateway docker mcp server enable atlassian

# Configure the server (interactive, if needed)
docker exec -it mcp-gateway docker mcp server configure atlassian

# List enabled servers
docker exec mcp-gateway docker mcp server list

# View server status
docker exec mcp-gateway docker mcp server status openapi
```

### Common Servers to Enable:

```bash
# OpenAPI Toolkit - 5 tools (validate specs, generate code/cURL)
docker exec mcp-gateway docker mcp server enable openapi

# Atlassian (Jira + Confluence) - 37 tools
docker exec mcp-gateway docker mcp server enable atlassian

# Obsidian vault management - 12 tools (requires REST API plugin)
docker exec mcp-gateway docker mcp server enable obsidian

# Reddit integration - 6 tools (fetch posts, comments, search)
docker exec mcp-gateway docker mcp server enable mcp-reddit

# Context7 library documentation - 2 tools (up-to-date docs)
docker exec mcp-gateway docker mcp server enable context7

# OpenAPI Schema analysis - 10 tools (spec analysis)
docker exec mcp-gateway docker mcp server enable openapi-schema

# Wikipedia knowledge - 11 tools (search, articles, summaries)
docker exec mcp-gateway docker mcp server enable wikipedia-mcp

# Komodo container management - 15 tools (manage THIS Komodo instance!)
docker exec mcp-gateway docker mcp server enable komodo-mcp

# Proxmox hypervisor management - 6 tools (manage VMs, nodes, clusters)
docker exec mcp-gateway docker mcp server enable proxmox-mcp

# Tailscale network management - 20+ tools (manage devices, ACLs, status)
docker exec mcp-gateway docker mcp server enable tailscale-mcp

# GitHub integration
docker exec mcp-gateway docker mcp server enable github

# Web browsing capabilities
docker exec mcp-gateway docker mcp server enable web-browse

# PostgreSQL database tools
docker exec mcp-gateway docker mcp server enable postgres

# Filesystem operations
docker exec mcp-gateway docker mcp server enable filesystem
```

Each new server is added to the same `mcp` network defined in the Compose file.
The Gateway will automatically discover and route to them.

### From Komodo UI:

Komodo also provides a terminal for your stack/container. You can:

1. Navigate to your **HL_DockerMCPGateway** stack
2. Click on the **mcp-gateway** container
3. Open the **Terminal** tab
4. Run commands directly: `docker mcp server enable atlassian`

---

## 🧰 Troubleshooting

For common issues and detailed solutions, see the **[Troubleshooting Guide](docs/TROUBLESHOOTING.md)**.

**Quick Fixes**:

| Issue                      | Quick Fix                                             |
| -------------------------- | ----------------------------------------------------- |
| Client can't see any tools | Check Tailscale IP and port 6277                      |
| Conversion fails           | Verify `/srv/mcp/workspace` mapping                   |
| Markitdown not starting    | Run `docker compose pull`                             |
| Gateway state missing      | Recreate with `docker volume create mcp-gateway-data` |
| High CPU/Memory usage      | Add resource limits in compose.yaml                   |

---

## 🧱 System Diagram

```
        ┌────────────────────────┐
        │        Clients         │
        │ (Claude, Cursor, etc.) │
        └──────────┬─────────────┘
                   │  HTTP/SSE (Tailscale)
                   ▼
        ┌────────────────────────┐
        │     MCP Gateway        │  (port 6277)
        │ docker/mcp-gateway     │
        └──────────┬─────────────┘
                   │ Docker Network (mcp)
                   ▼
        ┌────────────────────────┐
        │   Markitdown Server    │
        │   mcp/markitdown       │
        └────────────────────────┘
```

---

## 🧩 Maintenance Commands

Stop all services:

```bash
docker compose down
```

Update images:

```bash
docker compose pull && docker compose up -d
```

View logs:

```bash
docker compose logs -f mcp-gateway
```

---

## 🪪 License

MIT License — See [LICENSE](LICENSE) file for details.

---

## 👤 Author

**Matt Markham**  
Titan America Digital Transformation — Homelab / AI Integration Stack

---

## 🤝 Contributing

Contributions, issues, and feature requests are welcome! Feel free to check the [issues page](https://github.com/YOUR-USERNAME/HL_DockerMCPGateway/issues).

---

## ⭐ Support

If you find this project helpful, please give it a ⭐ on GitHub!
