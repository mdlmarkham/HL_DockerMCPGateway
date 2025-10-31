# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

### Added
- **Tailscale Services Integration** - MCP Gateway now exposed via Tailscale Serve
  - Automatic HTTPS with TLS certificates
  - MagicDNS hostname support (e.g., https://mcp-gateway.tail1234.ts.net)
  - Service discovery in Tailscale admin console
  - Tailscale sidecar container for secure networking
- Comprehensive Tailscale Services documentation (docs/TAILSCALE_SERVICES.md)
- Tailscale Serve configuration file (tailscale/serve-config.json)
- Enhanced .env.example with Tailscale-specific variables
- Docker Compose configuration for Tailscale sidecar container
- Detailed troubleshooting for Tailscale Services
- Migration guide from direct port exposure to Tailscale Services

### Changed
- **BREAKING**: MCP Gateway now requires Tailscale auth key for deployment
- **BREAKING**: Removed direct port exposure (6277, 6274) in favor of Tailscale Serve
- Updated README with Tailscale Services architecture diagram
- Enhanced prerequisites to include Tailscale account requirements
- Updated client configuration examples to use HTTPS and MagicDNS
- Improved security by eliminating public port exposure

### Deprecated
- Direct port binding configuration (ports: 6277, 6274)
- Plain HTTP access (replaced with HTTPS via Tailscale)

## [1.0.0] - 2025-10-31

### Added
- Initial repository setup
- Docker Compose configuration for MCP Gateway and Markitdown server
- Comprehensive README with deployment instructions
- Komodo-specific setup guide
- Tailscale integration documentation
- MIT License
- Contributing guidelines
- Example environment configuration
- Security hardening in compose.yaml
- Troubleshooting guide
- Workspace setup documentation
- GitHub Actions workflow for validation
- Issue and PR templates
- Code of Conduct
- Security policy

### Security
- Added security options in Docker containers (no-new-privileges)
- Read-only container filesystems where applicable
- Network isolation using Docker networks
- Secure workspace mounting (read-only)

---

## Migration Notes (1.0.0 → Unreleased)

### For Existing Users

If you're upgrading from version 1.0.0, you'll need to:

1. **Obtain a Tailscale auth key**:
   - Visit https://login.tailscale.com/admin/settings/keys
   - Generate a reusable key with `tag:mcp-server`

2. **Update your .env file**:
   ```bash
   TS_AUTHKEY=tskey-auth-xxxxxxxxxxxxxxxxxxxxxxxxxxxxx
   TS_CERT_DOMAIN=tail1234.ts.net
   ```

3. **Pull and restart containers**:
   ```bash
   docker compose pull
   docker compose down
   docker compose up -d
   ```

4. **Update MCP client configuration**:
   - Change from: `http://100.x.y.z:6277`
   - Change to: `https://mcp-gateway.tail1234.ts.net`

5. **Remove port forwarding rules** (if any):
   - Ports 6277 and 6274 are no longer needed

See [TAILSCALE_SERVICES.md](docs/TAILSCALE_SERVICES.md) for complete migration instructions.

---

## Release Types

- **Added**: New features
- **Changed**: Changes in existing functionality
- **Deprecated**: Soon-to-be removed features
- **Removed**: Now removed features
- **Fixed**: Bug fixes
- **Security**: Vulnerability fixes or security improvements
