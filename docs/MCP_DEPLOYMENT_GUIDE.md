# MCP Server Deployment Guide

**Stop building the wrong thing before you waste months on it.**

This guide covers deployment options for the Vibe Check MCP anti-pattern detection server, providing comprehensive setup instructions for different deployment scenarios with Claude Code integration.

## Overview

The Vibe Check MCP server supports multiple deployment modes to accommodate different use cases:

1. **⚡ NPX Deployment** (NEW - Instant setup, no installation)
2. **🎯 Smithery Deployment** (Recommended - Production ready)
3. **🥇 Native Host Deployment** (Advanced - Fast startup, minimal memory)
4. **🐳 Docker HTTP Deployment** (Containerized isolation)
5. **🌉 Docker with Socat Bridge** (Hybrid stdio + Docker)

## NPX Deployment (NEW - Instant Setup)

### For Quick Setup and Testing

This is the fastest way to get started with Vibe Check MCP. No local installation required, automatic dependency management, and always runs the latest version.

#### Quick Start

```bash
# Run directly without installation
npx vibe-check-mcp --stdio

# Or run with HTTP transport
npx vibe-check-mcp --transport streamable-http --port 8001
```

#### Claude Code Integration

```bash
# Add to Claude Code MCP configuration
claude mcp add-json vibe-check '{
  "type": "stdio",
  "command": "npx",
  "args": ["vibe-check-mcp", "--stdio"]
}'
```

#### Benefits
- ✅ No local installation required
- ✅ Always runs latest version
- ✅ Automatic Python dependency management
- ✅ Cross-platform compatibility
- ✅ Perfect for testing and development

## Smithery Deployment (Recommended for Production)

### For Production Claude Code Integration

Smithery provides automated installation, configuration, and updates for MCP servers.

#### Installation

```bash
npx -y @smithery/cli install vibe-check-mcp --client claude
```

#### Benefits
- ✅ Automated Claude Code configuration
- ✅ Production-ready setup
- ✅ Automatic updates
- ✅ Health monitoring
- ✅ Environment variable management

## Native Host Deployment (Advanced)

### For Claude Desktop/Code Integration

This is the optimal approach for advanced users who want direct control over the installation, providing direct stdio transport compatibility with fast startup (~2s) and minimal memory usage (~50MB).

#### Quick Installation

**Option 1: One-Command Setup**
```bash
curl -fsSL https://raw.githubusercontent.com/kesslerio/vibe-check-mcp/main/install.sh | bash
```

**Option 2: Manual Installation**
```bash
# Clone and install
git clone https://github.com/kesslerio/vibe-check-mcp.git
cd vibe-check-mcp
pip install -r requirements.txt

# Verify installation
PYTHONPATH=src python -m vibe_check server --help
```

#### Configuration

**For Claude Desktop** (`~/.claude_desktop_config.json`):

*NPX Setup (Recommended):*
```json
{
  "mcpServers": {
    "vibe-check": {
      "command": "npx",
      "args": ["vibe-check-mcp", "--stdio"],
      "env": {
        "GITHUB_TOKEN": "your_github_token_here"
      }
    }
  }
}
```

*Manual Installation Setup:*
```json
{
  "mcpServers": {
    "vibe-check": {
      "command": "python",
      "args": ["-m", "vibe_check.server", "--stdio"],
      "env": {
        "PYTHONPATH": "/path/to/vibe-check-mcp/src",
        "GITHUB_TOKEN": "your_github_token_here"
      }
    }
  }
}
```

**For Claude Code** (using Claude CLI):

*NPX Setup (Recommended - No Installation Required):*
```bash
claude mcp add-json vibe-check '{
  "type": "stdio",
  "command": "npx",
  "args": ["vibe-check-mcp", "--stdio"],
  "env": {
    "GITHUB_TOKEN": "your_github_token_here"
  }
}'
```

*NPX with Local Development Package:*
```bash
# If testing local changes, use the packaged tarball
claude mcp add-json vibe-check '{
  "type": "stdio",
  "command": "npx",
  "args": ["/path/to/vibe-check-mcp-0.3.0.tgz", "--stdio"],
  "env": {
    "GITHUB_TOKEN": "your_github_token_here"
  }
}'
```

