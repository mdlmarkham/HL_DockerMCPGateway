# Tailscale Configuration

This directory contains configuration files for Tailscale integration.

## Files

### serve-config.json

This file configures **Tailscale Serve** to expose the MCP Gateway over HTTPS within your tailnet.

```json
{
  "TCP": {
    "443": {
      "HTTPS": true
    }
  },
  "Web": {
    "mcp-gateway.${TS_CERT_DOMAIN}:443": {
      "Handlers": {
        "/": {
          "Proxy": "http://127.0.0.1:6277"
        },
        "/inspector": {
          "Proxy": "http://127.0.0.1:6274"
        }
      }
    }
  }
}
```

## Configuration Explained

### TCP Section
- **Port 443**: Configured for HTTPS traffic
- Tailscale automatically provisions TLS certificates

### Web Section
- **Hostname**: Uses your tailnet domain with MagicDNS
- **Root Path (/)**: Proxies to MCP Gateway on port 6277
- **Inspector Path (/inspector)**: Proxies to inspector UI on port 6274

## Environment Variables

The configuration uses the following environment variable:
- `TS_CERT_DOMAIN`: Your Tailscale tailnet domain (e.g., tail1234.ts.net)

This variable is automatically substituted by Tailscale when the container starts.

## Alternative: Dynamic Configuration

Instead of using this config file, you can configure Tailscale Serve dynamically:

```bash
# Enter the Tailscale container
docker exec -it tailscale-mcp-gateway sh

# Configure serve manually
tailscale serve https:443 / http://127.0.0.1:6277
tailscale serve https:443 /inspector http://127.0.0.1:6274

# Check configuration
tailscale serve status

# The configuration persists in Tailscale state
```

## Customization

### Adding More Paths

To expose additional services, add more handlers:

```json
{
  "Web": {
    "mcp-gateway.${TS_CERT_DOMAIN}:443": {
      "Handlers": {
        "/": {
          "Proxy": "http://127.0.0.1:6277"
        },
        "/inspector": {
          "Proxy": "http://127.0.0.1:6274"
        },
        "/api": {
          "Proxy": "http://127.0.0.1:8080"
        },
        "/docs": {
          "Path": "/var/www/docs"
        }
      }
    }
  }
}
```

### Multiple Hostnames

To expose services on different hostnames:

```json
{
  "Web": {
    "mcp-gateway.${TS_CERT_DOMAIN}:443": {
      "Handlers": {
        "/": {
          "Proxy": "http://127.0.0.1:6277"
        }
      }
    },
    "mcp-inspector.${TS_CERT_DOMAIN}:443": {
      "Handlers": {
        "/": {
          "Proxy": "http://127.0.0.1:6274"
        }
      }
    }
  }
}
```

### Custom Ports

To use a different port (e.g., 8443):

```json
{
  "TCP": {
    "8443": {
      "HTTPS": true
    }
  },
  "Web": {
    "mcp-gateway.${TS_CERT_DOMAIN}:8443": {
      "Handlers": {
        "/": {
          "Proxy": "http://127.0.0.1:6277"
        }
      }
    }
  }
}
```

Then access via: `https://mcp-gateway.tail1234.ts.net:8443`

### HTTP (Non-TLS)

For internal testing without TLS:

```json
{
  "TCP": {
    "80": {
      "HTTP": true
    }
  },
  "Web": {
    "mcp-gateway.${TS_CERT_DOMAIN}:80": {
      "Handlers": {
        "/": {
          "Proxy": "http://127.0.0.1:6277"
        }
      }
    }
  }
}
```

⚠️ **Not recommended**: Always use HTTPS (443) for production.

## Serving Static Files

You can serve static files or directories:

```json
{
  "Web": {
    "mcp-gateway.${TS_CERT_DOMAIN}:443": {
      "Handlers": {
        "/": {
          "Proxy": "http://127.0.0.1:6277"
        },
        "/static": {
          "Path": "/usr/share/nginx/html"
        },
        "/file.txt": {
          "Path": "/data/file.txt"
        }
      }
    }
  }
}
```

## TCP Forwarding (Layer 4)

For raw TCP forwarding without HTTP:

```json
{
  "TCP": {
    "5432": {
      "TCP": "127.0.0.1:5432"
    }
  }
}
```

This is useful for databases, SSH, or other TCP services.

## Validation

To validate your configuration:

```bash
# Check JSON syntax
cat serve-config.json | jq .

# Test in container
docker exec tailscale-mcp-gateway sh -c 'cat /config/serve.json | jq .'
```

## Troubleshooting

### Config Not Applied

1. **Check file is mounted**:
   ```bash
   docker exec tailscale-mcp-gateway ls -la /config/
   ```

2. **Verify environment variable**:
   ```bash
   docker exec tailscale-mcp-gateway env | grep TS_
   ```

3. **Check Tailscale logs**:
   ```bash
   docker compose logs tailscale-gateway
   ```

### Changes Not Taking Effect

After modifying `serve-config.json`:

```bash
# Restart the Tailscale container
docker compose restart tailscale-gateway

# Or restart the entire stack
docker compose down
docker compose up -d
```

### Test Configuration

```bash
# From another device on your tailnet
curl -v https://mcp-gateway.YOUR-TAILNET.ts.net

# Check serve status
docker exec tailscale-mcp-gateway tailscale serve status
```

## Security Notes

1. ✅ This configuration only exposes services within your tailnet
2. ✅ TLS certificates are automatically managed by Tailscale
3. ✅ Access is controlled via Tailscale ACLs
4. ⚠️ Never use Funnel unless you need public internet access
5. ⚠️ Always use HTTPS (port 443) for production services

## References

- [Tailscale Serve Documentation](https://tailscale.com/kb/1312/serve)
- [Tailscale Services](https://tailscale.com/kb/1100/services)
- [Serve Configuration Examples](https://tailscale.com/kb/1313/serve-examples)
- [Tailscale in Docker](https://tailscale.com/kb/1282/docker)

## Support

For issues with Tailscale configuration:
- Check the [Troubleshooting Guide](../docs/TROUBLESHOOTING.md)
- Review [Tailscale Services Guide](../docs/TAILSCALE_SERVICES.md)
- Open an issue on GitHub
