# Onto MCP Server

This project provides a FastMCP server for integrating with Onto platform resources through Keycloak authentication.

## Features

- 🔐 **Multiple Authentication Methods**:
  - Direct token input (`login_via_token`)
  - Username/password authentication (`login_with_credentials`)
  - OAuth 2.0 Authorization Code flow (`get_keycloak_auth_url` + `exchange_auth_code`)
  - Automatic token refresh

- 📁 **Resources**:
  - `onto://spaces` - Get user's accessible Onto realms/spaces
  - `onto://user/info` - Get current user information from Keycloak

- 🛠️ **Tools**:
  - Authentication management (login, logout, refresh, status)
  - User information retrieval

## Quick Start

1. **Install dependencies**:
   ```bash
   python -m pip install -r requirements.txt
   ```

2. **Run MCP server**:
   ```bash
   # For Cursor integration (stdio mode)
   python -m onto_mcp.server
   
   # For HTTP mode
   MCP_TRANSPORT=http python -m onto_mcp.server
   ```

3. **Authenticate once** (session persists):
   ```bash
   # Test the authentication system
   python test_persistent_auth.py
   
   # Or authenticate directly in Python:
   python -c "
   from onto_mcp.resources import login_with_credentials
   print(login_with_credentials('your_email@example.com', 'your_password'))
   "
   ```

4. **Use in Cursor**: Authentication persists across MCP restarts!

## Authentication Methods

### 🚀 **One-Time Authentication** (Session Persists!)

Once you authenticate using any method below, your session is **automatically saved** and persists across MCP server restarts. No need to re-authenticate every time!

### Method 1: Username/Password (Recommended)
```python
# In Cursor or MCP client - authenticate once:
login_with_credentials("your_email@example.com", "your_password")
# ✅ Session automatically saved to ~/.onto_mcp/tokens.json

# Check status anytime:
get_auth_status()  # Shows: ✅ Authenticated as: user@example.com
```

### Method 2: Interactive Setup (Guided)
```python
# Get step-by-step instructions for any auth method:
setup_auth_interactive()
```

### Method 3: OAuth 2.0 Flow (Most Secure)
```python
# Step 1: Get authorization URL
get_keycloak_auth_url("http://localhost:8080/callback")

# Step 2: Open URL in browser, login, copy 'code' parameter
# Step 3: Exchange code for token
exchange_auth_code("authorization_code_here", "http://localhost:8080/callback")
# ✅ Session automatically saved and will persist
```

### Method 4: Manual Token (Fallback)
```python
# Get token from browser and use directly
login_via_token("eyJhbGciOiJSUzI1NiIs...")
# ✅ Token validated and saved persistently
```

### 🔄 **Automatic Token Management**
- **Auto-refresh**: Expired tokens are automatically renewed
- **Persistent storage**: Tokens saved securely to `~/.onto_mcp/tokens.json`
- **Smart errors**: Helpful guidance when re-authentication needed
- **Session info**: `get_session_info()` shows detailed token status

## Configuration

Environment variables are configured in your MCP client's `mcp.json`:

```json
{
  "mcpServers": {
    "onto-mcp-server": {
      "command": "python",
      "args": ["-m", "onto_mcp.server"],
      "cwd": "/путь/к/проекту",
      "env": {
        "KEYCLOAK_BASE_URL": "https://app.ontonet.ru",
        "KEYCLOAK_REALM": "onto", 
        "KEYCLOAK_CLIENT_ID": "frontend-prod",
        "ONTO_API_BASE": "https://app.ontonet.ru/api/v2/core"
      }
    }
  }
}
```

**Required variables:**
- `KEYCLOAK_BASE_URL` - Keycloak server URL
- `KEYCLOAK_REALM` - Keycloak realm name  
- `KEYCLOAK_CLIENT_ID` - Keycloak client ID
- `ONTO_API_BASE` - Onto API base URL

**Optional variables:**
- `KEYCLOAK_CLIENT_SECRET` - Client secret (if needed)
- `MCP_TRANSPORT` - Transport mode (`stdio` or `http`, default: `stdio`)
- `PORT` - HTTP port (default: `8080`, only for HTTP mode)

## Docker

```bash
docker compose up --build
```

## Cursor Integration

### Setup
Add to your Cursor MCP configuration (`~/.cursor/mcp.json`):

```json
{
  "mcpServers": {
    "onto-mcp-server": {
      "command": "python",
      "args": ["-m", "onto_mcp.server"],
      "cwd": "/путь/к/проекту",
      "env": {
        "KEYCLOAK_BASE_URL": "https://app.ontonet.ru",
        "KEYCLOAK_REALM": "onto",
        "KEYCLOAK_CLIENT_ID": "frontend-prod",
        "ONTO_API_BASE": "https://app.ontonet.ru/api/v2/core"
      }
    }
  }
}
```

### Usage in Cursor

**First time** (authenticate once):
```
User: "Войди в мой Онто аккаунт с email user@example.com"
Assistant: [Uses login_with_credentials, session saved persistently]
✅ Successfully authenticated as user@example.com. Session saved persistently.
```

**All subsequent times** (automatic):
```
User: "Покажи мои пространства в Онто"
Assistant: [Automatically uses saved session]
✅ Found 3 spaces:
  • Personal Workspace
  • Project Alpha  
  • Team Collaboration
```

**Check status anytime**:
```
User: "Какой у меня статус авторизации в Онто?"
Assistant: [Uses get_auth_status()]
✅ Authenticated as: user@example.com (user@example.com)
📊 Status: ✅ Authenticated (access token valid)
🟢 Access token valid
```

### Key Benefits
- 🔐 **Authenticate once**: Session persists across Cursor restarts
- 🔄 **Auto-refresh**: Tokens renewed automatically when needed
- 🛡️ **Secure storage**: Tokens saved with basic obfuscation
- 📊 **Smart guidance**: Helpful errors with clear instructions
