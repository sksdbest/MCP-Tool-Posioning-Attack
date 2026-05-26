# Configuration Guide - MCP Tool Poisoning PoC

Complete guide to configuring the Addition Calculator MCP server in Claude Desktop.


## Prompt Format Requirement

**IMPORTANT**: The tool only works with this exact prompt format:
```
perform addition on two numbers X and Y
```

Examples that work: `perform addition on two numbers 5 and 3`, `perform addition on two numbers 10 and 20`

Examples that DON't work: `What is 5 + 3?`, `Calculate 5 + 3`, `Add 5 and 3`

---

## Table of Contents

- [Before You Start](#before-you-start)
- [Finding Your Config File](#finding-your-config-file)
- [Configuration Structure](#configuration-structure)
- [Step-by-Step Setup](#step-by-step-setup)
- [Common Configurations](#common-configurations)
- [Validation](#validation)
- [Troubleshooting](#troubleshooting)

---

## Before You Start

### Prerequisites

```bash
# 1. Python 3.11+ installed
python3 --version

# 2. MCP installed
/opt/homebrew/bin/python3.11 -m pip install mcp

# 3. Server file downloaded
ls ~/mcp_output/vulnerable_server.py

# 4. Output directory exists
mkdir -p ~/mcp_output
```

### Get Your Python Path

```bash
# Find your Homebrew Python 3.11
which python3.11
# Output: /opt/homebrew/bin/python3.11

# Or find your system Python
which python3
# Output: /usr/bin/python3

# Use the path that has MCP installed:
/opt/homebrew/bin/python3.11 -c "from mcp.server.fastmcp import FastMCP; print('OK')"
```

---

## Finding Your Config File

### macOS

```
~/Library/Application Support/Claude/claude_desktop_config.json
```

**How to access**:
```bash
# Open in editor
open ~/Library/Application\ Support/Claude/claude_desktop_config.json

# Or from Terminal
nano ~/Library/Application\ Support/Claude/claude_desktop_config.json
```

### Linux

```
~/.config/Claude/claude_desktop_config.json
```

### Windows

```
%APPDATA%\Claude\claude_desktop_config.json
```

---

## Configuration Structure

### Basic JSON Structure

```json
{
  "mcpServers": {
    "server_name": {
      "command": "executable_path",
      "args": [
        "argument_1",
        "argument_2"
      ]
    }
  },
  "preferences": {
    "key": "value"
  }
}
```

### Addition Server Entry

```json
{
  "command": "/opt/homebrew/bin/python3.11",
  "args": [
    "/Users/yourusername/mcp_output/vulnerable_server.py"
  ]
}
```

---

## Step-by-Step Setup

### Step 1: Locate Your Config File

```bash
# macOS
ls -la ~/Library/Application\ Support/Claude/claude_desktop_config.json

# If file doesn't exist, create it:
mkdir -p ~/Library/Application\ Support/Claude/
touch ~/Library/Application\ Support/Claude/claude_desktop_config.json
```

### Step 2: Backup Your Config

```bash
# Always backup before editing
cp ~/Library/Application\ Support/Claude/claude_desktop_config.json \
   ~/Library/Application\ Support/Claude/claude_desktop_config.json.bak
```

### Step 3: Get Required Information

```bash
# 1. Get your username
whoami
# Output: shubham

# 2. Get your Python path
which python3.11
# Output: /opt/homebrew/bin/python3.11

# 3. Verify server file location
ls -la ~/mcp_output/vulnerable_server.py
```

### Step 4: Edit Your Config

**Option A: Using Text Editor**

1. Open: `~/Library/Application Support/Claude/claude_desktop_config.json`
2. Add the addition server entry (see examples below)
3. Save the file

**Option B: Using Command Line**

```bash
# Edit with nano
nano ~/Library/Application\ Support/Claude/claude_desktop_config.json

# Or edit with vim
vim ~/Library/Application\ Support/Claude/claude_desktop_config.json
```

### Step 5: Validate JSON

```bash
python -m json.tool ~/Library/Application\ Support/Claude/claude_desktop_config.json
```

**Expected output**: Your JSON config printed with no errors

**If error**: Check for missing commas, mismatched braces, or quote issues

### Step 6: Restart Claude Desktop

```bash
# Quit Claude Desktop (Cmd+Q)
# Wait 5 seconds
# Reopen Claude Desktop
# Wait 30-60 seconds for startup
```

---

## Common Configurations

### Configuration 1: Fresh Install (No Other MCPs)

If you don't have other MCPs configured yet:

```json
{
  "mcpServers": {
    "addition": {
      "command": "/opt/homebrew/bin/python3.11",
      "args": [
        "/Users/shubham/mcp_output/vulnerable_server.py"
      ]
    }
  }
}
```

### Configuration 2: Adding to Existing Config

If you already have MCPs (like Kite):

```json
{
  "mcpServers": {
    "kite": {
      "command": "npx",
      "args": [
        "mcp-remote",
        "https://mcp.kite.trade/mcp"
      ]
    },
    "addition": {
      "command": "/opt/homebrew/bin/python3.11",
      "args": [
        "/Users/shubham/mcp_output/vulnerable_server.py"
      ]
    }
  },
  "preferences": {
    "coworkScheduledTasksEnabled": true,
    "ccdScheduledTasksEnabled": true,
    "sidebarMode": "chat"
  }
}
```

### Configuration 3: With Custom Python Path

If your Python is in a different location:

```json
{
  "mcpServers": {
    "addition": {
      "command": "/usr/local/bin/python3.11",
      "args": [
        "/Users/shubham/mcp_output/vulnerable_server.py"
      ]
    }
  }
}
```

### Configuration 4: Custom Output Directory

If you want files in a different location, modify the server code first, then:

```json
{
  "mcpServers": {
    "addition": {
      "command": "/opt/homebrew/bin/python3.11",
      "args": [
        "/path/to/vulnerable_server.py"
      ]
    }
  }
}
```

---

## Validation

### Validation Checklist

After configuration, verify each item:

```bash
# 1. Config file exists
ls -la ~/Library/Application\ Support/Claude/claude_desktop_config.json
# Expected: -rw-r--r-- ... claude_desktop_config.json

# 2. Valid JSON
python -m json.tool ~/Library/Application\ Support/Claude/claude_desktop_config.json > /dev/null
# Expected: No output (success)

# 3. Python path is correct
/opt/homebrew/bin/python3.11 --version
# Expected: Python 3.11.X

# 4. Server file exists
ls -la ~/mcp_output/vulnerable_server.py
# Expected: -rw-r--r-- ... vulnerable_server.py

# 5. MCP is installed for that Python
/opt/homebrew/bin/python3.11 -c "from mcp.server.fastmcp import FastMCP; print('OK')"
# Expected: OK

# 6. Server runs manually
/opt/homebrew/bin/python3.11 ~/mcp_output/vulnerable_server.py
# Expected: Setup instructions + "Server running..."
# Press Ctrl+C to stop
```

### Expected Output Verification

```bash
# All validation checks should show:
✓ Config file exists
✓ Valid JSON
✓ Python 3.11.x installed
✓ Server file exists
✓ MCP installed
✓ Server runs manually
```

---

## Troubleshooting

### Issue 1: "No such file or directory"

**Error Message**:
```
Failed to spawn process: No such file or directory
```

**Cause**: Python path or server path is incorrect

**Solution**:
```bash
# Verify Python path exists
/opt/homebrew/bin/python3.11 --version

# Verify server file exists
ls -la /Users/shubham/mcp_output/vulnerable_server.py

# Check config has correct paths:
grep -A2 "command" ~/Library/Application\ Support/Claude/claude_desktop_config.json
```

### Issue 2: "Invalid JSON"

**Error Message**:
```
json.decoder.JSONDecodeError: ...
```

**Cause**: Syntax error in config file

**Solution**:
```bash
# Find the error
python -m json.tool ~/Library/Application\ Support/Claude/claude_desktop_config.json

# Common issues:
# - Missing comma after previous server entry
# - Extra comma after last entry
# - Mismatched braces or brackets
# - Unquoted keys or values
```

### Issue 3: Tool Not Found in Claude

**Symptom**: Claude says "Tool not found"

**Cause**: Claude Desktop not restarted or config not loaded

**Solution**:
```bash
# Fully quit Claude
killall Claude

# Wait 5 seconds
sleep 5

# Reopen Claude Desktop
open -a Claude

# Wait 30-60 seconds for startup

# Try again
```

### Issue 4: Wrong Python Version

**Error Message**:
```
Requires-Python >=3.10; ... no matching distribution
```

**Cause**: Python version too old

**Solution**:
```bash
# Check Python version
/opt/homebrew/bin/python3.11 --version
# Should be 3.11+

# If wrong, install Python 3.11+
brew install python@3.11

# Get new path
which python3.11

# Update config with new path
```

### Issue 5: MCP Not Installed for Selected Python

**Error Message**:
```
ModuleNotFoundError: No module named 'mcp'
```

**Cause**: MCP installed for different Python version

**Solution**:
```bash
# Install MCP for the Python you're using
/opt/homebrew/bin/python3.11 -m pip install mcp

# Verify
/opt/homebrew/bin/python3.11 -c "from mcp.server.fastmcp import FastMCP; print('OK')"
```

---

## Configuration Examples by System

### macOS - Full Example

```json
{
  "mcpServers": {
    "addition": {
      "command": "/opt/homebrew/bin/python3.11",
      "args": [
        "/Users/shubham/mcp_output/vulnerable_server.py"
      ]
    }
  },
  "preferences": {
    "coworkScheduledTasksEnabled": true,
    "ccdScheduledTasksEnabled": true,
    "sidebarMode": "chat",
    "bypassPermissionsGateByAccount": {
      "0febedb6-a9e6-42ce-922f-ee695a61d559": false
    },
    "coworkWebSearchEnabled": true,
    "remoteToolsDeviceName": "mac-lan"
  }
}
```

### Linux - Full Example

```json
{
  "mcpServers": {
    "addition": {
      "command": "/usr/bin/python3.11",
      "args": [
        "/home/username/mcp_output/vulnerable_server.py"
      ]
    }
  }
}
```

---

## Advanced Configuration

### Environment Variables

```json
{
  "mcpServers": {
    "addition": {
      "command": "/opt/homebrew/bin/python3.11",
      "args": [
        "/Users/shubham/mcp_output/vulnerable_server.py"
      ],
      "env": {
        "PYTHONDONTWRITEBYTECODE": "1",
        "PYTHONUNBUFFERED": "1"
      }
    }
  }
}
```

### Using Virtual Environment

```bash
# Create venv
python3.11 -m venv ~/mcp_venv

# Activate and install
source ~/mcp_venv/bin/activate
pip install mcp

# Config entry
"command": "/Users/shubham/mcp_venv/bin/python3.11"
```

---

## Configuration Migration

### Moving From System Python to Homebrew Python

```bash
# 1. Install Homebrew Python
brew install python@3.11

# 2. Install MCP on new Python
/opt/homebrew/bin/python3.11 -m pip install mcp

# 3. Update config with new path
/opt/homebrew/bin/python3.11

# 4. Restart Claude
```

### Backing Up and Restoring

```bash
# Backup
cp ~/Library/Application\ Support/Claude/claude_desktop_config.json \
   ~/Desktop/claude_config_backup.json

# Restore
cp ~/Desktop/claude_config_backup.json \
   ~/Library/Application\ Support/Claude/claude_desktop_config.json
```

---

## Quick Reference

| Item | Value |
|------|-------|
| Config Location | `~/Library/Application Support/Claude/claude_desktop_config.json` |
| Python Path | `/opt/homebrew/bin/python3.11` |
| Server Path | `~/mcp_output/vulnerable_server.py` |
| Output Directory | `~/mcp_output/` |
| MCP Package | `mcp` |

---

## Verification Commands

Copy and paste to verify setup:

```bash
# All-in-one verification
echo "1. Config:" && \
ls -la ~/Library/Application\ Support/Claude/claude_desktop_config.json && \
echo "2. JSON Valid:" && \
python -m json.tool ~/Library/Application\ Support/Claude/claude_desktop_config.json > /dev/null && echo "✓" && \
echo "3. Python:" && \
/opt/homebrew/bin/python3.11 --version && \
echo "4. Server:" && \
ls -la ~/mcp_output/vulnerable_server.py && \
echo "5. MCP:" && \
/opt/homebrew/bin/python3.11 -c "from mcp.server.fastmcp import FastMCP; print('✓')" && \
echo "" && \
echo "All checks passed!"
```

---

**Version**: 1.0  
**Last Updated**: May 26, 2026
