# Railway Deployment Guide for Magg

This guide explains how to deploy Magg (MCP Aggregator) on Railway using Docker.

## Prerequisites

- Railway account (sign up at [railway.app](https://railway.app))
- Railway CLI installed (optional but recommended)
- Git repository pushed to GitHub

## Quick Deploy

### Option 1: Deploy from GitHub (Recommended)

1. **Connect Repository to Railway:**
   ```bash
   # If you have Railway CLI installed
   railway login
   railway link
   railway up
   ```

   Or use the Railway dashboard:
   - Go to [railway.app](https://railway.app)
   - Click "New Project"
   - Select "Deploy from GitHub repo"
   - Choose your `magg` repository

2. **Railway will automatically:**
   - Detect the `railway.toml` configuration
   - Build using the `Dockerfile` with `railway` target
   - Deploy to a public URL with proper environment variables

### Option 2: One-Click Deploy

[![Deploy on Railway](https://railway.app/button.svg)](https://railway.app/template/your-template-id)

## Configuration

### Environment Variables

Railway will automatically set these variables from `railway.toml`:

- `MAGG_LOG_LEVEL=INFO`
- `MAGG_CONFIG_PATH=/home/magg/.magg/config.json`
- `MAGG_READ_ONLY=false`
- `MAGG_AUTO_RELOAD=true`
- `PORT=8000` (or Railway's assigned port)

### Optional Configuration

You can set additional environment variables in Railway dashboard:

#### Authentication (Optional)
```bash
MAGG_PRIVATE_KEY=your-rsa-private-key
MAGG_JWT=your-jwt-token
```

#### Advanced Settings
```bash
MAGG_SELF_PREFIX=magg
MAGG_PREFIX_SEP=_
MAGG_RELOAD_POLL_INTERVAL=5.0
```

## Dockerfile Options

### Default Deployment (Recommended)
Uses the existing `dockerfile` with the `pro` (production) stage:
- Minimal image size
- WARNING log level
- Health checks included
- Optimized for production

### Railway-Specific Dockerfile
If you prefer, you can use `dockerfile.railway`:
```toml
# In railway.toml
[build]
dockerfilePath = "dockerfile.railway"
```

This provides:
- INFO log level (more verbose for debugging)
- Railway-specific optimizations
- Dynamic port binding

## Deployment Process

1. **Build Stage:**
   - Railway builds the Docker image using multi-stage build
   - Dependencies are cached for faster subsequent builds
   - Only production dependencies are included

2. **Deploy Stage:**
   - Container starts with `tini` init system
   - Magg serves HTTP on Railway's assigned port
   - Health checks ensure service availability

3. **Access:**
   - Railway provides a public URL (e.g., `https://your-app.railway.app`)
   - MCP endpoint available at `/mcp`

## Usage After Deployment

### Connecting MCP Clients

Your deployed Magg instance will be available at:
```
https://your-app.railway.app/mcp
```

### Using with MaggClient

```python
from magg.client import MaggClient

async def main():
    async with MaggClient("https://your-app.railway.app/mcp") as client:
        # List available tools
        tools = await client.list_tools()
        print(f"Available tools: {len(tools)}")
        
        # Add a new MCP server
        await client.call_tool("magg_add_server", {
            "name": "calculator",
            "source": "npx @modelcontextprotocol/server-calculator",
            "prefix": "calc"
        })
```

### Authentication (If Enabled)

If you've configured authentication:

```python
import os
from magg.client import MaggClient

# Set JWT token as environment variable
os.environ["MAGG_JWT"] = "your-jwt-token"

async with MaggClient("https://your-app.railway.app/mcp") as client:
    # Client automatically uses MAGG_JWT for authentication
    tools = await client.list_tools()
```

## Monitoring and Logs

### Railway Dashboard
- View deployment logs in Railway dashboard
- Monitor resource usage and performance
- Set up alerts for downtime

### Health Checks
Railway automatically monitors the health endpoint:
```bash
curl https://your-app.railway.app/health
```

### Application Logs
View logs using Railway CLI:
```bash
railway logs
```

## Troubleshooting

### Common Issues

1. **Build Failures:**
   - Check that all required files are included (not in `.dockerignore`)
   - Verify `uv.lock` and `pyproject.toml` are present

2. **Port Binding Issues:**
   - Ensure Dockerfile exposes the correct port
   - Railway automatically sets `PORT` environment variable

3. **Configuration Issues:**
   - Check environment variables in Railway dashboard
   - Verify `MAGG_CONFIG_PATH` is writable

### Debug Mode

For debugging, you can temporarily set:
```bash
MAGG_LOG_LEVEL=DEBUG
```

This will provide more detailed logs for troubleshooting.

## Scaling and Performance

### Resource Limits
Railway provides:
- 512MB RAM (Hobby plan)
- 1GB RAM+ (Pro plan)
- Auto-scaling based on demand

### Optimization Tips
1. Use the production `dockerfile` stage for minimal resource usage
2. Enable `MAGG_AUTO_RELOAD=false` if configuration changes are rare
3. Monitor memory usage with multiple MCP servers

## Security Considerations

1. **Authentication:** Enable JWT authentication for production use
2. **Read-Only Mode:** Set `MAGG_READ_ONLY=true` to prevent configuration changes
3. **Environment Variables:** Use Railway's environment variable management for secrets

## Support

- **Magg Issues:** [GitHub Issues](https://github.com/klogins-hash/magg/issues)
- **Railway Support:** [Railway Documentation](https://docs.railway.app/)
- **MCP Protocol:** [Model Context Protocol](https://modelcontextprotocol.io/)

## Next Steps

After deployment:
1. Test the MCP endpoint with a client
2. Add your first MCP servers using `magg_add_server`
3. Configure authentication if needed
4. Set up monitoring and alerts
5. Integrate with your AI applications

---

**Note:** This deployment guide assumes you're using the forked repository at `klogins-hash/magg`. Update URLs and references as needed for your specific setup.
