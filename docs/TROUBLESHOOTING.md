# Troubleshooting Guide

This guide covers common issues and their solutions when deploying the MCP Gateway Stack.

## Table of Contents
- [Container Issues](#container-issues)
- [Network Issues](#network-issues)
- [Komodo Issues](#komodo-issues)
- [Tailscale Issues](#tailscale-issues)
- [MCP Client Issues](#mcp-client-issues)
- [Performance Issues](#performance-issues)

---

## Container Issues

### Containers Won't Start

**Symptoms**: Containers exit immediately after starting

**Possible Causes**:
1. Port conflicts
2. Volume permission issues
3. Image pull failures
4. Invalid compose configuration

**Solutions**:

```bash
# Check container logs
docker compose logs mcp-gateway
docker compose logs markitdown

# Verify ports are available
sudo lsof -i :6277
sudo lsof -i :6274

# Check volume permissions
ls -la /srv/mcp/workspace

# Validate compose file
docker compose config

# Pull images manually
docker compose pull
```

### Gateway Can't Discover Markitdown

**Symptoms**: MCP clients don't see the markitdown tool

**Solutions**:

```bash
# Ensure both containers are on the same network
docker network inspect hl_dockermcpgateway_mcp

# Restart the gateway
docker compose restart mcp-gateway

# Check gateway logs for discovery messages
docker compose logs -f mcp-gateway | grep markitdown
```

### Container Keeps Restarting

**Symptoms**: Container in restart loop

**Solutions**:

```bash
# Check exit code and logs
docker compose ps
docker compose logs --tail=50 <service-name>

# Disable restart temporarily to debug
# Edit compose.yaml: change 'restart: unless-stopped' to 'restart: "no"'

# Check resource limits
docker stats
```

---

## Network Issues

### Cannot Access Gateway from Client

**Symptoms**: Connection timeout or refused

**Possible Causes**:
1. Firewall blocking
2. Wrong IP address
3. Container not running
4. Port not exposed

**Solutions**:

```bash
# Verify container is running
docker compose ps

# Check if port is listening
netstat -tulpn | grep 6277

# Test from host machine
curl http://localhost:6277

# Check firewall (Ubuntu/Debian)
sudo ufw status
sudo ufw allow 6277/tcp

# Check firewall (pfSense)
# Diagnostics → Packet Capture → Watch for blocked traffic
```

### Gateway Returns 502 or 503 Errors

**Symptoms**: Gateway accessible but returns HTTP errors

**Solutions**:

```bash
# Check if markitdown is running
docker compose ps markitdown

# Restart services
docker compose restart

# Check Docker network
docker network ls
docker network inspect hl_dockermcpgateway_mcp

# Verify DNS resolution between containers
docker exec mcp-gateway ping markitdown
```

---

## Komodo Issues

### Stack Won't Deploy in Komodo

**Symptoms**: Deployment fails in Komodo UI

**Solutions**:

1. **Check Komodo Logs**:
   ```bash
   journalctl -u komodo -n 50 -f
   ```

2. **Verify Docker Access**:
   ```bash
   # Ensure Komodo can access Docker
   sudo usermod -aG docker komodo
   sudo systemctl restart komodo
   ```

3. **Test Compose File Manually**:
   ```bash
   cd /path/to/stack
   docker compose config
   docker compose up -d
   ```

4. **Check File Permissions**:
   ```bash
   ls -la compose.yaml
   # Should be readable by Komodo user
   ```

### Stack Shows as Running but Containers Aren't

**Symptoms**: Komodo shows success but containers not running

**Solutions**:

```bash
# Check actual Docker state
docker compose ps

# Compare with Komodo state
# Check Komodo logs for sync issues

# Force refresh in Komodo
# Stack → Refresh Status

# Manually reconcile
docker compose down
# Redeploy from Komodo
```

---

## Tailscale Issues

### Cannot Connect via Tailscale

**Symptoms**: Client can't reach host over Tailscale IP

**Solutions**:

```bash
# Verify Tailscale is running on host
sudo systemctl status tailscaled

# Check Tailscale status
tailscale status

# Get current IP
tailscale ip -4

# Test connectivity
tailscale ping <host-ip>

# Check if firewall allows Tailscale
sudo iptables -L -n | grep tailscale0
```

### Slow Connection Through Tailscale

**Symptoms**: High latency or slow data transfer

**Solutions**:

```bash
# Check if using relay (DERP) instead of direct connection
tailscale status
# Look for "direct" vs "relay"

# Test latency
tailscale ping <host-ip>

# Force direct connection
tailscale up --accept-routes

# Optimize DERP selection
# Tailscale Admin → Advanced → DERP map settings
```

### Tailscale ACLs Blocking Access

**Symptoms**: Connection timeout despite Tailscale being connected

**Solutions**:

1. **Check ACLs in Tailscale Admin Console**:
   - Go to https://login.tailscale.com/admin/acls
   - Verify your user/device has access to ports 6274 and 6277

2. **Test with Permissive ACL First**:
   ```json
   {
     "acls": [
       {
         "action": "accept",
         "src": ["*"],
         "dst": ["*:*"]
       }
     ]
   }
   ```
   (Don't use this in production!)

3. **Use Tailscale Debug Tool**:
   ```bash
   tailscale debug --acl <source-ip> <dest-ip>:<port>
   ```

---

## MCP Client Issues

### Claude Desktop Can't Connect

**Symptoms**: Claude shows no tools or connection error

**Solutions**:

1. **Verify Configuration**:
   - Check `~/.mcp/config.json` (or similar)
   - Ensure URL is correct: `http://<tailscale-ip>:6277`

2. **Test Connection Manually**:
   ```bash
   curl http://<tailscale-ip>:6277
   ```

3. **Check Client Logs**:
   - Claude Desktop logs location varies by OS
   - Look for connection errors

4. **Restart Claude Desktop**:
   - Close completely and reopen
   - Some clients cache configuration

### Cursor/VS Code MCP Not Working

**Symptoms**: MCP features not available in editor

**Solutions**:

1. **Verify Extension Installed**:
   - Check for MCP extension
   - Ensure it's enabled

2. **Check Settings**:
   ```json
   {
     "mcp.servers": {
       "gateway": {
         "url": "http://<tailscale-ip>:6277"
       }
     }
   }
   ```

3. **Check Extension Logs**:
   - View → Output → Select MCP extension
   - Look for connection errors

4. **Restart Extension Host**:
   - Command Palette → Reload Window

### Tools Not Appearing in Client

**Symptoms**: Gateway connects but no tools available

**Solutions**:

```bash
# Verify markitdown is running
docker compose ps markitdown

# Check gateway discovery
docker compose logs mcp-gateway | grep -i discover

# Manually restart markitdown
docker compose restart markitdown

# Wait for gateway to rediscover (may take 30-60 seconds)
```

---

## Performance Issues

### High Memory Usage

**Symptoms**: Containers using excessive memory

**Solutions**:

```bash
# Check resource usage
docker stats

# Add memory limits to compose.yaml
services:
  mcp-gateway:
    mem_limit: 512m
    mem_reservation: 256m

# Restart with limits
docker compose up -d
```

### High CPU Usage

**Symptoms**: High CPU utilization from containers

**Solutions**:

```bash
# Identify which container
docker stats

# Check if processing large files
docker compose logs markitdown | tail -n 50

# Add CPU limits
services:
  mcp-gateway:
    cpus: '0.5'

# Consider upgrading host resources
```

### Slow File Conversions

**Symptoms**: Markitdown tool takes long time

**Possible Causes**:
1. Large files
2. Complex documents
3. Limited resources

**Solutions**:

- Reduce file size before conversion
- Increase container resources
- Convert files in smaller batches
- Check disk I/O performance

---

## Diagnostic Commands

### Collect All Relevant Info

```bash
#!/bin/bash
echo "=== System Info ==="
uname -a
docker version
docker compose version

echo "\n=== Container Status ==="
docker compose ps

echo "\n=== Container Logs ==="
docker compose logs --tail=50

echo "\n=== Network Status ==="
docker network ls
docker network inspect hl_dockermcpgateway_mcp

echo "\n=== Tailscale Status ==="
tailscale status
tailscale ip

echo "\n=== Listening Ports ==="
sudo netstat -tulpn | grep -E '6277|6274'

echo "\n=== Firewall Status ==="
sudo ufw status
sudo iptables -L -n

echo "\n=== Resource Usage ==="
docker stats --no-stream
```

Save as `diagnose.sh` and run: `bash diagnose.sh > diagnostic-output.txt`

---

## Getting Help

If you can't resolve your issue:

1. **Check Existing Issues**: https://github.com/YOUR-USERNAME/HL_DockerMCPGateway/issues
2. **Collect Diagnostic Info**: Run the diagnostic script above
3. **Open New Issue**: Include diagnostic output and describe your problem
4. **Community Help**: Check Docker MCP Gateway and Komodo communities

---

## Additional Resources

- [Komodo Setup Guide](KOMODO_SETUP.md)
- [Tailscale Setup Guide](TAILSCALE_SETUP.md)
- [Docker MCP Gateway Docs](https://github.com/docker/mcp-gateway)
- [Markitdown MCP Server](https://github.com/mcp/markitdown)