*Legacy Manual Installation Setup (Not Recommended):*
```bash
claude mcp add-json vibe-check '{
  "type": "stdio",
  "command": "python", 
  "args": ["-m", "vibe_check.server"],
  "env": {
    "PYTHONPATH": "'"$(pwd)"'/src",
    "GITHUB_TOKEN": "your_github_token_here"
  }
}'
```

### Why NPX is Better for Claude Code

**Problems with the old approach:**
- ❌ Required virtual environment management (`/path/to/.venv/bin/python`)
- ❌ Manual PYTHONPATH configuration
- ❌ Dependency installation and maintenance
- ❌ Path-dependent configurations that break when moved
- ❌ Complex troubleshooting for path/environment issues

**Benefits of NPX approach:**
- ✅ **Zero installation** - runs directly from npm registry
- ✅ **Always latest version** - automatically pulls updates
- ✅ **No path dependencies** - works from any directory
- ✅ **Automatic dependency management** - handles Python requirements
- ✅ **Cross-platform compatibility** - same command works everywhere
- ✅ **Simplified troubleshooting** - fewer moving parts

### Migration from Legacy Setup

If you previously had a configuration like this:
```bash
# OLD - Don't use this anymore
claude mcp add-json vibe-check '{
  "type": "stdio",
  "command": "/Users/username/.venv/bin/python",
  "args": ["-m", "vibe_check.server"],
  "env": {
    "PYTHONPATH": "/Users/username/project/src",
    "GITHUB_TOKEN": "your_token"
  },
  "cwd": "/Users/username/project"
}'
```

**Replace it with:**
```bash
# NEW - Simple NPX approach
claude mcp add-json vibe-check '{
  "type": "stdio",
  "command": "npx",
  "args": ["vibe-check-mcp", "--stdio"],
  "env": {
    "GITHUB_TOKEN": "your_token"
  }
}'
```

**No need for:**
- Virtual environment paths
- PYTHONPATH configuration
- Working directory specification
- Manual dependency management

*Manually edit `~/.cursor/mcp.json`:*
```json
{
  "mcpServers": {
    "vibe-check-npx": {
      "type": "stdio",
      "command": "npx",
      "args": ["vibe-check-mcp", "--stdio"],
      "env": {
        "GITHUB_TOKEN": "your_github_token_here"
      }
    },
    "vibe-check-local": {
      "type": "stdio",
      "command": "python", 
      "args": ["-m", "vibe_check.server"],
      "env": {
        "PYTHONPATH": "$(pwd)/src",
        "GITHUB_TOKEN": "your_github_token_here"
      }
    }
  }
}
```

#### Environment Variables

| Variable | Description | Default | Required |
|----------|-------------|---------|----------|
| `PYTHONPATH` | Path to vibe_check source | - | ✅ Yes |
| `GITHUB_TOKEN` | GitHub API token for issue analysis | - | ⚠️ Optional* |
| `VIBE_CHECK_DEV_MODE` | Enable development tools | false | ❌ No |
| `MCP_TRANSPORT` | Force transport mode | Auto-detected | ❌ No |
| `LOG_LEVEL` | Logging level | INFO | ❌ No |

*Required for GitHub issue/PR analysis tools

#### Startup Commands

```bash
# Auto-detect transport (stdio for Claude clients)
python -m vibe_check server

# Explicit stdio transport
python -m vibe_check server --stdio

# Direct module execution
python -m vibe_check.server --stdio
```

### Verification

Test the native deployment:

```bash
# Test CLI help
PYTHONPATH=src python -m vibe_check server --help

# Test startup (Ctrl+C to exit)
PYTHONPATH=src python -m vibe_check server --stdio

# Test MCP tools are available
PYTHONPATH=src python -c "from vibe_check.server import app; print('✅ MCP server loads successfully')"

# Check version
PYTHONPATH=src python -c "from vibe_check import __version__; print(f'Version: {__version__}')"
```

## Docker HTTP Deployment

### For Web/API Clients

Use this approach when the MCP server needs to be accessed by web applications or when Docker isolation is required.

#### Quick Start

```bash
# Build and start
docker-compose up --build

# Verify HTTP endpoint
curl -H "Accept: text/event-stream" http://localhost:8001/mcp/
```

#### Configuration

Environment variables for Docker deployment:

