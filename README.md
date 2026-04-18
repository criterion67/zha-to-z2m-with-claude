# ZHA → Zigbee2MQTT Migration with Claude + HA MCP

**Use AI to audit, plan, and document your entire Zigbee migration — automatically, from your live Home Assistant instance.**

Migrating from ZHA to Zigbee2MQTT is one of the most complex operations in Home Assistant. Between hundreds of entities, custom names, Zigbee bindings, disabled entities, custom icons, area assignments, and the risk of breaking automations — it's a lot to track manually.

This guide shows you how to use **Claude (Anthropic's AI) with the Home Assistant MCP server** to do all the pre-migration auditing and planning for you. Claude queries your live HA instance directly, builds a full Excel reference spreadsheet, and writes a step-by-step migration plan tailored to your specific devices.

No manual data entry. No guesswork. No broken automations.

---

## What You Get

After running the prompt against your Home Assistant instance, Claude produces:

### 📊 Excel Spreadsheet (`zigbee_migration.xlsx`)
| Tab | Contents |
|-----|----------|
| **ZHA Devices** | Every device sorted by area — name, manufacturer, model, firmware, IEEE address, all entity IDs |
| **Entity Registry** | All entities with enabled/disabled status (color-coded), disabled-by reason, custom icons, area assignments, friendly names |
| **[Device] Switch** *(optional)* | Full parameter dump for advanced Zigbee switches like the Inovelli VZM31-SN — all configurable values captured live, ready to restore post-migration |

### 📄 Migration Plan Document (`ZHA_to_Z2M_Migration_Plan.docx`)
- Overview of your current setup and migration strategy
- Entity ID reuse strategy — how to preserve all automations without any changes
- Pre-migration checklist
- Recommended migration order (mains-powered routers first, bindings-dependent devices last)
- Per-device procedures for Hue bulbs, Aqara devices, IKEA remotes, smart plugs, and more
- Post-migration restoration steps (areas, icons, disabled entities)
- 19-item validation checklist

---

## How It Works

The key tool is the **Home Assistant MCP server**, which gives Claude read/write access to your HA instance. Once connected, you paste a single prompt and Claude:

1. Audits all your ZHA devices — every entity, area, IEEE address, firmware version
2. Checks for Zigbee bindings and flags devices that need special handling
3. Captures the full entity registry — including *disabled* entities that don't appear in states
4. Collects custom icons and area assignments for post-migration restoration
5. (Optional) Pulls all configurable parameters from advanced devices like Inovelli switches
6. Builds the Excel spreadsheet from all collected data
7. Writes the migration plan document
8. Delivers a summary report

> **The entity ID reuse strategy is the key insight:** By naming your Z2M devices to exactly match their ZHA device names, Home Assistant auto-generates identical entity IDs — preserving all automations, scripts, and dashboards with zero changes.

---

## Quick Start

### Step 1 — Install the HA MCP Server

Follow the setup guide: **[docs/prerequisites.md](docs/prerequisites.md)**

The recommended server is the [homeassistant-ai/ha-mcp](https://github.com/homeassistant-ai/ha-mcp) — 86 tools, Windows one-liner install, HA Add-on support, and Nabu Casa remote access.

**Windows one-liner:**
```powershell
irm https://raw.githubusercontent.com/homeassistant-ai/ha-mcp/main/scripts/install-windows.ps1 | iex
```

### Step 2 — Connect Claude to Your HA Instance

Open Claude desktop or Cowork, add the MCP server using your HA URL and a long-lived access token. See [docs/prerequisites.md](docs/prerequisites.md) for full instructions.

### Step 3 — Run the Prompt

Open a new Claude session with the HA MCP server connected and paste the prompt from **[PROMPT.md](PROMPT.md)**.

Edit the two bracketed sections at the top to describe your setup, then let Claude run. With a typical setup (50–100 devices), expect 10–20 minutes of tool calls.

### Step 4 — Use Your Outputs

Open the generated spreadsheet and migration plan. Migrate devices using the plan. After migrating each device, Claude can help restore parameters, icons, and disabled-entity settings from the spreadsheet.

---

## What Setup Does This Work With?

**This guide is written for the most common setup:**
- Single Zigbee coordinator (moving from one to another, or reconfiguring)
- ZHA currently active, moving to Zigbee2MQTT

**Advanced: dual coordinator simultaneous migration** (migrate devices one at a time while keeping both integrations running) is covered in [docs/dual-coordinator.md](docs/dual-coordinator.md). This requires two Zigbee dongles — for example, a Home Assistant SkyConnect (ZBT-1) for ZHA and an [Smlight SLZB-06M](https://smlight.tech/product/slzb-06m/) for Z2M.

---

## Device-Specific Guides

| Device Type | Guide |
|-------------|-------|
| Philips Hue bulbs | [devices/philips-hue.md](devices/philips-hue.md) |
| Aqara sensors, buttons, curtain motors | [devices/aqara.md](devices/aqara.md) |
| IKEA TRADFRI bulbs and remotes | [devices/ikea.md](devices/ikea.md) |
| Generic smart plugs | [devices/generic-plugs.md](devices/generic-plugs.md) |
| Inovelli VZM31-SN dimmer switch *(advanced)* | [devices/inovelli-vzm31-sn.md](devices/inovelli-vzm31-sn.md) |

---

## Tips for Best Results

- **Give Claude time** — don't interrupt it while it's running tool calls. It's querying your live HA instance and building files.
- **If it hits a rate limit**, say "continue from where you left off" and it will resume.
- **After migration**, start a new Claude session, upload your spreadsheet, and say: *"I've migrated [device name] to Z2M. Please restore all settings from the spreadsheet."* Claude will apply them automatically.
- **The `zha_event` audit matters** — remotes and dimmers stop working silently after leaving ZHA if automations aren't updated to Z2M device triggers.

---

## Repository Contents

```
├── README.md                   ← You are here
├── PROMPT.md                   ← The full Claude prompt to copy and paste
├── docs/
│   ├── prerequisites.md        ← HA MCP server setup (required first step)
│   ├── migration-guide.md      ← Step-by-step migration walkthrough
│   ├── dual-coordinator.md     ← Advanced: running ZHA + Z2M simultaneously
│   └── tips.md                 ← Pro tips and troubleshooting
├── devices/
│   ├── philips-hue.md
│   ├── aqara.md
│   ├── ikea.md
│   ├── generic-plugs.md
│   └── inovelli-vzm31-sn.md   ← Advanced switch with 42 configurable parameters
└── CONTRIBUTING.md
```

---

## Tools Used

- [Claude](https://claude.ai) by Anthropic (Sonnet or Opus recommended)
- [homeassistant-ai/ha-mcp](https://github.com/homeassistant-ai/ha-mcp) — Home Assistant MCP Server
- [Zigbee2MQTT](https://www.zigbee2mqtt.io/)
- [ZHA (Zigbee Home Automation)](https://www.home-assistant.io/integrations/zha/)

---

## Contributing

Found a device procedure that should be added? Improved the prompt for a specific device type? See [CONTRIBUTING.md](CONTRIBUTING.md).

---

*This guide was created with Claude + HA MCP and documents a real migration of 87 ZHA devices and 961 entities.*
