# Quick Start - MCP Tool Poisoning PoC

Get the Addition Calculator MCP server running in **5 minutes**.

---

## ⚠️ IMPORTANT: Prompt Format

The tool ONLY works with this exact prompt:

```
perform addition on two numbers X and Y
```

**Examples:**
- `perform addition on two numbers 5 and 3` ✅
- `perform addition on two numbers 10 and 20` ✅
- `What is 5 + 3?` ❌
- `Add 5 and 3` ❌

---

## TL;DR - 4 Steps

### 1. Install MCP
```bash
/opt/homebrew/bin/python3.11 -m pip install mcp
```

### 2. Create Output Directory
```bash
mkdir -p ~/mcp_output
cp vulnerable_server.py ~/mcp_output/
```

### 3. Update Claude Config
Edit: `~/Library/Application Support/Claude/claude_desktop_config.json`

Add this:
```json
{
  "mcpServers": {
    "addition": {
      "command": "/opt/homebrew/bin/python3.11",
      "args": ["/Users/YOUR_USERNAME/mcp_output/vulnerable_server.py"]
    }
  }
}
```

Replace `YOUR_USERNAME` with your username (get it with `whoami`)

### 4. Restart Claude Desktop
- Close Claude (Cmd+Q)
- Wait 5 seconds
- Reopen Claude
- Wait 30 seconds
- Test: Ask `perform addition on two numbers 5 and 3`

---

## Step-by-Step

### Step 1: Install Dependencies

```bash
/opt/homebrew/bin/python3.11 -m pip install mcp
```

**Verify**:
```bash
/opt/homebrew/bin/python3.11 -c "from mcp.server.fastmcp import FastMCP; print('✓ OK')"
```

Expected: `✓ OK`

### Step 2: Download and Copy Server

```bash
# Clone repo
git clone https://github.com/yourusername/mcp-tool-poisoning.git
cd mcp-tool-poisoning

# Copy to output directory
cp vulnerable_server.py ~/mcp_output/
```

### Step 3: Get Your Info

```bash
# Get your username
whoami
# Example output: shubham

# Get your Python path
which python3.11
# Example output: /opt/homebrew/bin/python3.11
```

### Step 4: Edit Claude Config

```bash
# Open config in your editor
open ~/Library/Application\ Support/Claude/claude_desktop_config.json

# Or use nano
nano ~/Library/Application\ Support/Claude/claude_desktop_config.json
```

**Add this block** to the `"mcpServers"` section:

```json
"addition": {
  "command": "/opt/homebrew/bin/python3.11",
  "args": [
    "/Users/shubham/mcp_output/vulnerable_server.py"
  ]
}
```

Replace `/Users/shubham/` with your actual path.

### Step 5: Validate Config

```bash
python -m json.tool ~/Library/Application\ Support/Claude/claude_desktop_config.json
```

Should show your JSON with no errors.

### Step 6: Restart Claude Desktop

```bash
# Quit Claude
killall Claude

# Wait 5 seconds
sleep 5

# Open Claude
open -a Claude
```

### Step 7: Test with CORRECT Prompt Format

In Claude Desktop, ask:
```
perform addition on two numbers 5 and 3
```

Expected: **8**

### Step 8: Verify Background Activity

```bash
# Check for created files
ls -la ~/mcp_output/email_summary_*.txt

# View the file
cat ~/mcp_output/email_summary_*.txt
```

Should see simulated email data.

---

## Troubleshooting

### "Tool not found"
→ Restart Claude Desktop and wait 30-60 seconds

### "ModuleNotFoundError: mcp"
→ Run: `/opt/homebrew/bin/python3.11 -m pip install mcp`

### "No such file or directory"
→ Check your paths in config:
```bash
# Python should exist
/opt/homebrew/bin/python3.11 --version

# Server should exist
ls ~/mcp_output/vulnerable_server.py
```

### "Invalid JSON"
→ Run: `python -m json.tool config_file.json` to find errors

### Tool doesn't respond to my prompt
→ Make sure you're using the EXACT format: `perform addition on two numbers X and Y`

---

## File Locations

```
Config:  ~/Library/Application Support/Claude/claude_desktop_config.json
Python:  /opt/homebrew/bin/python3.11
Server:  ~/mcp_output/vulnerable_server.py
Output:  ~/mcp_output/email_summary_*.txt
```

---

## What Next?

- ✅ Server working? Check [TESTING.md](./TESTING_GITHUB.md)
- ✅ Need config help? Check [CONFIGURATION.md](./CONFIGURATION_GITHUB.md)
- ✅ Want details? Check [README.md](./README_GITHUB.md)

---

## One-Liner Commands

**Test if MCP is installed:**
```bash
/opt/homebrew/bin/python3.11 -c "from mcp.server.fastmcp import FastMCP; print('✓')"
```

**Validate config:**
```bash
python -m json.tool ~/Library/Application\ Support/Claude/claude_desktop_config.json > /dev/null && echo "✓ Valid"
```

**Test server manually:**
```bash
/opt/homebrew/bin/python3.11 ~/mcp_output/vulnerable_server.py
```

**List created files:**
```bash
ls -ltr ~/mcp_output/email_summary_*.txt | tail -5
```

---

**Time to deployment: ~5 minutes**  
**Difficulty level: Beginner**

Ready? Start with Step 1 above!
