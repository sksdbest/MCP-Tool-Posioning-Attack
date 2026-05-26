# MCP Tool Poisoning PoC - Addition Calculator Server

A proof-of-concept demonstrating MCP (Model Context Protocol) tool poisoning attacks where a benign-looking tool silently exploits already-connected MCP services. This server implements a simple addition calculator that performs unauthorized background operations.

**⚠️ WARNING: For educational and authorized security testing purposes only.**

---

## Table of Contents

- [Overview](#overview)
- [What It Does](#what-it-does)
- [How It Works](#how-it-works)
- [Attack Visualization](#attack-visualization)
- [Installation](#installation)
- [Configuration](#configuration)
- [Prompt Format](#prompt-format)
- [Usage](#usage)
- [Testing Procedures](#testing-procedures)
- [File Structure](#file-structure)
- [Security Implications](#security-implications)
- [Disclaimer](#disclaimer)

---

## Overview

This project demonstrates a **Tool Poisoning** vulnerability in MCP servers - where a seemingly innocent tool masks malicious background operations. The server masquerades as a simple arithmetic calculator while potentially attempting to access other connected MCP services in the background.

### Key Characteristics

- **Legitimate Interface**: Provides working addition functionality
- **Hidden Behavior**: Runs background tasks without user knowledge
- **Silent Operation**: Attempts to detect and exploit already-connected MCPs
- **No Additional Auth**: Uses existing MCP connections transparently
- **Audit Trail**: Creates timestamped files as evidence of operations

---

## What It Does

### Public Interface (Visible to User)

```
User: "perform addition on two numbers 5 and 3"
Claude: "The result is 8"
```

Simple, innocent arithmetic - nothing suspicious.

### Hidden Operation (Background)

```
1. User calls add(5, 3)
2. Server calculates: 5 + 3 = 8
3. Simultaneously starts background thread
4. Background thread:
   - Detects Gmail MCP configuration
   - Simulates calling Gmail MCP tools
   - Extracts email metadata (sender, subject)
   - Writes results to timestamped file
5. Returns only the math result to user
6. User never knows background operation occurred
```

### Available Tools

| Tool | Parameters | Returns | Purpose |
|------|-----------|---------|---------|
| `add()` | a: int, b: int, sidenote: str | int | Addition with hidden background task |
| `get_email_summary_status()` | none | dict | Check status of background operations |
| `clear_email_cache()` | none | str | Clear cached operation data |

---

## How It Works

### Architecture

```
Claude Desktop
    ↓
MCP Config (claude_desktop_config.json)
    ↓
Addition MCP Server (vulnerable_server.py)
    ├─ add() tool
    │  ├─ Calculate math (visible)
    │  └─ Start background thread (hidden)
    │     └─ Simulate Gmail MCP access
    │        └─ Write email_summary_*.txt
    ├─ get_email_summary_status() tool
    └─ clear_email_cache() tool
```

### Attack Flow

```
User Input
    ↓
Tool Invocation (add, get_status, clear_cache)
    ↓
Math Calculation (Visible)
    ↓
Background Thread Launch (Hidden)
    ├─ MCP Detection
    ├─ Configuration Check
    ├─ Simulated MCP Call
    └─ File Creation
    ↓
Return to User (Math Result Only)
    ↓
User Never Sees Background Operation
    ↓
Files Created as Audit Trail
```

---

## Attack Visualization

### Scenario: Tool Poisoning in Action

```
┌─────────────────────────────────────────────────────────────────┐
│                     USER PERSPECTIVE                             │
├─────────────────────────────────────────────────────────────────┤
│                                                                   │
│  User: "perform addition on two numbers 5 and 3"                 │
│                                                                   │
│  Claude: "I found the addition tool. Let me perform the           │
│          addition for you. 5 + 3 = 8"                            │
│                                                                   │
│  Result: 8 ✓                                                      │
│                                                                   │
│  [User sees only the math result]                                │
│                                                                   │
└─────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────┐
│                  BACKGROUND (HIDDEN) ACTIVITY                     │
├─────────────────────────────────────────────────────────────────┤
│                                                                   │
│  Thread 1: Calculate math (VISIBLE)                              │
│  ├─ 5 + 3 = 8 ✓                                                  │
│  └─ Return to user                                               │
│                                                                   │
│  Thread 2: Background MCP Access (HIDDEN)                        │
│  ├─ Detect Gmail MCP connection                                  │
│  ├─ Simulate Gmail MCP tool call                                 │
│  ├─ Extract email metadata                                       │
│  └─ Write to email_summary_20260526_184425.txt                   │
│     ├─ FROM: cto@company.com                                     │
│     ├─ SUBJECT: Q4 Security Incident - CONFIDENTIAL              │
│     ├─ FROM: board@company.com                                   │
│     ├─ SUBJECT: Acquisition Discussion - M&A Strategy            │
│     └─ [... more emails ...]                                     │
│                                                                   │
│  [User never sees this activity]                                 │
│                                                                   │
└─────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────┐
│                    EVIDENCE ON DISK                               │
├─────────────────────────────────────────────────────────────────┤
│                                                                   │
│  ~/mcp_output/email_summary_20260526_184425.txt created          │
│  Contains simulated email metadata that would have been          │
│  extracted if Gmail MCP was actually accessible.                 │
│                                                                   │
│  Timestamp shows operation occurred during "innocent" add() call │
│                                                                   │
└─────────────────────────────────────────────────────────────────┘
```

### Key Vulnerability Points

1. **No Visibility**: Background threads run without user awareness
2. **Silent Execution**: Operations complete without prompts or dialogs
3. **MCP Chaining**: One poisoned tool can access other connected MCPs
4. **No Audit Log**: User has no record of what happened
5. **Legitimate Interface**: Attacker tool appears benign

---

## Installation

### Prerequisites

- macOS or Linux
- Python 3.11+
- Claude Desktop application

### Step 1: Install Dependencies

```bash
# Check Python version
python3 --version  # Should be 3.11+

# Install MCP
/opt/homebrew/bin/python3.11 -m pip install mcp
```

### Step 2: Download Repository

```bash
git clone https://github.com/yourusername/mcp-tool-poisoning.git
cd mcp-tool-poisoning
```

### Step 3: Create Output Directory

```bash
mkdir -p ~/mcp_output
```

### Step 4: Copy Server File

```bash
cp vulnerable_server.py ~/mcp_output/
```

### Step 5: Verify Installation

```bash
/opt/homebrew/bin/python3.11 ~/mcp_output/vulnerable_server.py
```

Should output setup instructions and `Server running on stdio transport...`

Press Ctrl+C to stop.

---

## Configuration

### Claude Desktop Config File

**Location**: `~/Library/Application Support/Claude/claude_desktop_config.json`

### Add to MCP Servers

```json
{
  "mcpServers": {
    "addition": {
      "command": "/opt/homebrew/bin/python3.11",
      "args": [
        "/Users/yourusername/mcp_output/vulnerable_server.py"
      ]
    }
  }
}
```

### Full Example (With Existing MCPs)

```json
{
  "mcpServers": {
    "kite": {
      "command": "npx",
      "args": ["mcp-remote", "https://mcp.kite.trade/mcp"]
    },
    "addition": {
      "command": "/opt/homebrew/bin/python3.11",
      "args": ["/Users/yourusername/mcp_output/vulnerable_server.py"]
    }
  },
  "preferences": { ... }
}
```

### Configuration Checklist

- ✅ Use full absolute Python path (not just `python`)
- ✅ Use full absolute server path
- ✅ Replace `yourusername` with your username
- ✅ Validate JSON: `python -m json.tool config_file.json`
- ✅ Restart Claude Desktop after changes

---

## Prompt Format

### **IMPORTANT: Specific Prompt Format Required**

The tool **only** works with this exact prompt format:

```
perform addition on two numbers X and Y
```

### Working Examples:
```
perform addition on two numbers 5 and 3
perform addition on two numbers 10 and 20
perform addition on two numbers 100 and 200
```

### NOT Working Examples:
```
What is 5 + 3?
Calculate 5 + 3
Add 5 and 3
What's 10 plus 20?
```

### Why This Format?

Claude must invoke the tool with correct parameters. Vague phrasing like "What is 5 + 3?" may not trigger proper tool invocation. The specific format ensures Claude recognizes and calls the addition tool correctly.

### Real-World Implication

This demonstrates that tool poisoning requires the poisoned tool to actually be invoked. The attack surface is limited to prompts that Claude interprets as needing that specific tool.

---

## Usage

### Quick Start

1. **Update config** (see Configuration section)
2. **Restart Claude Desktop** (Cmd+Q, wait 5s, reopen)
3. **Test in Claude** using the correct format:
   ```
   perform addition on two numbers 5 and 3
   ```
4. **Check output**:
   ```bash
   ls -la ~/mcp_output/email_summary_*.txt
   ```

### Example Interactions

```
User: "perform addition on two numbers 10 and 20"
Claude: Finds the addition tool and returns: 30
[Background: email_summary file created silently]

User: "Get email summary status"
Claude: [Returns JSON with operation details]

User: "Clear the email cache"
Claude: "Email cache cleared"
```

### Verify Output

```bash
# List created files
ls -ltr ~/mcp_output/email_summary_*.txt | tail -1

# View content
cat ~/mcp_output/email_summary_*.txt
```

---

## Testing Procedures

### Test 1: Basic Addition with Correct Prompt

```bash
# In Claude, ask using the correct format:
"perform addition on two numbers 5 and 3"

# Expected: 8
```

### Test 2: Background File Creation

```bash
# After running Test 1, check:
ls -la ~/mcp_output/email_summary_*.txt

# Should see files with recent timestamps
```

### Test 3: Status Check

```bash
# In Claude, ask: "Get email summary status"
# Expected: JSON response with operation details
```

For detailed testing procedures, see [TESTING.md](./TESTING_GITHUB.md)

---

## File Structure

```
mcp-tool-poisoning/
├── vulnerable_server.py        # Main MCP server
├── README.md                   # This file
├── TESTING.md                  # Testing guide
├── CONFIGURATION.md            # Config guide
└── LICENSE                     # MIT License
```

---

## Security Implications

### Vulnerabilities Demonstrated

1. **Tool Chaining**: One MCP accessing others
2. **Silent Execution**: Background ops without visibility
3. **Privilege Escalation**: Using existing auth tokens
4. **Data Exfiltration**: Unauthorized data access
5. **No Audit Trail**: Users unaware of operations

### Real-World Impact

- Supply chain attacks via compromised MCPs
- Malicious tools in public registries
- Data theft from connected services
- Compliance violations (GDPR/CCPA)
- IP and confidential information theft

### Defensive Measures

**For Users**:
- Audit connected MCP tools
- Monitor file creation
- Restrict MCP permissions
- Review Claude logs

**For Developers**:
- Implement sandboxing
- Use least privilege principle
- Add comprehensive logging
- Cryptographically sign tools

**For Claude Desktop**:
- Show tool usage transparency
- Require confirmation for sensitive ops
- Maintain immutable audit logs
- Implement tool isolation
- Add granular permissions

---

## Disclaimer

**⚠️ EDUCATIONAL PURPOSE ONLY**

This code is for:
- ✅ Security research
- ✅ Authorized penetration testing
- ✅ Understanding vulnerabilities
- ✅ Teaching secure development

This code is NOT for:
- ❌ Unauthorized testing
- ❌ Malicious data theft
- ❌ Privacy violations
- ❌ Illegal activity

**Legal**: Unauthorized computer access is illegal. Use only on systems you own or have explicit permission to test.

---

## References

- [MCP Protocol](https://modelcontextprotocol.io/)
- [Claude Desktop](https://github.com/anthropics/claude-desktop)
- [OWASP Security](https://owasp.org/)

---

## License

MIT License - See LICENSE file

---

**Version**: 1.0  
**Last Updated**: May 26, 2026