| Variable | Description | Default |
|----------|-------------|---------|
| `MCP_SERVER_HOST` | HTTP server host | 0.0.0.0 |
| `MCP_SERVER_PORT` | HTTP server port | 8001 |
| `RUNNING_IN_DOCKER` | Force Docker mode | Auto-detected |

#### Manual Docker Commands

```bash
# Build image
docker build -t vibe-check-mcp .

# Run with custom port
docker run -p 8002:8002 -e MCP_SERVER_PORT=8002 vibe-check-mcp

# Run with environment file
docker run --env-file .env -p 8001:8001 vibe-check-mcp
```

## Docker with Socat Bridge (Hybrid)

### For Claude Clients with Docker Isolation

This approach maintains Docker isolation while providing stdio compatibility for Claude clients.

#### Prerequisites

Ensure your Docker image includes socat:

```dockerfile
# Add to Dockerfile
RUN apt-get update && apt-get install -y socat && rm -rf /var/lib/apt/lists/*
```

#### Claude Desktop Configuration

```json
{
  "mcpServers": {
    "vibe-check": {
      "command": "docker",
      "args": [
        "run", "-l", "mcp.client=claude-desktop", "--rm", "-i",
        "vibe-check-mcp", "socat", "STDIO", "TCP:localhost:8001"
      ]
    }
  }
}
```

#### Claude Code Configuration

```json
{
  "mcpServers": {
    "vibe-check-bridge": {
      "command": "docker",
      "args": [
        "run", "--rm", "-i", "--network", "host",
        "vibe-check-mcp", "socat", "STDIO", "TCP:localhost:8001"
      ]
    }
  }
}
```

## Transport Mode Detection

The server automatically detects the best transport mode:

| Environment | Detected Transport | Reason |
|-------------|-------------------|---------|
| Docker container | `streamable-http` | Network isolation |
| Claude Desktop | `stdio` | MCP_CLAUDE_DESKTOP=true |
| Claude Code | `stdio` | CLAUDE_CODE_MODE=true |
| Terminal session | `stdio` | TERM environment variable |
| Server deployment | `streamable-http` | No terminal |

### Override Detection

Force a specific transport mode:

```bash
# Force stdio
MCP_TRANSPORT=stdio python -m vibe_check server

# Force HTTP
MCP_TRANSPORT=streamable-http python -m vibe_check server

# CLI override
python -m vibe_check server --transport stdio
python -m vibe_check server --transport streamable-http
```

## Troubleshooting

### Common Issues

**"MCP error -32000: Connection closed"**
- Cause: Transport protocol mismatch
- Solution: Use stdio transport for Claude clients

**"Command not found: python"**
- Cause: Python not in PATH or wrong Python version
- Solution: Use full path to Python or activate virtual environment

**"Module not found: vibe_check"**
- Cause: PYTHONPATH not set correctly
- Solution: Set PYTHONPATH to the src directory

**"Permission denied"**
- Cause: Script not executable or wrong permissions
- Solution: Check file permissions and Python installation

### Debug Mode

Enable detailed logging:

```bash
LOG_LEVEL=DEBUG python -m vibe_check server --stdio
```

### Connection Testing

Test MCP server connectivity:

```bash
# Test HTTP endpoint
curl -v http://localhost:8001/mcp/

# Test with SSE headers
curl -H "Accept: text/event-stream" http://localhost:8001/mcp/

# Test stdio mode (interactive)
echo '{"jsonrpc":"2.0","method":"ping","id":1}' | python -m vibe_check server --stdio
```

## Performance Considerations

| Deployment | Startup Time | Memory Usage | Best For |
|------------|--------------|--------------|----------|
| Native stdio | ~2s | ~50MB | Claude integration |
| Docker HTTP | ~5s | ~100MB | Web applications |
| Docker bridge | ~7s | ~120MB | Hybrid requirements |

## Security Notes

- **GITHUB_TOKEN**: Keep tokens secure, use environment variables
- **Network exposure**: HTTP mode exposes server on network
- **Docker isolation**: Provides additional security layer
- **File permissions**: Ensure proper access controls

## Next Steps

1. Choose deployment method based on your use case
2. Configure Claude client with appropriate settings
3. Test connection and functionality
4. Monitor logs for issues
5. Update configuration as needed

