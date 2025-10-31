# Tailscale Setup Guide

This guide explains how to securely access your MCP Gateway through Tailscale.

## Prerequisites

- Tailscale installed on the Komodo host
- Tailscale installed on client devices
- Both devices connected to the same Tailnet

## Host Configuration

### 1. Install Tailscale on Komodo Host

```bash
# Ubuntu/Debian
curl -fsSL https://tailscale.com/install.sh | sh

# Start Tailscale
sudo tailscale up

# Verify connection
tailscale status
```

### 2. Get Host Tailscale IP

```bash
tailscale ip -4
# Output example: 100.x.x.x
```

Note this IP address for client configuration.

### 3. Enable Subnet Routing (Optional)

If you want to access other devices on your LAN through Tailscale:

```bash
sudo tailscale up --advertise-routes=192.168.1.0/24 --accept-routes
```

Replace `192.168.1.0/24` with your LAN subnet.

### 4. Set Up ACLs (Recommended)

In your Tailscale admin console (https://login.tailscale.com/admin/acls):

```json
{
  "acls": [
    {
      "action": "accept",
      "src": ["autogroup:member"],
      "dst": ["tag:mcp-gateway:6274,6277"]
    }
  ],
  "tagOwners": {
    "tag:mcp-gateway": ["autogroup:admin"]
  }
}
```

This restricts MCP Gateway port access to authenticated Tailscale users only.

## Client Configuration

### 1. Install Tailscale on Client

Visit https://tailscale.com/download and install for your platform:
- **Windows**: Download installer
- **macOS**: `brew install tailscale`
- **Linux**: `curl -fsSL https://tailscale.com/install.sh | sh`

### 2. Connect to Tailnet

```bash
tailscale up
# Follow authentication prompts
```

### 3. Verify Connection to Host

```bash
# Ping the host
tailscale ping <HOST-IP>

# Test MCP Gateway port
curl http://<HOST-TAILSCALE-IP>:6277
```

## MCP Client Configuration

### Claude Desktop

Edit `~/.mcp/config.json` (or similar):

```json
{
  "mcpServers": {
    "gateway": {
      "url": "http://100.x.x.x:6277",
      "timeout": 30000
    }
  }
}
```

Replace `100.x.x.x` with your host's Tailscale IP.

### Cursor / VS Code

Add to your MCP configuration:

```json
{
  "mcp.servers": {
    "gateway": {
      "command": "mcp-client",
      "args": ["--url", "http://100.x.x.x:6277"]
    }
  }
}
```

### GitHub Copilot

Configure in `settings.json`:

```json
{
  "github.copilot.mcp.gateway": "http://100.x.x.x:6277"
}
```

## Security Best Practices

### 1. Use Tailscale DNS

Enable MagicDNS in Tailscale for friendly hostnames:

```bash
# In Tailscale admin console
DNS → Enable MagicDNS

# Then use hostname instead of IP
http://komodo-host:6277
```

### 2. Enable Tailscale SSH (Optional)

For secure SSH access:

```bash
sudo tailscale up --ssh
```

### 3. Set Up Key Expiry

Configure automatic key expiry for enhanced security:

1. Go to Tailscale admin console
2. Settings → Keys
3. Set key expiry (e.g., 90 days)

### 4. Use Exit Nodes (Optional)

Route all traffic through Tailscale:

```bash
tailscale up --exit-node=<EXIT-NODE-IP>
```

## Firewall Configuration

### pfSense Integration

If using pfSense with your Komodo host:

1. **Create Tailscale Interface Alias**:
   - Interfaces → Assignments
   - Add Tailscale interface

2. **Create Firewall Rules**:
   ```
   Interface: Tailscale
   Protocol: TCP
   Source: Tailscale Net
   Destination: LAN Net (Komodo Host)
   Destination Port: 6274, 6277
   Action: Allow
   ```

3. **Block External Access**:
   ```
   Interface: WAN
   Protocol: TCP
   Source: Any
   Destination: Komodo Host
   Destination Port: 6274, 6277
   Action: Block
   ```

### iptables Rules

If managing firewall directly on host:

```bash
# Allow Tailscale interface
sudo iptables -A INPUT -i tailscale0 -p tcp --dport 6277 -j ACCEPT
sudo iptables -A INPUT -i tailscale0 -p tcp --dport 6274 -j ACCEPT

# Block external access (adjust interface name)
sudo iptables -A INPUT -i eth0 -p tcp --dport 6277 -j DROP
sudo iptables -A INPUT -i eth0 -p tcp --dport 6274 -j DROP

# Save rules
sudo iptables-save > /etc/iptables/rules.v4
```

## Troubleshooting

### Cannot Connect to Gateway

1. **Verify Tailscale is running**:
   ```bash
   tailscale status
   ```

2. **Check connectivity**:
   ```bash
   tailscale ping <host-ip>
   ```

3. **Test port access**:
   ```bash
   curl -v http://<host-tailscale-ip>:6277
   ```

4. **Check firewall rules**:
   ```bash
   sudo iptables -L -n
   ```

### Slow Connection

1. **Use direct connection** (not relay):
   ```bash
   tailscale status
   # Look for "direct" not "relay"
   ```

2. **Enable DERP map optimization**:
   - Tailscale admin → Advanced → Enable nearest DERP

3. **Check latency**:
   ```bash
   tailscale ping <host-ip>
   ```

### Authentication Issues

1. **Re-authenticate**:
   ```bash
   tailscale up --force-reauth
   ```

2. **Check ACLs**:
   - Verify your user has access in Tailscale admin console

3. **Check key expiry**:
   ```bash
   tailscale status
   # Look for expiration warnings
   ```

## Monitoring

### Connection Status

```bash
# Check Tailscale status
tailscale status

# View network info
tailscale netcheck

# Monitor logs
journalctl -u tailscaled -f
```

### Performance Testing

```bash
# Test bandwidth
iperf3 -c <host-tailscale-ip>

# Test latency
tailscale ping <host-ip>

# Check MTU
ping -M do -s 1472 <host-tailscale-ip>
```

## Mobile Access

### iOS/Android Setup

1. Install Tailscale app from App Store/Google Play
2. Sign in to your Tailnet
3. Configure MCP client (if available on mobile)

Note: Direct MCP access on mobile may be limited depending on client availability.

## Advanced Configuration

### Custom DERP Servers

For lowest latency, run your own DERP server:

```bash
# See Tailscale documentation
https://tailscale.com/kb/1118/custom-derp-servers/
```

### Subnet Routing for Multiple Services

```bash
sudo tailscale up \
  --advertise-routes=192.168.1.0/24 \
  --advertise-tags=tag:homelab \
  --accept-routes
```

### Tailscale Funnel (Public Access)

⚠️ **Not recommended for MCP Gateway** - but available if needed:

```bash
sudo tailscale funnel 6277
```

This exposes the service publicly. Only use in trusted scenarios.

## Additional Resources

- [Tailscale Documentation](https://tailscale.com/kb/)
- [Tailscale ACL Reference](https://tailscale.com/kb/1018/acls/)
- [Tailscale Best Practices](https://tailscale.com/kb/1019/subnets/)
