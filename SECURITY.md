# Security Policy

## Supported Versions

Currently supported versions with security updates:

| Version | Supported          |
| ------- | ------------------ |
| 1.0.x   | :white_check_mark: |

## Reporting a Vulnerability

If you discover a security vulnerability in this project, please follow these steps:

### 1. **Do Not** Open a Public Issue

Security vulnerabilities should not be disclosed publicly until a fix is available.

### 2. Report Privately

Please report security vulnerabilities by:
- **Email**: [Your security contact email - add yours]
- **GitHub Security Advisory**: Use the "Security" tab in the repository

### 3. Include Details

When reporting, please include:
- Description of the vulnerability
- Steps to reproduce
- Potential impact
- Suggested fix (if any)
- Your contact information

### 4. Response Timeline

- **Initial Response**: Within 48 hours
- **Status Update**: Within 7 days
- **Fix Timeline**: Depends on severity
  - Critical: 1-7 days
  - High: 7-14 days
  - Medium: 14-30 days
  - Low: 30-90 days

## Security Best Practices

When deploying this stack:

### Network Security
- ✅ Use Tailscale for all external access
- ✅ Never expose ports directly to the internet
- ✅ Use firewall rules to restrict access
- ✅ Keep Tailscale ACLs up to date

### Container Security
- ✅ Use the provided security options (`no-new-privileges`)
- ✅ Keep images updated regularly
- ✅ Use read-only filesystems where possible
- ✅ Limit container capabilities

### Access Control
- ✅ Use strong authentication on Komodo
- ✅ Regularly rotate Tailscale auth keys
- ✅ Monitor access logs
- ✅ Use principle of least privilege

### Updates
- ✅ Subscribe to security advisories for:
  - Docker MCP Gateway
  - Markitdown
  - Komodo
  - Tailscale
- ✅ Update regularly
- ✅ Test updates in non-production first

## Known Security Considerations

### Docker Socket Access
This stack does not require access to the Docker socket. If you modify it to add such access, be aware of the security implications.

### Port Exposure
The default configuration exposes ports 6274 and 6277. These should only be accessible via:
- Localhost
- Internal networks
- Tailscale VPN

### Volume Mounts
The workspace volume is mounted read-only by default. Ensure sensitive files are not placed in the workspace directory.

## Security Disclosure History

No security issues have been disclosed yet.

## Contact

For security concerns, please contact:
- **GitHub**: Open a private security advisory
- **Email**: [Add your security contact email]

## Acknowledgments

We appreciate responsible disclosure of security vulnerabilities. Contributors who report valid security issues may be acknowledged in our README (with permission).
