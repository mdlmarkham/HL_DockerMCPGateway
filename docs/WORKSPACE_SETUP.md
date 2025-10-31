# Workspace Example

This directory is meant to be mounted into the MCP containers for file access.

## Structure

```
workspace/
├── documents/          # Your documents for conversion
│   ├── reports/
│   ├── papers/
│   └── notes/
├── data/              # Data files
└── output/            # Processed outputs (if needed)
```

## Usage

The workspace is mounted **read-only** by default for security. Files placed here can be accessed by MCP tools like Markitdown for conversion.

### Example Files You Might Place Here

- PDF documents
- Word documents (DOCX)
- HTML files
- PowerPoint presentations
- Excel spreadsheets
- Images
- Text files

### Configuration

Update the volume mount in `compose.yaml` to point to your desired workspace location:

```yaml
volumes:
  - /srv/mcp/workspace:/workspace:ro  # Change to your path
```

## Security Note

⚠️ **Important**: This directory is accessible to all MCP containers. Do not place sensitive or confidential files here unless you understand the security implications.

Consider:
- Using a dedicated directory for MCP operations
- Not storing credentials or API keys
- Being mindful of file permissions on the host
- Regularly cleaning up processed files

## Example Directory Setup

```bash
# Create the workspace structure
mkdir -p /srv/mcp/workspace/{documents,data,output}

# Set appropriate permissions
chmod 755 /srv/mcp/workspace
chmod 755 /srv/mcp/workspace/{documents,data,output}

# Optional: Create a dedicated user for MCP operations
sudo useradd -r -s /bin/false mcpuser
sudo chown -R mcpuser:mcpuser /srv/mcp/workspace
```
