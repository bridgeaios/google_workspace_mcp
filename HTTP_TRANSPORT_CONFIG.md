# Google Workspace MCP - HTTP Transport Configuration

## Configuration Summary

The HTTP transport for Google Workspace MCP server has been configured with the following settings:

### Transport Mode
- **Type**: `streamable-http`
- **Host**: `0.0.0.0` (all interfaces)
- **Port**: `8000` (configurable via `PORT` or `WORKSPACE_MCP_PORT`)
- **Base URI**: `http://localhost` (configurable via `WORKSPACE_MCP_BASE_URI`)

### Server Startup Commands

#### Docker Compose (Recommended)
```bash
# Start with HTTP transport
docker-compose up -d

# View logs
docker-compose logs -f gws_mcp

# Stop server
docker-compose down
```

#### Direct Python Execution
```bash
# Using uv (recommended)
uv run python main.py --transport streamable-http

# With custom host/port
uv run python main.py --transport streamable-http --host 0.0.0.0 --port 8000

# With specific tools
uv run python main.py --transport streamable-http --tools gmail drive calendar

# With tool tier
uv run python main.py --transport streamable-http --tool-tier core
```

#### Docker Run
```bash
# Build image
docker build -t google-workspace-mcp .

# Run with HTTP transport
docker run -p 8000:8000 \
  -e GOOGLE_OAUTH_CLIENT_ID="your-client-id" \
  -e GOOGLE_OAUTH_CLIENT_SECRET="your-client-secret" \
  -e MCP_ENABLE_OAUTH21="true" \
  google-workspace-mcp
```

### Required Environment Variables

| Variable | Required | Description |
|----------|----------|-------------|
| `GOOGLE_OAUTH_CLIENT_ID` | Yes | Google OAuth 2.0 Client ID |
| `GOOGLE_OAUTH_CLIENT_SECRET` | Yes | Google OAuth 2.0 Client Secret |
| `MCP_ENABLE_OAUTH21` | Yes | Enable OAuth 2.1 (`true` for HTTP transport) |
| `WORKSPACE_MCP_PORT` | No | Server port (default: 8000) |
| `WORKSPACE_MCP_HOST` | No | Bind host (default: 0.0.0.0) |
| `WORKSPACE_MCP_BASE_URI` | No | Base URI (default: http://localhost) |
| `WORKSPACE_EXTERNAL_URL` | No | External URL for reverse proxy |

### OAuth 2.1 Configuration (Required for HTTP)

HTTP transport requires OAuth 2.1 to be enabled for secure multi-user authentication:

```bash
# Required
cp .env.oauth21 .env
# Edit .env and add your Google OAuth credentials
```

Key OAuth 2.1 settings:
- `MCP_ENABLE_OAUTH21=true` - Enable OAuth 2.1 protocol
- `WORKSPACE_MCP_STATELESS_MODE=true` - Stateless operation (recommended for containers)
- `WORKSPACE_MCP_OAUTH_PROXY_STORAGE_BACKEND=memory` - In-memory storage (options: memory, disk, valkey)

### Network Configuration

#### Port Exposure
- **Internal**: Port 8000 (or `$PORT`/`$WORKSPACE_MCP_PORT`)
- **External**: Mapped via Docker (`8000:8000`)
- **Health Check**: `GET http://localhost:8000/health`

#### Firewall Rules
Ensure the following ports are open:
```bash
# Default HTTP port
8000/tcp

# Custom port (if WORKSPACE_MCP_PORT is set)
$WORKSPACE_MCP_PORT/tcp
```

#### Reverse Proxy (Optional)
For production with reverse proxy:
```nginx
server {
    listen 443 ssl;
    server_name your-domain.com;

    location / {
        proxy_pass http://localhost:8000;
        proxy_http_version 1.1;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

Set `WORKSPACE_EXTERNAL_URL=https://your-domain.com` when using reverse proxy.

### CORS Configuration

The server uses FastMCP's built-in CORS support. CORS is automatically configured for:
- All origins in development mode (`OAUTHLIB_INSECURE_TRANSPORT=true`)
- OAuth 2.1 endpoints at `/.well-known/oauth-authorization-server`
- MCP protocol endpoints at `/mcp`

### Health Check Endpoints

| Endpoint | Method | Description |
|----------|--------|-------------|
| `/health` | GET | Server health status |
| `/` | GET | Health check (alias) |
| `/oauth2callback` | GET | OAuth callback handler |

### MCP Protocol Endpoints

| Endpoint | Description |
|----------|-------------|
| `/mcp` | Main MCP protocol endpoint |
| `/.well-known/oauth-authorization-server` | OAuth 2.1 metadata |
| `/.well-known/oauth-protected-resource` | Protected resource metadata |

### Transport Validation Checklist

Before starting the server, verify:

- [ ] `.env` file exists with OAuth credentials
- [ ] `MCP_ENABLE_OAUTH21=true` is set
- [ ] Port 8000 is available (or custom port configured)
- [ ] Docker daemon is running (if using Docker)
- [ ] Google OAuth credentials are valid
- [ ] Redirect URI is configured in Google Cloud Console

### Testing HTTP Transport

```bash
# Test health endpoint
curl http://localhost:8000/health

# Expected response:
# {"status": "healthy", "service": "workspace-mcp", "version": "1.13.1", "transport": "streamable-http"}

# Test OAuth metadata
curl http://localhost:8000/.well-known/oauth-authorization-server
```

### Troubleshooting

#### Port Already in Use
```bash
# Check what's using port 8000
lsof -i :8000

# Use different port
WORKSPACE_MCP_PORT=8080 uv run python main.py --transport streamable-http
```

#### OAuth 2.1 Not Enabled
Error: "OAuth 2.1 is required for HTTP transport"
- Ensure `MCP_ENABLE_OAUTH21=true` in `.env`

#### CORS Issues
- Verify `OAUTHLIB_INSECURE_TRANSPORT=true` for HTTP development
- Use HTTPS in production

### Next Steps for Server Testing

1. **Verify Configuration**:
   ```bash
   cat .env | grep -E "(MCP_ENABLE_OAUTH21|GOOGLE_OAUTH_CLIENT)"
   ```

2. **Start Server**:
   ```bash
   docker-compose up -d
   ```

3. **Check Health**:
   ```bash
   curl http://localhost:8000/health
   ```

4. **Verify OAuth Endpoints**:
   ```bash
   curl http://localhost:8000/.well-known/oauth-authorization-server
   ```

5. **Review Logs**:
   ```bash
   docker-compose logs -f gws_mcp
   ```

### Security Notes

- Always use HTTPS in production
- Set strong `FASTMCP_SERVER_AUTH_GOOGLE_JWT_SIGNING_KEY` for production
- Use Valkey/Redis storage backend for distributed deployments
- Enable stateless mode for containerized deployments
- Never commit `.env` files with real credentials
