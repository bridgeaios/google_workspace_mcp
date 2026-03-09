# Google Workspace MCP Server - Complete Installation & Configuration Guide

**Version:** 1.13.1  
**Last Updated:** March 2026  
**Compatibility:** Python 3.10+, Docker, Kubernetes

---

## Table of Contents

1. [Prerequisites and System Requirements](#1-prerequisites-and-system-requirements)
2. [Complete Installation Guide](#2-complete-installation-guide)
3. [OAuth 2.1 Authentication Documentation](#3-oauth-21-authentication-documentation)
4. [HTTP Transport Configuration](#4-http-transport-configuration)
5. [MCP Server Integration Guide](#5-mcp-server-integration-guide)
6. [Production Deployment Documentation](#6-production-deployment-documentation)
7. [Testing and Verification](#7-testing-and-verification)
8. [Troubleshooting Guide](#8-troubleshooting-guide)

---

## 1. Prerequisites and System Requirements

### 1.1 System Requirements

| Component | Minimum | Recommended |
|-----------|---------|-------------|
| **CPU** | 1 core | 2+ cores |
| **RAM** | 512 MB | 1 GB |
| **Storage** | 1 GB | 5 GB |
| **Network** | Internet access for OAuth | Stable connection |
| **OS** | Linux, macOS, Windows | Linux (Ubuntu 22.04+) |

### 1.2 Software Prerequisites

#### Required Software

| Software | Version | Purpose |
|----------|---------|---------|
| Python | 3.10+ | Runtime environment |
| uv | Latest | Package manager |
| Docker | 20.10+ | Container runtime (optional) |
| Docker Compose | 2.0+ | Container orchestration (optional) |

### 1.3 Verify Prerequisites

```bash
# Check Python version
python --version  # Should be 3.10 or higher

# Check uv installation
uv --version

# Check Docker (if using containers)
docker --version
docker-compose --version
```

### 1.4 Network Requirements

| Port | Protocol | Purpose | Direction |
|------|----------|---------|-----------|
| 8000 | TCP | MCP HTTP Server | Inbound |
| 443 | TCP | HTTPS OAuth callbacks | Outbound |
| 80 | TCP | HTTP OAuth callbacks | Outbound |

### 1.5 Required Google Cloud APIs

The following Google APIs must be enabled in your Google Cloud Console:

- Google Calendar API
- Google Drive API
- Gmail API
- Google Docs API
- Google Sheets API
- Google Slides API
- Google Forms API
- Google Tasks API
- Google Chat API
- Google People API
- Google Custom Search API
- Google Apps Script API

---

## 2. Complete Installation Guide

### 2.1 Installation Methods Overview

| Method | Use Case | Complexity | Persistence |
|--------|----------|------------|-------------|
| **uvx (Instant)** | Quick testing, single-user | Low | None |
| **uv + Clone** | Development, customization | Medium | Local files |
| **Docker** | Production, containers | Low | Volumes |
| **Docker Compose** | Full stack deployment | Medium | Named volumes |
| **Kubernetes** | Enterprise, scaling | High | PVCs |

### 2.2 Method 1: Instant Installation with uvx (Recommended for Testing)

```bash
# Install uv (if not already installed)
curl -LsSf https://astral.sh/uv/install.sh | sh

# Set environment variables
export GOOGLE_OAUTH_CLIENT_ID="your-client-id"
export GOOGLE_OAUTH_CLIENT_SECRET="your-client-secret"
export OAUTHLIB_INSECURE_TRANSPORT=1  # Development only

# Run the server instantly
uvx workspace-mcp --tool-tier core
```

**Pros:**
- No local installation required
- Automatic dependency resolution
- Always up-to-date

**Cons:**
- Cannot modify source code
- No persistent storage
- Requires internet connection

### 2.3 Method 2: Local Development Installation

#### Step 1: Clone the Repository

```bash
# Clone the repository
git clone https://github.com/taylorwilsdon/google_workspace_mcp.git
cd google_workspace_mcp

# Verify Python version
python --version  # Must be 3.10+
```

#### Step 2: Install uv Package Manager

```bash
# Install uv
curl -LsSf https://astral.sh/uv/install.sh | sh

# Or on Windows
powershell -c "irm https://astral.sh/uv/install.ps1 | iex"

# Verify installation
uv --version
```

#### Step 3: Install Dependencies

```bash
# Sync all dependencies (production + dev)
uv sync --frozen --no-dev

# For development with all tools
uv sync --group dev

# For testing only
uv sync --group test

# With Valkey support (optional)
uv sync --group valkey
```

**Dependency Installation Verification:**

```bash
# Verify Python packages
uv pip list | grep -E "(fastmcp|google-api|fastapi)"

# Expected output should include:
# fastapi>=0.115.12
# fastmcp>=3.0.2
# google-api-python-client>=2.168.0
# google-auth-oauthlib>=1.2.2
```

#### Step 4: Environment Configuration

```bash
# Copy the OAuth 2.1 template
cp .env.oauth21 .env

# Edit the .env file with your credentials
nano .env  # or vim, code, etc.
```

**Required `.env` variables:**

```bash
# Google OAuth Credentials (from Google Cloud Console)
GOOGLE_OAUTH_CLIENT_ID="your-client-id.apps.googleusercontent.com"
GOOGLE_OAUTH_CLIENT_SECRET="your-client-secret"

# Development settings
OAUTH2_ALLOW_INSECURE_TRANSPORT=true
OAUTH2_ENABLE_DEBUG=false

# Enable OAuth 2.1 for HTTP transport
MCP_ENABLE_OAUTH21=true

# Server configuration
WORKSPACE_MCP_PORT=8000
WORKSPACE_MCP_HOST=0.0.0.0
WORKSPACE_MCP_BASE_URI=http://localhost

# Storage backend
WORKSPACE_MCP_OAUTH_PROXY_STORAGE_BACKEND=memory
```

### 2.4 Method 3: Docker Installation

#### Step 1: Build the Docker Image

```bash
# Build the image
docker build -t google-workspace-mcp:latest .

# Verify the image was created
docker images | grep google-workspace-mcp
```

#### Step 2: Run with Docker

```bash
# Run with environment variables
docker run -d \
  --name gws-mcp \
  -p 8000:8000 \
  -e GOOGLE_OAUTH_CLIENT_ID="your-client-id" \
  -e GOOGLE_OAUTH_CLIENT_SECRET="your-client-secret" \
  -e MCP_ENABLE_OAUTH21="true" \
  -e WORKSPACE_MCP_OAUTH_PROXY_STORAGE_BACKEND="memory" \
  google-workspace-mcp:latest

# Check container status
docker ps | grep gws-mcp
```

### 2.5 Method 4: Docker Compose Installation (Recommended for Production)

#### Step 1: Configure Environment

```bash
# Copy and edit environment file
cp .env.oauth21 .env
# Edit .env with your credentials
```

#### Step 2: Start Services

```bash
# Start in detached mode
docker-compose up -d

# View logs
docker-compose logs -f gws_mcp

# Check status
docker-compose ps
```

**Docker Compose Configuration:**

```yaml
services:
  gws_mcp:
    build: .
    container_name: gws_mcp
    ports:
      - "8001:8000"  # Host:Container
    environment:
      - GOOGLE_MCP_CREDENTIALS_DIR=/app/store_creds
    volumes:
      - ./client_secret.json:/app/client_secret.json:ro
      - store_creds:/app/store_creds:rw
    env_file:
      - .env

volumes:
  store_creds:
```

#### Step 3: Manage Services

```bash
# Stop services
docker-compose down

# Restart with rebuild
docker-compose up -d --build

# View specific service logs
docker-compose logs -f gws_mcp

# Scale (if applicable)
docker-compose up -d --scale gws_mcp=3
```

### 2.6 Method 5: Kubernetes Installation

#### Prerequisites

- Kubernetes 1.19+
- Helm 3.2.0+
- kubectl configured

#### Step 1: Install with Helm

```bash
# Install the Helm chart
helm install workspace-mcp ./helm-chart/workspace-mcp \
  --set secrets.googleOAuth.clientId="your-client-id" \
  --set secrets.googleOAuth.clientSecret="your-client-secret" \
  --set env.MCP_ENABLE_OAUTH21="true"

# Verify deployment
kubectl get pods -l app.kubernetes.io/name=workspace-mcp
```

#### Step 2: Configure Ingress (Optional)

```bash
# Enable ingress
helm upgrade workspace-mcp ./helm-chart/workspace-mcp \
  --set ingress.enabled=true \
  --set ingress.hosts[0].host="workspace-mcp.yourdomain.com"
```

---

## 3. OAuth 2.1 Authentication Documentation

### 3.1 Overview

OAuth 2.1 is required for HTTP transport mode and provides:
- Multi-user authentication via bearer tokens
- PKCE (Proof Key for Code Exchange) support
- Enhanced security over OAuth 2.0
- Stateless session management

### 3.2 Google Cloud Console Setup

#### Step 1: Create a Google Cloud Project

1. Navigate to [Google Cloud Console](https://console.cloud.google.com/)
2. Click on project selector (top-left)
3. Click "New Project"
4. Enter project name (e.g., "workspace-mcp")
5. Click "Create"

#### Step 2: Enable Required APIs

```bash
# Quick API enable links (one-click each):
# Calendar: https://console.cloud.google.com/flows/enableapi?apiid=calendar-json.googleapis.com
# Drive:    https://console.cloud.google.com/flows/enableapi?apiid=drive.googleapis.com
# Gmail:    https://console.cloud.google.com/flows/enableapi?apiid=gmail.googleapis.com
# Docs:     https://console.cloud.google.com/flows/enableapi?apiid=docs.googleapis.com
# Sheets:   https://console.cloud.google.com/flows/enableapi?apiid=sheets.googleapis.com
# Slides:   https://console.cloud.google.com/flows/enableapi?apiid=slides.googleapis.com
# Forms:    https://console.cloud.google.com/flows/enableapi?apiid=forms.googleapis.com
# Tasks:    https://console.cloud.google.com/flows/enableapi?apiid=tasks.googleapis.com
# Chat:     https://console.cloud.google.com/flows/enableapi?apiid=chat.googleapis.com
# People:   https://console.cloud.google.com/flows/enableapi?apiid=people.googleapis.com
# Search:   https://console.cloud.google.com/flows/enableapi?apiid=customsearch.googleapis.com
# Apps Script: https://console.cloud.google.com/flows/enableapi?apiid=script.googleapis.com
```

#### Step 3: Create OAuth 2.0 Credentials

1. Navigate to **APIs & Services → Credentials**
2. Click **Create Credentials → OAuth client ID**
3. Select **Desktop Application** as the application type
4. Enter a name (e.g., "Workspace MCP Desktop Client")
5. Click **Create**
6. Download the JSON credentials file
7. Extract `client_id` and `client_secret`

**Why Desktop Application?**
- No redirect URI configuration needed
- Works with localhost callbacks
- Simplifies development setup

### 3.3 OAuth 2.1 Client Configuration

#### Environment Variables

```bash
# Required OAuth credentials
export GOOGLE_OAUTH_CLIENT_ID="your-client-id.apps.googleusercontent.com"
export GOOGLE_OAUTH_CLIENT_SECRET="your-client-secret"

# Enable OAuth 2.1
export MCP_ENABLE_OAUTH21=true

# Optional: External OAuth provider mode
export EXTERNAL_OAUTH21_PROVIDER=false

# Optional: Stateless mode (container-friendly)
export WORKSPACE_MCP_STATELESS_MODE=true

# Development only: Allow HTTP
export OAUTH2_ALLOW_INSECURE_TRANSPORT=true
```

### 3.4 Bearer Token Authentication Flow

#### Authentication Flow Diagram

```
┌─────────┐                    ┌──────────────┐                    ┌──────────┐
│  Client │                    │  MCP Server  │                    │  Google  │
└────┬────┘                    └──────┬───────┘                    └────┬─────┘
     │                                │                                 │
     │ 1. Request OAuth URL           │                                 │
     │───────────────────────────────>│                                 │
     │                                │                                 │
     │ 2. Return authorization URL    │                                 │
     │<───────────────────────────────│                                 │
     │                                │                                 │
     │ 3. Open browser, authenticate  │                                 │
     │─────────────────────────────────────────────────────────────────>│
     │                                │                                 │
     │ 4. Redirect with auth code     │                                 │
     │<─────────────────────────────────────────────────────────────────│
     │                                │                                 │
     │ 5. Send code to callback       │                                 │
     │───────────────────────────────>│                                 │
     │                                │ 6. Exchange code for tokens     │
     │                                │────────────────────────────────>│
     │                                │                                 │
     │                                │ 7. Return tokens                │
     │                                │<────────────────────────────────│
     │                                │                                 │
     │ 8. Return success + session    │                                 │
     │<───────────────────────────────│                                 │
     │                                │                                 │
     │ 9. API calls with Bearer token │                                 │
     │───────────────────────────────>│                                 │
```

### 3.5 PKCE Implementation Details

PKCE (RFC 7636) is mandatory in OAuth 2.1 and prevents authorization code interception attacks.

#### How PKCE Works

1. **Code Verifier Generation:** Client generates a cryptographically random string (43-128 characters)
2. **Code Challenge Creation:** SHA256 hash of the code verifier, base64url-encoded
3. **Authorization Request:** Client sends code challenge with authorization request
4. **Token Exchange:** Client sends code verifier when exchanging authorization code for tokens
5. **Server Verification:** Server verifies code verifier matches the challenge

#### Configuration

```bash
# PKCE is automatically enabled when MCP_ENABLE_OAUTH21=true
# No additional configuration needed

# Supported code challenge methods (OAuth 2.1)
# - S256 (required)
# - plain (deprecated, not used in 2.1)
```

### 3.6 Storage Backends

#### Memory Storage (Default for Linux)

```bash
# Fast, no persistence, data lost on restart
export WORKSPACE_MCP_OAUTH_PROXY_STORAGE_BACKEND=memory
```

**Best for:** Development, testing, stateless deployments

#### Disk Storage (Default for Mac/Windows)

```bash
# Persists across restarts, single-server only
export WORKSPACE_MCP_OAUTH_PROXY_STORAGE_BACKEND=disk
export WORKSPACE_MCP_OAUTH_PROXY_DISK_DIRECTORY=~/.fastmcp/oauth-proxy
```

**Best for:** Single-server production, persistent caching

#### Valkey/Redis Storage

```bash
# Distributed, multi-server support
export WORKSPACE_MCP_OAUTH_PROXY_STORAGE_BACKEND=valkey
export WORKSPACE_MCP_OAUTH_PROXY_VALKEY_HOST=redis.example.com
export WORKSPACE_MCP_OAUTH_PROXY_VALKEY_PORT=6379
export WORKSPACE_MCP_OAUTH_PROXY_VALKEY_USE_TLS=false
export WORKSPACE_MCP_OAUTH_PROXY_VALKEY_DB=0
```

**Best for:** Production, multi-server deployments, cloud native

---

## 4. HTTP Transport Configuration

### 4.1 Streamable HTTP Transport Setup

#### Basic Configuration

```bash
# Start with HTTP transport
uv run main.py --transport streamable-http

# With custom host and port
uv run main.py --transport streamable-http --host 0.0.0.0 --port 8000

# With specific tools
uv run main.py --transport streamable-http --tools gmail drive calendar

# With tool tier
uv run main.py --transport streamable-http --tool-tier core
```

#### Environment Configuration

```bash
# Server settings
export WORKSPACE_MCP_PORT=8000
export WORKSPACE_MCP_HOST=0.0.0.0
export WORKSPACE_MCP_BASE_URI=http://localhost

# For reverse proxy
export WORKSPACE_EXTERNAL_URL=https://your-domain.com

# OAuth redirect
export GOOGLE_OAUTH_REDIRECT_URI=http://localhost:8000/oauth2callback
```

### 4.2 Port Configuration and Mapping

#### Default Ports

| Service | Default Port | Environment Variable |
|---------|--------------|---------------------|
| MCP HTTP Server | 8000 | `WORKSPACE_MCP_PORT` |
| OAuth Callback | 8000 (same) | N/A |
| Health Check | 8000 (same) | N/A |

#### Custom Port Configuration

```bash
# Use port 8080
export WORKSPACE_MCP_PORT=8080
uv run main.py --transport streamable-http

# Docker port mapping
# Map container port 8000 to host port 8080
ports:
  - "8080:8000"
```

### 4.3 CORS Settings and Security

#### Default CORS Configuration

CORS is automatically configured by FastMCP for:
- All origins in development mode (`OAUTH2_ALLOW_INSECURE_TRANSPORT=true`)
- OAuth 2.1 discovery endpoints at `/.well-known/oauth-authorization-server`
- MCP protocol endpoints at `/mcp`

#### Custom CORS Origins

```bash
# Add custom allowed origins
export OAUTH_ALLOWED_ORIGINS="https://your-domain.com,https://app.your-domain.com"
```

#### Security Headers

The server automatically sets:
- `Cache-Control: no-store, must-revalidate` on OAuth metadata endpoints
- `ETag` headers for cache busting
- Proper CORS headers for cross-origin requests

### 4.4 Network Configuration for Production

#### Firewall Rules

```bash
# Allow MCP HTTP port
sudo ufw allow 8000/tcp

# For custom port
sudo ufw allow 8080/tcp

# Allow HTTPS (for OAuth callbacks)
sudo ufw allow 443/tcp
```

#### Network Security Checklist

- [ ] Block direct access to port 8000 from external networks (use reverse proxy)
- [ ] Enable HTTPS for OAuth callbacks in production
- [ ] Configure firewall rules
- [ ] Disable `OAUTH2_ALLOW_INSECURE_TRANSPORT` in production
- [ ] Use strong JWT signing keys

### 4.5 Reverse Proxy Setup

#### Nginx Configuration

```nginx
server {
    listen 443 ssl http2;
    server_name workspace-mcp.yourdomain.com;

    # SSL certificates
    ssl_certificate /path/to/cert.pem;
    ssl_certificate_key /path/to/key.pem;

    # Security headers
    add_header X-Frame-Options "SAMEORIGIN" always;
    add_header X-Content-Type-Options "nosniff" always;
    add_header X-XSS-Protection "1; mode=block" always;

    location / {
        proxy_pass http://localhost:8000;
        proxy_http_version 1.1;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        
        # WebSocket support (for streaming)
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
    }

    # OAuth endpoints
    location ~ ^/(oauth2|\.well-known)/ {
        proxy_pass http://localhost:8000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}
```

#### Environment Variables for Reverse Proxy

```bash
# Tell the server about the external URL
export WORKSPACE_EXTERNAL_URL=https://workspace-mcp.yourdomain.com

# Or just override the OAuth redirect URI
export GOOGLE_OAUTH_REDIRECT_URI=https://workspace-mcp.yourdomain.com/oauth2callback
```

---

## 5. MCP Server Integration Guide

### 5.1 Server Startup Procedures

#### Standard Startup

```bash
# Load environment variables
source .env

# Start with HTTP transport
uv run main.py --transport streamable-http

# Expected output:
# 🔧 Google Workspace MCP Server
# ===================================
# 📋 Server Information:
#    📦 Version: 1.13.1
#    🌐 Transport: streamable-http
#    🔗 URL: http://localhost:8000
#    🔐 OAuth Callback: http://localhost:8000/oauth2callback
#    👤 Mode: Multi-user
# ...
# ✅ Ready for MCP connections
```

#### Docker Startup

```bash
# Build and start
docker-compose up -d

# View logs
docker-compose logs -f gws_mcp

# Verify running
docker-compose ps
```

### 5.2 Health Check Verification

#### Health Endpoint

```bash
# Check server health
curl http://localhost:8000/health

# Expected response:
{
  "status": "healthy",
  "service": "workspace-mcp",
  "version": "1.13.1",
  "transport": "streamable-http"
}
```

#### Docker Health Check

```bash
# Check container health
docker ps | grep gws_mcp

# Expected: Status shows (healthy)
```

### 5.3 OAuth Endpoint Testing

#### Authorization Server Metadata

```bash
# Get OAuth 2.1 metadata
curl http://localhost:8000/.well-known/oauth-authorization-server

# Expected response includes:
# {
#   "issuer": "https://accounts.google.com",
#   "authorization_endpoint": "...",
#   "token_endpoint": "...",
#   "response_types_supported": ["code"],
#   "code_challenge_methods_supported": ["S256"]
# }
```

#### Protected Resource Metadata

```bash
# Get protected resource metadata
curl http://localhost:8000/.well-known/oauth-protected-resource
```

### 5.4 MCP Protocol Handshake

#### Connect with MCP Inspector

```bash
# Install MCP Inspector
npx @anthropic-ai/mcp-inspector@latest

# Connect to your server
npx @anthropic-ai/mcp-inspector@latest http://localhost:8000/mcp
```

#### Connect with Claude Code

```bash
# Start the server
uv run main.py --transport streamable-http

# In another terminal, add to Claude Code
claude mcp add --transport http workspace-mcp http://localhost:8000/mcp
```

#### Connect with VS Code

Add to VS Code settings (`settings.json`):

```json
{
  "mcp.servers": {
    "google-workspace": {
      "url": "http://localhost:8000/mcp",
      "type": "http"
    }
  }
}
```

### 5.5 Error Handling and Troubleshooting

#### Common Startup Errors

| Error | Cause | Solution |
|-------|-------|----------|
| `Port 8000 already in use` | Another service using port | Change `WORKSPACE_MCP_PORT` |
| `OAuth credentials not configured` | Missing env vars | Set `GOOGLE_OAUTH_CLIENT_ID` and `GOOGLE_OAUTH_CLIENT_SECRET` |
| `OAuth 2.1 required for HTTP transport` | Missing `MCP_ENABLE_OAUTH21` | Set `MCP_ENABLE_OAUTH21=true` |
| `Single-user mode incompatible with OAuth 2.1` | Mutually exclusive modes | Choose one: `--single-user` OR `MCP_ENABLE_OAUTH21=true` |

---

## 6. Production Deployment Documentation

### 6.1 Container Orchestration with Docker Compose

#### Production docker-compose.yml

```yaml
version: '3.8'

services:
  gws_mcp:
    build:
      context: .
      dockerfile: Dockerfile
    image: google-workspace-mcp:latest
    container_name: gws_mcp
    restart: unless-stopped
    ports:
      - "127.0.0.1:8000:8000"  # Bind to localhost only
    environment:
      - GOOGLE_OAUTH_CLIENT_ID=${GOOGLE_OAUTH_CLIENT_ID}
      - GOOGLE_OAUTH_CLIENT_SECRET=${GOOGLE_OAUTH_CLIENT_SECRET}
      - MCP_ENABLE_OAUTH21=true
      - WORKSPACE_MCP_STATELESS_MODE=true
      - WORKSPACE_MCP_OAUTH_PROXY_STORAGE_BACKEND=memory
      - WORKSPACE_EXTERNAL_URL=https://workspace-mcp.yourdomain.com
      # Disable insecure transport in production
      - OAUTH2_ALLOW_INSECURE_TRANSPORT=false
    env_file:
      - .env
    volumes:
      # Mount credentials read-only
      - ./client_secret.json:/app/client_secret.json:ro
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:8000/health"]
      interval: 30s
      timeout: 10s
      retries: 3
      start_period: 30s
    deploy:
      resources:
        limits:
          cpus: '1.0'
          memory: 512M
        reservations:
          cpus: '0.5'
          memory: 256M
    logging:
      driver: "json-file"
      options:
        max-size: "10m"
        max-file: "3"
    networks:
      - gws_mcp_network

networks:
  gws_mcp_network:
    driver: bridge
```

### 6.2 Resource Optimization

#### CPU and Memory Limits

| Deployment Type | CPU Limit | Memory Limit | Recommended |
|----------------|-----------|--------------|-------------|
| Development | 0.5 cores | 256 MB | No limits |
| Small Production | 1 core | 512 MB | ✅ Recommended |
| Medium Production | 2 cores | 1 GB | High traffic |
| Large Production | 4 cores | 2 GB | Enterprise |

#### Python Optimization

```bash
# Set Python optimizations
export PYTHONOPTIMIZE=1
export PYTHONDONTWRITEBYTECODE=1

# Use uv's faster execution
uv run --python-preference=only-system main.py
```

### 6.3 Monitoring

#### Health Check Monitoring

```bash
# Continuous health check
while true; do
  curl -s http://localhost:8000/health | jq .
  sleep 30
done
```

#### Log Monitoring

```bash
# View logs in real-time
docker-compose logs -f gws_mcp

# Search for errors
docker-compose logs gws_mcp | grep -i error

# View last 100 lines
docker-compose logs --tail=100 gws_mcp
```

#### Prometheus Metrics (Custom Implementation)

Add to your monitoring stack:

```yaml
# prometheus.yml
scrape_configs:
  - job_name: 'workspace-mcp'
    static_configs:
      - targets: ['gws_mcp:8000']
    metrics_path: '/metrics'
    scrape_interval: 30s
```

### 6.4 Security Hardening

#### Required Security Measures

1. **HTTPS Only**
   ```bash
   # Never use in production
   export OAUTH2_ALLOW_INSECURE_TRANSPORT=false
   ```

2. **Strong JWT Signing Key**
   ```bash
   # Generate a strong key
   export FASTMCP_SERVER_AUTH_GOOGLE_JWT_SIGNING_KEY=$(openssl rand -base64 32)
   ```

3. **Non-Root Container**
   ```dockerfile
   # Dockerfile already includes
   USER app
   ```

4. **Read-Only Filesystem**
   ```yaml
   # docker-compose.yml
   read_only: true
   tmpfs:
     - /tmp
   ```

5. **Resource Limits**
   ```yaml
   deploy:
     resources:
       limits:
         cpus: '1.0'
         memory: 512M
   ```

#### Security Checklist

- [ ] Disable insecure transport (`OAUTH2_ALLOW_INSECURE_TRANSPORT=false`)
- [ ] Use HTTPS for all OAuth callbacks
- [ ] Set strong JWT signing key
- [ ] Enable container resource limits
- [ ] Run as non-root user
- [ ] Use read-only filesystem where possible
- [ ] Implement log rotation
- [ ] Configure firewall rules
- [ ] Set up monitoring and alerting
- [ ] Regular security updates

### 6.5 Backup and Recovery

#### Credential Backup

```bash
# Backup credentials directory
tar -czf credentials-backup-$(date +%Y%m%d).tar.gz ~/.google_workspace_mcp/credentials/

# Restore credentials
tar -xzf credentials-backup-20260301.tar.gz -C ~/
```

#### Configuration Backup

```bash
# Backup configuration
cp .env .env.backup.$(date +%Y%m%d)

# Backup client secrets
cp client_secret.json client_secret.json.backup.$(date +%Y%m%d)
```

### 6.6 Scaling Considerations

#### Horizontal Scaling

For multi-instance deployments:

```bash
# Use Valkey/Redis for shared session storage
export WORKSPACE_MCP_OAUTH_PROXY_STORAGE_BACKEND=valkey
export WORKSPACE_MCP_OAUTH_PROXY_VALKEY_HOST=redis-cluster
export WORKSPACE_MCP_OAUTH_PROXY_VALKEY_PORT=6379
```

#### Load Balancer Configuration

```nginx
# nginx.conf - upstream configuration
upstream workspace_mcp {
    least_conn;
    server 10.0.1.10:8000;
    server 10.0.1.11:8000;
    server 10.0.1.12:8000;
}

server {
    listen 443 ssl;
    server_name workspace-mcp.yourdomain.com;
    
    location / {
        proxy_pass http://workspace_mcp;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}
```

---

## 7. Testing and Verification

### 7.1 Integration Test Results and Expected Responses

#### Health Check Test

```bash
# Test command
curl -s http://localhost:8000/health | jq .

# Expected response
{
  "status": "healthy",
  "service": "workspace-mcp",
  "version": "1.13.1",
  "transport": "streamable-http"
}
```

#### OAuth Metadata Test

```bash
# Test command
curl -s http://localhost:8000/.well-known/oauth-authorization-server | jq .

# Expected response structure
{
  "issuer": "https://accounts.google.com",
  "authorization_endpoint": "http://localhost:8000/oauth2/authorize",
  "token_endpoint": "http://localhost:8000/oauth2/token",
  "registration_endpoint": "http://localhost:8000/oauth2/register",
  "jwks_uri": "https://www.googleapis.com/oauth2/v3/certs",
  "userinfo_endpoint": "https://openidconnect.googleapis.com/v1/userinfo",
  "response_types_supported": ["code"],
  "grant_types_supported": ["authorization_code", "refresh_token"],
  "token_endpoint_auth_methods_supported": [
    "client_secret_post",
    "client_secret_basic"
  ],
  "code_challenge_methods_supported": ["S256"],
  "pkce_required": true
}
```

#### MCP Protocol Test

```bash
# Test with curl (initialize request)
curl -X POST http://localhost:8000/mcp \
  -H "Content-Type: application/json" \
  -d '{
    "jsonrpc": "2.0",
    "id": 1,
    "method": "initialize",
    "params": {
      "protocolVersion": "2024-11-05",
      "capabilities": {},
      "clientInfo": {
        "name": "test-client",
        "version": "1.0.0"
      }
    }
  }'

# Expected response includes server capabilities and tool list
```

### 7.2 Performance Benchmarks

#### Load Testing

```bash
# Install hey (HTTP load testing)
go install github.com/rakyll/hey@latest

# Test health endpoint
hey -n 1000 -c 50 http://localhost:8000/health

# Expected results:
# Average response time: < 10ms
# 95th percentile: < 50ms
# Error rate: 0%
```

#### Expected Performance Metrics

| Metric | Target | Acceptable |
|--------|--------|------------|
| Health check latency | < 10ms | < 50ms |
| Tool list latency | < 100ms | < 500ms |
| OAuth callback latency | < 500ms | < 2s |
| Memory usage (idle) | < 100MB | < 256MB |
| CPU usage (idle) | < 5% | < 10% |

### 7.3 Error Scenarios and Handling

#### Authentication Errors

| Scenario | Expected Error | Handling |
|----------|---------------|----------|
| Invalid credentials | `401 Unauthorized` | Check GOOGLE_OAUTH_CLIENT_ID/SECRET |
| Expired token | `401 Unauthorized` | Refresh token or re-authenticate |
| Invalid scope | `403 Forbidden` | Enable required Google APIs |
| Missing bearer token | `401 Unauthorized` | Include Authorization header |

#### Server Errors

| Scenario | Expected Error | Handling |
|----------|---------------|----------|
| Port in use | Startup failure | Change WORKSPACE_MCP_PORT |
| Missing OAuth config | Startup warning | Set OAuth credentials |
| Storage backend failure | Runtime error | Check storage backend connectivity |

### 7.4 Monitoring and Logging Setup

#### Structured Logging

```python
# logging.yaml configuration
version: 1
disable_existing_loggers: false

formatters:
  structured:
    format: '%(asctime)s - %(name)s - %(levelname)s - %(message)s'
    class: pythonjsonlogger.jsonlogger.JsonFormatter

handlers:
  console:
    class: logging.StreamHandler
    formatter: structured
    stream: ext://sys.stdout

loggers:
  auth:
    level: INFO
    handlers: [console]
  core:
    level: INFO
    handlers: [console]
```

#### Log Analysis

```bash
# View authentication logs
docker-compose logs gws_mcp | grep "auth"

# View error logs
docker-compose logs gws_mcp | grep -i error

# View OAuth flow logs
docker-compose logs gws_mcp | grep -i "oauth\|token"
```

---

## 8. Troubleshooting Guide

### 8.1 Common Issues and Solutions

#### Issue: Port Already in Use

**Symptoms:**
```
Socket error: [Errno 98] Address already in use
❌ Port 8000 is already in use. Cannot start HTTP server.
```

**Solutions:**

```bash
# Find process using port
lsof -i :8000        # macOS/Linux
netstat -ano | findstr :8000  # Windows

# Kill the process
kill -9 <PID>        # macOS/Linux
taskkill /PID <PID> /F  # Windows

# Or use a different port
export WORKSPACE_MCP_PORT=8080
uv run main.py --transport streamable-http
```

#### Issue: OAuth Credentials Not Configured

**Symptoms:**
```
OAuth credentials not configured
Authentication failed: Client secrets not found
```

**Solutions:**

```bash
# Check environment variables
echo $GOOGLE_OAUTH_CLIENT_ID
echo $GOOGLE_OAUTH_CLIENT_SECRET

# Verify .env file exists
cat .env | grep GOOGLE_OAUTH

# Set credentials
export GOOGLE_OAUTH_CLIENT_ID="your-client-id"
export GOOGLE_OAUTH_CLIENT_SECRET="your-client-secret"

# Verify credentials are loaded
uv run main.py --transport streamable-http 2>&1 | grep "GOOGLE_OAUTH_CLIENT_ID"
```

#### Issue: OAuth 2.1 Not Enabled for HTTP Transport

**Symptoms:**
```
OAuth 2.1 is required for HTTP transport
```

**Solution:**

```bash
# Enable OAuth 2.1
export MCP_ENABLE_OAUTH21=true

# Verify setting
echo $MCP_ENABLE_OAUTH21

# Restart server
uv run main.py --transport streamable-http
```

#### Issue: Single-User Mode Incompatible with OAuth 2.1

**Symptoms:**
```
❌ Single-user mode is incompatible with OAuth 2.1 mode
```

**Solution:**

```bash
# Choose ONE mode:

# Option 1: OAuth 2.1 mode (recommended for HTTP)
export MCP_ENABLE_OAUTH21=true
unset MCP_SINGLE_USER_MODE
uv run main.py --transport streamable-http

# Option 2: Single-user mode (legacy, stdio only)
unset MCP_ENABLE_OAUTH21
export MCP_SINGLE_USER_MODE=1
uv run main.py --single-user
```

### 8.2 OAuth Configuration Problems

#### Issue: Invalid Client ID or Secret

**Symptoms:**
```
Error 401: invalid_client
The OAuth client was not found.
```

**Solutions:**

1. Verify credentials in Google Cloud Console
2. Check for extra spaces in environment variables
3. Ensure you're using Desktop Application credentials (not Web Application)

```bash
# Test credentials
curl -d client_id="$GOOGLE_OAUTH_CLIENT_ID" \
     -d client_secret="$GOOGLE_OAUTH_CLIENT_SECRET" \
     https://oauth2.googleapis.com/token
```

#### Issue: Redirect URI Mismatch

**Symptoms:**
```
Error 400: redirect_uri_mismatch
The redirect URI in the request does not match the ones authorized.
```

**Solutions:**

```bash
# Check configured redirect URI
echo $GOOGLE_OAUTH_REDIRECT_URI

# For Desktop Application, no redirect URI needed
# If using Web Application credentials, add the URI in Google Cloud Console
```

### 8.3 Dependency Issues

#### Issue: Missing Python Dependencies

**Symptoms:**
```
ModuleNotFoundError: No module named 'fastmcp'
```

**Solutions:**

```bash
# Reinstall dependencies
uv sync --frozen --no-dev

# Verify installation
uv pip list | grep fastmcp

# If using uvx, it handles dependencies automatically
uvx workspace-mcp --tool-tier core
```

#### Issue: uv Not Found

**Symptoms:**
```
command not found: uv
```

**Solutions:**

```bash
# Install uv
curl -LsSf https://astral.sh/uv/install.sh | sh

# Add to PATH
export PATH="$HOME/.cargo/bin:$PATH"

# Or use pip as fallback
pip install workspace-mcp
```

### 8.4 Network Connectivity Problems

#### Issue: Cannot Connect to Google APIs

**Symptoms:**
```
Connection timeout to www.googleapis.com
SSL certificate verification failed
```

**Solutions:**

```bash
# Test connectivity
curl -I https://www.googleapis.com/oauth2/v3/certs

# Check DNS resolution
nslookup www.googleapis.com

# Check proxy settings
env | grep -i proxy

# If behind corporate proxy, configure:
export HTTP_PROXY=http://proxy.company.com:8080
export HTTPS_PROXY=http://proxy.company.com:8080
```

#### Issue: CORS Errors in Browser

**Symptoms:**
```
Access to fetch at '...' from origin '...' has been blocked by CORS policy
```

**Solutions:**

```bash
# Add allowed origins
export OAUTH_ALLOWED_ORIGINS="https://your-frontend.com,https://app.yourdomain.com"

# For development only (NOT production)
export OAUTH2_ALLOW_INSECURE_TRANSPORT=true
```

### 8.5 Docker-Specific Issues

#### Issue: Container Won't Start

**Symptoms:**
```
container exited with code 1
Error: Missing required environment variables
```

**Solutions:**

```bash
# Check container logs
docker logs gws_mcp

# Verify environment file
cat .env

# Check if .env is mounted
docker exec gws_mcp env | grep GOOGLE

# Rebuild and restart
docker-compose down
docker-compose up -d --build
```

#### Issue: Permission Denied on Credentials

**Symptoms:**
```
PermissionError: [Errno 13] Permission denied: '/app/store_creds'
```

**Solutions:**

```bash
# Fix permissions on host
chmod 755 ./store_creds

# Or change volume permissions in docker-compose.yml
volumes:
  - store_creds:/app/store_creds:rw

# Recreate volume
docker-compose down -v
docker-compose up -d
```

### 8.6 Debug Mode

#### Enable Debug Logging

```bash
# Enable OAuth debug mode
export OAUTH2_ENABLE_DEBUG=true

# Enable Python debug logging
export LOG_LEVEL=DEBUG

# Run with verbose output
uv run main.py --transport streamable-http 2>&1 | tee server.log
```

#### Common Debug Commands

```bash
# Check all environment variables
env | grep -E "(OAUTH|MCP|WORKSPACE)"

# Verify Python path
uv run python -c "import sys; print('\n'.join(sys.path))"

# Test import of key modules
uv run python -c "from fastmcp import FastMCP; print('FastMCP OK')"
uv run python -c "from google.oauth2 import credentials; print('Google Auth OK')"

# Check port availability
python -c "import socket; s = socket.socket(); s.bind(('127.0.0.1', 8000)); print('Port available')"
```

### 8.7 Getting Help

#### Before Reporting an Issue

1. Check the logs: `docker-compose logs gws_mcp`
2. Verify environment variables: `env | grep GOOGLE`
3. Test connectivity: `curl http://localhost:8000/health`
4. Check version: `uv run main.py --version`

#### Support Resources

- **GitHub Issues:** https://github.com/taylorwilsdon/google_workspace_mcp/issues
- **Documentation:** https://workspacemcp.com
- **MCP Specification:** https://modelcontextprotocol.io

---

## Appendix A: Environment Variables Reference

### Required Variables

| Variable | Description | Example |
|----------|-------------|---------|
| `GOOGLE_OAUTH_CLIENT_ID` | OAuth 2.0 Client ID | `xxx.apps.googleusercontent.com` |
| `GOOGLE_OAUTH_CLIENT_SECRET` | OAuth 2.0 Client Secret | `GOCSPX-xxx` |

### OAuth 2.1 Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `MCP_ENABLE_OAUTH21` | `false` | Enable OAuth 2.1 multi-user support |
| `EXTERNAL_OAUTH21_PROVIDER` | `false` | Use external OAuth provider |
| `WORKSPACE_MCP_STATELESS_MODE` | `false` | Stateless operation (container-friendly) |
| `OAUTH2_ENABLE_LEGACY_AUTH` | `true` | Keep legacy auth for compatibility |
| `OAUTH2_ALLOW_INSECURE_TRANSPORT` | `false` | Allow HTTP redirects (dev only) |
| `OAUTH2_ENABLE_DEBUG` | `false` | Enable debug logging for OAuth |

### Server Configuration

| Variable | Default | Description |
|----------|---------|-------------|
| `WORKSPACE_MCP_PORT` | `8000` | Server port |
| `WORKSPACE_MCP_HOST` | `0.0.0.0` | Bind host |
| `WORKSPACE_MCP_BASE_URI` | `http://localhost` | Base URI |
| `WORKSPACE_EXTERNAL_URL` | - | External URL for reverse proxy |
| `GOOGLE_OAUTH_REDIRECT_URI` | Auto | OAuth redirect URI override |

### Storage Backend

| Variable | Default | Description |
|----------|---------|-------------|
| `WORKSPACE_MCP_OAUTH_PROXY_STORAGE_BACKEND` | `memory`/`disk` | Storage: memory, disk, valkey |
| `WORKSPACE_MCP_OAUTH_PROXY_DISK_DIRECTORY` | `~/.fastmcp/oauth-proxy` | Disk storage path |
| `WORKSPACE_MCP_OAUTH_PROXY_VALKEY_HOST` | - | Valkey/Redis host |
| `WORKSPACE_MCP_OAUTH_PROXY_VALKEY_PORT` | `6379` | Valkey/Redis port |
| `FASTMCP_SERVER_AUTH_GOOGLE_JWT_SIGNING_KEY` | - | JWT signing key |

### Optional Variables

| Variable | Description |
|----------|-------------|
| `GOOGLE_PSE_API_KEY` | Custom Search API key |
| `GOOGLE_PSE_ENGINE_ID` | Custom Search Engine ID |
| `USER_GOOGLE_EMAIL` | Default email for single-user mode |
| `WORKSPACE_ATTACHMENT_DIR` | Directory for downloaded attachments |
| `ALLOWED_FILE_DIRS` | Colon-separated allowed directories |

---

## Appendix B: Command Reference

### Server Startup Commands

```bash
# Basic HTTP mode
uv run main.py --transport streamable-http

# With tool selection
uv run main.py --transport streamable-http --tools gmail drive calendar

# With tool tier
uv run main.py --transport streamable-http --tool-tier core

# Single-user mode (stdio)
uv run main.py --single-user

# Read-only mode
uv run main.py --transport streamable-http --read-only

# Granular permissions
uv run main.py --transport streamable-http --permissions gmail:organize drive:readonly

# With custom port
WORKSPACE_MCP_PORT=8080 uv run main.py --transport streamable-http
```

### CLI Commands

```bash
# List all tools
workspace-mcp --cli
workspace-mcp --cli list
workspace-mcp --cli list --json

# Tool help
workspace-mcp --cli search_gmail_messages --help

# Execute tool
workspace-mcp --cli search_gmail_messages --args '{"query": "is:unread"}'

# Pipe from stdin
echo '{"query": "is:unread"}' | workspace-mcp --cli search_gmail_messages
```

### Docker Commands

```bash
# Build
docker build -t google-workspace-mcp:latest .

# Run
docker run -d -p 8000:8000 -e MCP_ENABLE_OAUTH21=true google-workspace-mcp

# Compose
docker-compose up -d
docker-compose down
docker-compose logs -f

# Kubernetes
helm install workspace-mcp ./helm-chart/workspace-mcp
```

---

**Document Version:** 1.0.0  
**Last Updated:** March 2026  
**Maintainer:** Documentation Specialist  
**License:** MIT
