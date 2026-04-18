# Prerequisites — Setting Up the Home Assistant MCP Server

Before you can use the Claude prompt to audit your ZHA setup, you need the **Home Assistant MCP server** installed and connected to Claude. This gives Claude read/write access to your live HA instance.

This guide covers the recommended server: **[homeassistant-ai/ha-mcp](https://github.com/homeassistant-ai/ha-mcp)** — an unofficial but well-maintained server with 86 tools, simple installation, and support for Nabu Casa remote access.

---

## What You Need

- Home Assistant running (any recent version)
- A **long-lived access token** from your HA profile
- Claude desktop app or Cowork (the prompt also works in Claude.ai with MCP support)
- Windows, macOS, or Linux on your computer

---

## Step 1 — Create a Long-Lived Access Token in Home Assistant

1. Open Home Assistant in your browser
2. Click your profile icon (bottom-left corner)
3. Scroll down to **Long-lived access tokens**
4. Click **Create Token**
5. Give it a name (e.g., "Claude MCP")
6. Copy the token — you won't see it again

---

## Step 2 — Install the HA MCP Server

### Option A: Windows (Recommended — one-liner)

Open PowerShell and run:

```powershell
irm https://raw.githubusercontent.com/homeassistant-ai/ha-mcp/main/scripts/install-windows.ps1 | iex
```

The installer will:
- Download and configure the ha-mcp server
- Prompt you for your HA URL and access token
- Configure it for Claude desktop automatically

### Option B: HA Add-on (runs on your HA host)

If you'd prefer to run the MCP server directly on your HA instance:

1. In Home Assistant, go to **Settings → Add-ons → Add-on Store**
2. Search for **ha-mcp** or add the custom repository: `https://github.com/homeassistant-ai/ha-mcp`
3. Install and start the add-on
4. Configure your access token in the add-on settings

### Option C: Manual (macOS / Linux)

See the full installation guide at [github.com/homeassistant-ai/ha-mcp](https://github.com/homeassistant-ai/ha-mcp).

---

## Step 3 — Connect Claude to the MCP Server

### In Claude Desktop or Cowork

1. Open **Claude Settings** → **MCP Servers** (or Developer → MCP Servers)
2. Add a new server with these details:
   - **Name**: Home Assistant
   - **URL**: `http://localhost:PORT` (check your ha-mcp config for the port, typically 3000)
   - Or use the HA URL directly if using the Add-on
3. Save and restart Claude if prompted

### Nabu Casa / Remote Access

If your HA is remote (not on the same network), ha-mcp supports Nabu Casa's Webhook Proxy for remote access. See the [ha-mcp documentation](https://github.com/homeassistant-ai/ha-mcp) for setup instructions.

---

## Step 4 — Verify the Connection

In a new Claude session, type:

> "What devices does my Home Assistant have?"

If Claude responds with actual device information from your HA instance, the connection is working.

---

## Step 5 — Run the Migration Prompt

Once connected, open [PROMPT.md](../PROMPT.md) and copy the full prompt into Claude. Fill in the two bracketed sections at the top, then let Claude run through all 8 tasks.

---

## Troubleshooting

**Claude says it doesn't have HA access**
Make sure the MCP server is running and the connection is active in Claude settings. Try restarting Claude after adding the server.

**Authentication errors**
Double-check your long-lived access token — make sure you copied it correctly and haven't accidentally included whitespace.

**MCP server connection refused**
Check that the ha-mcp process is actually running. On Windows, check Task Manager or run the installer again.

**Nabu Casa / remote HA**
If your HA is only accessible via Nabu Casa, you'll need to use the Webhook Proxy method described in the ha-mcp documentation.
