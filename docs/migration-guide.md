# Migration Guide — ZHA to Zigbee2MQTT

This guide covers the actual migration process after Claude has generated your spreadsheet and plan document. Read this alongside your personalized `ZHA_to_Z2M_Migration_Plan.docx`.

---

## Overview of the Migration Process

Moving from ZHA to Z2M involves re-pairing each Zigbee device to the Z2M coordinator. There is no automated migration tool that moves devices — each one must be factory-reset (or removed from ZHA and re-paired) individually.

The migration approach in this guide:

1. **Run the Claude prompt** to audit your ZHA setup and build your reference documents
2. **Install Zigbee2MQTT** and get it running (before migrating any devices)
3. **Migrate devices one at a time**, following your personalized plan
4. **Restore entity settings** after each device migrates (areas, icons, disabled status)
5. **Update automations** that use `zha_event` triggers
6. **Remove ZHA** once all devices are migrated

---

## Before You Start

### Take a Full HA Backup
Go to Settings → System → Backups → Create Backup. Do this right before starting migration and keep it somewhere safe.

### Install Zigbee2MQTT
Z2M can run as an HA Add-on (recommended), a Docker container, or standalone. The HA Add-on is the simplest path:
1. Settings → Add-ons → Add-on Store → search "Zigbee2MQTT"
2. Install, then configure with your coordinator details
3. Verify Z2M starts and the frontend loads at your HA address

### Configure Your Z2M Coordinator
In your Z2M configuration, set the serial port for your Zigbee coordinator. Common examples:
- USB dongle: `/dev/ttyUSB0` or `/dev/ttyACM0` (check with `ls /dev/serial/by-id/`)
- Network coordinator (e.g., Smlight SLZB-06M): `tcp://[IP address]:6638`

### Verify Zigbee Channels (Dual Coordinator Only)
If you're running ZHA and Z2M simultaneously with two different coordinators, they **must be on different Zigbee channels** that don't overlap with each other or with your WiFi. See [dual-coordinator.md](dual-coordinator.md).

---

## The Entity ID Reuse Strategy

This is the most important concept in the entire migration — understanding it will save you from having to update any automations, scripts, or dashboards.

**How entity IDs are generated in HA:**
- HA generates entity IDs from the device name and entity type
- Example: A device named "Bedroom Light Switch" gets entity IDs like `switch.bedroom_light_switch` and `light.bedroom_light_switch`

**How to preserve them:**
- When you add a device to Z2M, Z2M gives it a "Friendly Name"
- If you set the Z2M Friendly Name to **exactly match** the ZHA device name, HA will generate the same entity IDs
- All automations, scripts, and dashboards that reference those entity IDs will continue working without any changes

**Example:**
- ZHA device name: `Bedroom Ceiling Light`
- ZHA entity: `light.bedroom_ceiling_light`
- Z2M Friendly Name: `Bedroom Ceiling Light` ← must match exactly
- Z2M entity: `light.bedroom_ceiling_light` ← same entity ID, everything works

Your migration plan document includes a table of all your ZHA device names — use these as your Z2M Friendly Names.

---

## Migration Order

Follow the order in your migration plan document. The general priority is:

1. **Test device first** — pick one low-risk plug or bulb to verify your process works end-to-end
2. **Mains-powered router devices** (smart plugs, always-on switches) — these extend the Z2M mesh for the devices that follow
3. **Non-critical lights** — bedroom accent lights, decorative bulbs, etc.
4. **Critical lights** — main room lights, areas where a failure would be disruptive
5. **Sensors** (temperature, motion, door/window) — these are battery-powered and don't affect the mesh
6. **Remote controls and buttons** — these require updating `zha_event` automations
7. **Zigbee-bound switches/dimmers** — always last, after their bound devices and Z2M groups are set up

---

## Migrating Each Device

### Standard Procedure (Works for Most Devices)

1. **Open Z2M frontend** → enable pairing mode (the permit join button)
2. **Remove the device from ZHA**: Settings → Devices & Services → ZHA → [device] → Delete Device
3. **Factory reset the device** (see device-specific guides for methods)
4. **Pair to Z2M**: device should appear in Z2M frontend within 30–60 seconds
5. **Set the Friendly Name** in Z2M to exactly match the original ZHA device name
6. **Verify in HA**: the same entity IDs should now appear under the Zigbee2MQTT integration

### After Each Device Migrates

- Check the entity is working (state changes as expected)
- Assign the area in HA if it didn't auto-assign
- Note any custom icons that need to be re-applied (listed in your spreadsheet)
- Note any entities that were user-disabled and need to be re-disabled

---

## Post-Migration Tasks

After all devices are migrated:

### Re-Apply Custom Icons
Your spreadsheet's Entity Registry tab lists all entities that had custom icons in ZHA. Re-apply them in HA:
Settings → Devices & Services → [entity] → Edit → Icon

### Re-Disable User-Disabled Entities
Your spreadsheet flags which entities were user-disabled (orange rows). Re-disable them:
Settings → Devices & Services → [entity] → Disable

### Update `zha_event` Automations
Any automation using `zha_event` as a trigger must be updated to use the Z2M device trigger instead. Your migration plan includes a list of affected automations.

### Recreate Zigbee Groups and Bindings
If any of your devices used Zigbee bindings (switches bound to lights), recreate them in Z2M. Your migration plan includes instructions for this.

### Remove ZHA
Once all devices are migrated and verified:
1. Settings → Devices & Services → ZHA → Delete
2. The ZHA integration and coordinator can now be repurposed or removed

### Final Backup
Take another full HA backup after the migration is complete and everything is verified.

---

## Getting Help from Claude Post-Migration

After migrating a device, you can use Claude (with HA MCP connected) to restore settings automatically:

> "I've migrated my Bedroom Light Switch to Z2M. Please restore its area, custom icon, and any disabled entities from the migration spreadsheet."

Upload your `zigbee_migration.xlsx` and Claude will apply the settings.

For devices with many configurable parameters (like Inovelli switches), Claude can re-apply all parameters in one session:

> "The Bedroom Dimmer Switch is now in Z2M. Please restore all 42 parameters from the spreadsheet tab."
