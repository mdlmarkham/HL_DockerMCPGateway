# Repository Summary

This document provides a quick overview of the HL_DockerMCPGateway repository structure and key features.

## Purpose

Deploy Docker MCP Gateway in a HomeLab environment using:
- 🐳 **Docker Compose** for containerization
- 🚀 **Komodo** for orchestration and management
- 🔒 **Tailscale Services** for secure, zero-trust networking

## Key Features

✅ **Automatic HTTPS** - TLS certificates via Tailscale  
✅ **MagicDNS Hostnames** - Access via `https://mcp-gateway.your-tailnet.ts.net`  
✅ **Service Discovery** - Appears in Tailscale admin console  
✅ **Zero Trust Security** - No public ports, ACL-based access control  
✅ **Easy Deployment** - Docker Compose + Komodo = simple setup  
✅ **Production Ready** - Security hardening, monitoring, troubleshooting docs

## Quick Links

| Document | Purpose |
|----------|---------|
| [README.md](README.md) | Main documentation and quick start |
| [TAILSCALE_SERVICES.md](docs/TAILSCALE_SERVICES.md) | **START HERE** - Complete Tailscale Services setup |
| [KOMODO_SETUP.md](docs/KOMODO_SETUP.md) | Deploy with Komodo |
| [TROUBLESHOOTING.md](docs/TROUBLESHOOTING.md) | Solve common issues |
| [compose.yaml](compose.yaml) | Docker Compose configuration |
| [.env.example](.env.example) | Environment template |

## File Structure

```
HL_DockerMCPGateway/
├── compose.yaml                    # Docker Compose config with Tailscale sidecar
├── .env.example                    # Environment template with Tailscale vars
├── tailscale/
│   ├── serve-config.json          # Tailscale Serve configuration
│   └── README.md                  # Tailscale config documentation
├── docs/
│   ├── TAILSCALE_SERVICES.md      # ⭐ Comprehensive Tailscale Services guide
│   ├── KOMODO_SETUP.md            # Komodo deployment guide
│   ├── TAILSCALE_SETUP.md         # Basic Tailscale setup
│   ├── TROUBLESHOOTING.md         # Common issues and solutions
│   └── WORKSPACE_SETUP.md         # Workspace configuration
├── .github/
│   ├── workflows/
│   │   └── validate.yml           # CI workflow
│   ├── ISSUE_TEMPLATE/            # Bug report and feature request templates
│   └── pull_request_template.md   # PR template
├── LICENSE                         # MIT License
├── CONTRIBUTING.md                 # Contribution guidelines
├── CODE_OF_CONDUCT.md             # Code of conduct
├── SECURITY.md                     # Security policy
└── CHANGELOG.md                    # Version history

```

## Architecture Overview

```
┌────────────────────────────────────────────┐
│         Your Tailscale Network              │
│                                            │
│  ┌──────────┐      ┌──────────────────┐   │
│  │  Clients │─────▶│  mcp-gateway     │   │
│  │ (Claude) │HTTPS │  .tailnet.ts.net │   │
│  └──────────┘      └────────┬─────────┘   │
│                              │             │
│                    ┌─────────▼──────────┐  │
│                    │ Tailscale Sidecar  │  │
│                    │ (Serve Enabled)    │  │
│                    └─────────┬──────────┘  │
│                              │             │
│                    ┌─────────▼──────────┐  │
│                    │  MCP Gateway       │  │
│                    │  + Markitdown      │  │
│                    └────────────────────┘  │
└────────────────────────────────────────────┘
            Managed by Komodo
```

## Technology Stack

| Component | Technology | Purpose |
|-----------|-----------|---------|
| **Container Runtime** | Docker | Run MCP Gateway and Markitdown |
| **Orchestration** | Docker Compose | Define multi-container setup |
| **Management** | Komodo | Deploy, monitor, and manage stack |
| **Networking** | Tailscale | Zero-trust VPN with MagicDNS |
| **Service Exposure** | Tailscale Serve | HTTPS with automatic TLS |
| **MCP Gateway** | docker/mcp-gateway | Aggregate MCP servers |
| **MCP Server** | mcp/markitdown | Document to Markdown conversion |

