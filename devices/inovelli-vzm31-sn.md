# Inovelli VZM31-SN Zigbee 2-in-1 Dimmer Switch

> **Advanced device guide.** This procedure is specific to the Inovelli VZM31-SN and similar Zigbee switches with many configurable parameters and Zigbee bindings. Most users won't have this device — see the main [migration guide](../docs/migration-guide.md) for standard procedures.

---

## Why This Switch Needs Special Handling

The Inovelli VZM31-SN has:
1. **42 configurable parameters** (dimming speeds, ramp rates, LED colors/intensity, Smart Bulb Mode, etc.) that all need to be captured before migration and restored after
2. **Zigbee bindings** — the switch may be bound directly to light devices or groups at the Zigbee level, bypassing HA entirely. These bindings are stored in the switch's firmware and are lost when it leaves ZHA. They must be recreated in Z2M after migration.
3. **Smart Bulb Mode** — if enabled, the switch doesn't cut power to connected bulbs (lets the bulbs handle dimming). Z2M supports this mode but it must be re-enabled after migration.

**This switch must be migrated last** — after all its bound bulbs are already in Z2M and Z2M groups are set up.

---

## Step 1 — Capture All Parameters (Before Migration)

Run the full Claude prompt with this switch listed in the "advanced devices" section. Claude will capture all 42 parameter values and save them to your migration spreadsheet.

If you want to capture just the parameters without the full audit, ask Claude:

> "Using the HA MCP tools, retrieve the current value of every entity for my Inovelli VZM31-SN device named '[device name]' in Home Assistant and save the results to a spreadsheet."

**Parameters captured include:**

*Switch entities (on/off):*
- Smart Bulb Mode
- Aux Switch Scenes
- Invert Switch
- Double Tap Up/Down Enabled
- Local Protection
- Disable Relay Click
- Only 1 LED Mode
- Firmware Progress LED
- and more

*Number entities:*
- Dimming speeds (local and remote, up and down)
- Ramp rates (local and remote)
- LED colors and intensity (on and off states)
- Double tap target levels
- Button delay
- Min/max dimming levels
- On level, start-up level
- On/off transition time

*Select entities:*
- Switch type (single pole, multi-way with neutral, etc.)
- Output mode
- Start-up behavior
- LED scaling mode
- Non-neutral output (if applicable)
- Dimming mode

---

## Step 2 — Migrate Bound Bulbs First

Identify all bulbs that are Zigbee-bound to this switch (your migration plan will list them). Migrate every one of those bulbs to Z2M **before** touching the switch.

After each bulb migrates to Z2M, create a **Z2M Group** containing those bulbs:
1. Z2M Frontend → Groups → Add Group
2. Name the group something logical (e.g., "Bedroom Downlights")
3. Add all the bound bulbs to the group

The switch will bind to this Z2M group after migration.

---

## Step 3 — Migrate the Switch

1. Remove from ZHA: Settings → Devices & Services → ZHA → [switch device] → Delete Device
2. Factory reset the VZM31-SN: press the config button 3 times rapidly (refer to Inovelli documentation for your firmware version)
3. Enable pairing mode in Z2M
4. The switch should pair to Z2M
5. Set the Friendly Name to exactly match the original ZHA device name

---

## Step 4 — Restore Parameters

After migration, ask Claude to restore all parameters from your spreadsheet:

> "I've migrated the [Bedroom/Bathroom] Dimmer Switch to Z2M. Please restore all parameters from the '[Tab Name]' tab of my migration spreadsheet."

Upload your `zigbee_migration.xlsx` and Claude will call `ha_call_service` for each entity to restore the saved values. This is much faster than clicking through the HA UI for 42+ parameters.

---

## Step 5 — Recreate Zigbee Bindings

After parameters are restored, recreate the Zigbee bindings between the switch and its Z2M group:

In Z2M Frontend:
1. Go to the switch device → Bind tab
2. Bind the switch to the Z2M group you created in Step 2
3. Bind `genOnOff` and `genLevelCtrl` clusters

Or ask Claude to do this:

> "Please recreate the Zigbee bindings for my [device name] switch in Z2M. Bind it to the Z2M group '[group name]' for genOnOff and genLevelCtrl clusters."

---

## Step 6 — Verify

1. Test Smart Bulb Mode — the switch should dim the lights without cutting power
2. Test double-tap behavior (double-tap up = full brightness, double-tap down = off)
3. Test the aux switch if applicable
4. Verify LED indicator behavior
5. Test physical dimming (ramp up and down)

---

## ZHA Naming Quirk — Bathroom Switch Button Delay

> **Note for owners of multiple VZM31-SN switches:** In ZHA, there's a known naming inconsistency where the second switch's `button_delay` entity ID maps to "Local Dimming Up Speed" in the HA UI, and the actual button delay is at `button_delay_2`. This is a ZHA quirk — Z2M names these entities consistently. Your spreadsheet will reflect your actual live values regardless.

---

## Resources

- [Inovelli VZM31-SN Product Page](https://inovelli.com/products/blue-series-smart-2-1-switch-matter-zigbee)
- [Inovelli Community Forum](https://community.inovelli.com/)
- [Z2M device support page for VZM31-SN](https://www.zigbee2mqtt.io/devices/VZM31-SN.html)