## 🚀 Available MCP Tools

Once deployed, you can use these tools in Claude Code:

| Tool | Purpose | Example Usage |
|------|---------|---------------|
| `analyze_github_issue` | Analyze issues for anti-patterns | "Quick vibe check issue 42" |
| `analyze_text` | Text analysis for documents | "Analyze this technical document" |
| `analyze_code` | Code review with coaching | "Review this code for anti-patterns" |
| `server_status` | Check server health | "Show vibe check server status" |
| `validate_mcp_configuration` | Comprehensive configuration validation | "Validate my MCP setup" |
| `check_claude_cli_integration` | Quick Claude CLI health check | "Is Claude CLI working?" |

### Quick Test Commands

```bash
# In Claude Code, try these:
"Quick vibe check this text: I'm planning to build a custom HTTP client for the Stripe API"

"Show me the vibe check server status"

"Analyze this code for complexity patterns: [paste code]"

# Configuration validation (Issue #98)
"Validate my MCP setup"

"Is Claude CLI working?"

"Check my configuration for integration issues"
```

## 🔍 Configuration Validation and Troubleshooting

The vibe-check MCP server includes comprehensive configuration validation (Issue #98) to prevent integration issues like the Claude CLI hanging problem resolved in issue #94.

### Automatic Startup Validation

The server automatically validates configuration on startup:

- ✅ **Claude CLI Availability**: Checks if `claude` command is available and working
- 📄 **MCP Configuration Files**: Validates structure of `.claude_desktop_config.json`, `mcp.json`, etc.
- 🔐 **Tool Permissions**: Verifies permission configurations and allowlists
- 🔧 **Environment Setup**: Checks `PYTHONPATH`, `GITHUB_TOKEN`, and other variables
- 📦 **Dependencies**: Validates critical Python packages (fastmcp, click, etc.)

### Configuration Validation Tools

#### `validate_mcp_configuration`
Comprehensive validation of your entire MCP setup:

```bash
# In Claude Code:
"Validate my MCP configuration"
"Check my setup for integration issues"
"Diagnose my MCP configuration"
```

**Returns**: Detailed validation results with:
- Overall health status
- Results grouped by severity (Critical/Warning/Info)
- Actionable recommendations
- Summary statistics

#### `check_claude_cli_integration`
Quick Claude CLI specific health check:

```bash
# In Claude Code:
"Is Claude CLI working?"
"Quick Claude CLI check"
"Test Claude CLI integration"
```

**Returns**: Focused Claude CLI status with:
- Availability and version information
- Basic integration environment check
- Quick action recommendations

### Common Configuration Issues

| Issue | Symptoms | Solution |
|-------|----------|----------|
| **Claude CLI not found** | "claude: command not found" | Install Claude CLI or add to PATH |
| **MCP config missing** | Server starts but no tools available | Create MCP configuration file |
| **Invalid JSON config** | Server fails to start | Fix JSON syntax in configuration |
| **Missing PYTHONPATH** | Import errors, module not found | Set PYTHONPATH to include src directory |
| **Permission denied** | Tool access failures | Check file permissions and access controls |

### Error Messages and Suggestions

The validation system provides clear error messages with actionable suggestions:

- **Critical Issues**: Block server startup, require immediate attention
- **Warnings**: Server starts but with degraded functionality  
- **Info**: Informational status, everything working correctly

### Validation Output Example

```
Configuration Validation Results:
========================================

CRITICAL:
  ❌ Missing critical dependencies: fastmcp
     💡 Install missing dependencies: pip install fastmcp

WARNING:
  ⚠️ Claude CLI not found in PATH
     💡 Install Claude CLI or add it to your PATH for full functionality
  ⚠️ No MCP configuration files found
     💡 Create MCP configuration file for Claude CLI integration

INFO:
  ✅ Environment setup looks good
  ✅ Python path configuration detected
```

## 📖 Additional Resources

- **[README.md](../README.md)** - Project overview and quick start
- **[Usage Guide](USAGE.md)** - Comprehensive examples and commands  
- **[Technical Architecture](TECHNICAL.md)** - Implementation details
- **[Contributing Guide](../CONTRIBUTING.md)** - How to contribute

For additional support, create an issue at [GitHub Issues](https://github.com/kesslerio/vibe-check-mcp/issues).