## Security Features

🔒 **Zero Trust Network** - Tailscale mesh VPN with WireGuard®  
🔒 **No Public Ports** - All traffic via Tailscale  
🔒 **Automatic TLS** - Let's Encrypt certificates via Tailscale  
🔒 **ACL-Based Access** - Fine-grained access control  
🔒 **Container Hardening** - Read-only filesystems, no-new-privileges  
🔒 **Service Discovery** - Visible only to authorized users  

## Prerequisites Checklist

Before deploying, ensure you have:

- [ ] Komodo installed and running
- [ ] Docker and Docker Compose v2.20+
- [ ] Tailscale account created
- [ ] Tailscale auth key generated
- [ ] Tailscale Services enabled
- [ ] Workspace directory created
- [ ] .env file configured

## Quick Deploy

```bash
# 1. Clone repository
git clone https://github.com/YOUR-USERNAME/HL_DockerMCPGateway.git
cd HL_DockerMCPGateway

# 2. Configure environment
cp .env.example .env
nano .env  # Add your TS_AUTHKEY

# 3. Create workspace
sudo mkdir -p /srv/mcp/workspace

# 4. Deploy
docker compose up -d

# 5. Verify
docker exec tailscale-mcp-gateway tailscale serve status
```

Access at: `https://mcp-gateway.YOUR-TAILNET.ts.net`

## Common Use Cases

### 1. Claude Desktop Integration
Configure Claude to use MCP Gateway for enhanced capabilities like document conversion, API development, project management, and knowledge retrieval.

### 2. Cursor/VS Code Integration
Enable MCP tools in your IDE for AI-assisted development with file processing, OpenAPI spec validation, Jira/Confluence integration, and real-time library documentation.

### 3. GitHub Copilot Enhancement
Extend Copilot with MCP servers for specialized tasks like API documentation, task tracking, and code library references.

### 4. API Development & Documentation
Use OpenAPI toolkit to validate specs, generate code snippets, and create cURL commands. Analyze existing schemas with detailed component inspection tools.

### 5. Project Management & Documentation
Integrate with Atlassian tools - manage Jira issues, update Confluence pages, track sprints, all through AI assistants.

### 6. Personal Knowledge Management
Connect to your Obsidian vault for note-taking, search, and content management with AI assistance.

### 7. Research & Information Gathering
Access Wikipedia for knowledge retrieval, Reddit for community insights, and web browsing for current information.

### 8. HomeLab Document Processing
Convert PDFs, DOCX, and other formats to Markdown automatically with the Markitdown server.

### 9. Team Collaboration
Share MCP tools across your organization via Tailscale ACLs for secure, private access.

### 10. Self-Service HomeLab Management
Use the Komodo MCP server to manage containers, deployments, and stacks within your own Komodo instance - meta-orchestration at its finest!

### 11. HomeLab Hypervisor Management
Manage Proxmox VE/PVE through AI assistants - monitor nodes, control VMs, check storage, and maintain cluster health conversationally.

### 12. Tailscale Network Automation
Programmatically manage your Tailscale network - automate device management, update ACLs, monitor network status, and control access through AI-driven conversations.

## Support and Community

- 📝 **Documentation**: See `/docs` directory
- 🐛 **Bug Reports**: Use GitHub Issues with bug report template
- 💡 **Feature Requests**: Use GitHub Issues with feature request template
- 🤝 **Contributing**: See [CONTRIBUTING.md](CONTRIBUTING.md)
- 🔒 **Security**: See [SECURITY.md](SECURITY.md)

## License

MIT License - See [LICENSE](LICENSE) for details.

## Maintenance Status

🟢 **Actively Maintained** - Regular updates and support provided.

## Author

**Matt Markham**  
Titan America Digital Transformation  
HomeLab / AI Integration Stack

---

## Version

Current: **v1.1.0** (Unreleased - Tailscale Services Integration)  
Previous: **v1.0.0** (Initial Release)

See [CHANGELOG.md](CHANGELOG.md) for version history.

---

**Last Updated**: 2025-10-31
