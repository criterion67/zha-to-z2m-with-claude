# The Claude Prompt

Copy and paste this entire prompt into a Claude session that has the **Home Assistant MCP server connected**.

**Works best with Claude Sonnet or Claude Opus.**

Before using this prompt, complete the setup in [docs/prerequisites.md](docs/prerequisites.md).

---

## How to Customize This Prompt

At the top of the prompt, there are two sections in brackets — fill those in before running:

- `[YOUR SETUP]` — describe your coordinator hardware (e.g., "Home Assistant SkyConnect ZBT-1 USB dongle for ZHA, Smlight SLZB-06M network coordinator for Z2M")
- `[INOVELLI / ADVANCED DEVICES]` — list any Zigbee switches with configurable parameters, or write "none" to skip Task 4

Everything else works automatically against your live HA instance — no manual exports needed.

---

## The Prompt

> Copy everything between the horizontal lines below.

---

```
I need your help planning and documenting a complete migration from ZHA (Zigbee Home Automation) 
to Zigbee2MQTT (Z2M) in Home Assistant.

MY SETUP: [describe your coordinator hardware here — e.g., "single Smlight SLZB-06M coordinator, 
currently running ZHA, migrating to Z2M on the same hardware" OR "dual coordinators: 
Home Assistant SkyConnect ZBT-1 for ZHA, Smlight SLZB-06M for Z2M running simultaneously"]

ADVANCED DEVICES WITH CONFIGURABLE PARAMETERS (for Task 4): [list any Zigbee dimmers/switches 
with many configurable parameters, e.g., "Two Inovelli VZM31-SN dimmer switches" — or write 
"none" to skip Task 4]

Please complete ALL of the following tasks in order. Do not stop after any single task — work 
through all of them and give me a complete picture before asking any questions.

---

TASK 1 — AUDIT ALL ZHA DEVICES

Use the Home Assistant MCP tools to retrieve every device registered under the ZHA integration. 
For each device, collect:
- Device name (friendly name)
- Manufacturer and model
- Firmware / software version
- Area assignment
- IEEE address
- Every entity ID and entity name associated with the device
- Integration type (confirm ZHA)

If the result is paginated, retrieve ALL pages until you have every device. Save all device data 
for use in the spreadsheet you will build in Task 5.

---

TASK 2 — CHECK FOR ZIGBEE BINDINGS

For any Zigbee switches, dimmers, or remotes in ZHA:
- Identify what devices or groups they are bound to
- Note that Zigbee bindings are stored in device firmware and will be lost when the device is 
  removed from ZHA
- Flag these devices in your report as requiring binding recreation in Z2M after migration
- Recommend creating Z2M Groups for the binding targets before migrating the switch

For any Inovelli VZM31-SN switches found:
- Note they must be migrated LAST (after all bound bulbs are in Z2M and groups are set up)
- Confirm whether Smart Bulb Mode is enabled (switch.DEVICENAME_smart_bulb_mode = on)
- Note the binding targets (which lights/groups the switch controls)

---

TASK 3 — IDENTIFY AUTOMATIONS USING zha_event TRIGGERS

Search for any automations in Home Assistant that use zha_event as a trigger. These automations 
will break when their associated device is removed from ZHA, because zha_event is ZHA-specific.

For each automation found:
- Report the automation name and which device it references
- Flag it as requiring update to Z2M device trigger after the device is migrated
- Note that Z2M uses "device trigger" automations instead of zha_event

---

TASK 4 — CAPTURE ADVANCED DEVICE PARAMETERS (if applicable)

[Skip this task if you wrote "none" in the ADVANCED DEVICES section above]

If you have Inovelli VZM31-SN switches (or similar Zigbee devices with many configurable 
parameters), retrieve the current value of EVERY configurable parameter entity for each device. 
These include:

Switch entities (on/off): smart_bulb_mode, aux_switch_scenes, binding_off_to_on_sync_level, 
disable_config_2x_tap_to_clear_notifications, disable_relay_click_in_on_off_mode, 
double_tap_down_enabled, double_tap_up_enabled, firmware_progress_led, invert_switch, 
local_protection, only_1_led_mode

Number entities: automatic_switch_shutoff_timer, button_delay, default_all_led_off_color, 
default_all_led_off_intensity, default_all_led_on_color, default_all_led_on_intensity, 
double_tap_down_level, double_tap_up_level, load_level_indicator_timeout, 
local_dimming_down_speed, local_dimming_up_speed, local_ramp_rate_off_to_on, 
local_ramp_rate_on_to_off, maximum_load_dimming_level, minimum_load_dimming_level, 
on_level, on_off_transition_time, remote_dimming_down_speed, remote_dimming_up_speed, 
remote_ramp_rate_off_to_on, remote_ramp_rate_on_to_off, start_up_current_level

Select entities: led_scaling_mode, non_neutral_output, output_mode, start_up_behavior, 
switch_type

For each device, retrieve each entity individually using ha_get_state (do not try to batch 
them as a JSON array — call ha_get_state once per entity). Save all values for the spreadsheet.

---

TASK 5 — CAPTURE ENTITY REGISTRY METADATA

I need to know the following for every ZHA entity, so I can restore settings after migration:

A) ENABLED/DISABLED STATUS
Use ha_eval_template with the integration_entities('zha') function to get all currently 
ENABLED ZHA entity IDs. Compare this list against the full entity list from Task 1. Any 
entity in the Task 1 list that does NOT appear in the integration_entities() result is disabled.

Categorize disabled entities as:
- "disabled by integration" — RSSI sensors, LQI sensors, identify buttons, firmware update 
  entities (standard ZHA behavior, will replicate automatically in Z2M)
- "likely user-disabled" — any other disabled entity (climate, lock, switch, sensor domains 
  that don't match the diagnostic patterns above)

B) CUSTOM ICONS
Use ha_eval_template to retrieve all ZHA entities that have a non-null icon attribute:

Template:
{%- set ns = namespace(results=[]) -%}
{%- for entity in integration_entities('zha') -%}
  {%- set ico = state_attr(entity, 'icon') -%}
  {%- if ico -%}
    {%- set ns.results = ns.results + [entity ~ '|' ~ ico] -%}
  {%- endif -%}
{%- endfor -%}
{{ ns.results | tojson }}

C) ENTITY AREA ASSIGNMENTS
Use ha_eval_template with the area_id() function:

Template:
{%- set ns = namespace(results=[]) -%}
{%- for entity in integration_entities('zha') -%}
  {%- set ar = area_id(entity) -%}
  {%- if ar -%}
    {%- set ns.results = ns.results + [entity ~ '|' ~ ar] -%}
  {%- endif -%}
{%- endfor -%}
{{ ns.results | tojson }}

D) FRIENDLY NAMES
Use ha_eval_template to get all enabled ZHA entity friendly names:

Template:
{%- set ns = namespace(results=[]) -%}
{%- for entity in integration_entities('zha') -%}
  {%- set fn = state_attr(entity, 'friendly_name') -%}
  {%- if fn -%}
    {%- set ns.results = ns.results + [entity ~ '|' ~ fn] -%}
  {%- endif -%}
{%- endfor -%}
{{ ns.results | tojson }}

---

TASK 6 — BUILD THE MIGRATION SPREADSHEET

Using all data collected in Tasks 1–5, build an Excel file (.xlsx) with the following tabs:

TAB 1 — "ZHA Devices"
One row per entity. Columns: #, Device Name, Manufacturer, Model, Firmware Version, Area, 
IEEE Address, Entity ID, Entity Name / Domain. Sort by area then device name. Include all 
devices and all their entities.

TAB 2 — "Entity Registry"
One row per entity. Columns: #, Entity ID, Friendly Name, Domain, Category, Status 
(enabled/disabled), Disabled By (integration/user/blank), Area, Custom Icon, Device Name.
Color-code: red for integration-disabled rows, orange for user-disabled rows, yellow for 
rows with custom icons. Include a summary row at the bottom with counts.

TAB 3+ — One tab per advanced device found in Task 4 (if applicable)
Columns: #, Parameter Name, Entity ID, Type (Switch/Number/Select), Current Value, Notes.
Group rows by type (Switches section, Numbers section, Selects section). Color-code the 
Current Value cell green for on/enabled, orange for off/disabled.

Save the file as: zigbee_migration.xlsx
Use professional formatting (Arial font, frozen header rows, column widths sized to content).
Run formula recalculation after saving to verify zero errors.

---

TASK 7 — WRITE THE MIGRATION PLAN DOCUMENT

Write a comprehensive migration plan as a Word document (.docx) covering:

1. OVERVIEW — Current setup summary (device counts, coordinator type, migration strategy)

2. ENTITY ID REUSE STRATEGY — Explain that setting Z2M device Friendly Names to exactly 
   match ZHA device names preserves existing automation/script/dashboard entity ID references

3. PRE-MIGRATION CHECKLIST — HA backup, Z2M installation/configuration, Zigbee channel 
   verification (if using dual coordinators, channels must not overlap), spreadsheet review, 
   zha_event automation audit, ZHA group documentation

4. MIGRATION ORDER — Prioritized device list. Always migrate in this order:
   - Test/low-risk devices first
   - Mains-powered router devices (plugs, switches) before battery end devices
   - Non-critical lights before critical ones
   - Remote controls and buttons near the end (require automation updates)
   - Any bound Zigbee switches/dimmers LAST (after all bound devices are in Z2M)

5. PER-DEVICE PROCEDURES — Standard steps for all devices plus specific notes for:
   - Philips Hue bulbs (factory reset via Hue Dimmer, or 5x power cycle)
   - Aqara devices (pairing button hold duration, curtain motors preserve calibration)
   - IKEA TRADFRI/Shortcut dimmers (zha_event automation update required)
   - Dual-outlet or multi-channel plug devices (verify both channels)
   - Any advanced Zigbee switches found in Task 4 (full procedure: migrate bound devices 
     first, create Z2M groups, migrate switch, restore parameters, rebind, test)
   - Any switch_as_x helpers wrapping ZHA entities (update to Z2M entity after migration)

6. POST-MIGRATION RESTORATION — 
   - Assign areas in HA
   - Re-apply custom icons (list all from spreadsheet)
   - Re-disable user-disabled entities (list all from spreadsheet)
   - Restore advanced device parameters (note that Claude can do this automatically)
   - Recreate Z2M groups and bindings
   - Update zha_event automations to Z2M device triggers
   - Update switch_as_x helpers

7. VALIDATION CHECKLIST — Printable checklist with checkboxes for final verification steps 
   including: all devices available in Z2M, areas assigned, entities restored, bindings 
   verified, automations tested, ZHA integration removal, final HA backup

Save the document as: ZHA_to_Z2M_Migration_Plan.docx

---

TASK 8 — SUMMARY REPORT

After completing all tasks, provide a plain-text summary covering:
- Total ZHA devices found and any issues identified
- Count of user-disabled entities that need manual re-disabling after migration
- Count of entities with custom icons that need re-applying
- Count of advanced devices found and total parameters captured (if applicable)
- Any devices with Zigbee bindings that require special handling
- Any automations using zha_event that will need updating
- The recommended migration order based on your device list
- Confirmation that the spreadsheet and plan document were saved

---

IMPORTANT NOTES FOR CLAUDE:
- Use the HA MCP tools throughout — do not ask me to export data manually
- Do not try to pass JSON arrays as entity_id to ha_get_state — call it once per entity
- Use ha_eval_template for bulk entity data (icons, areas, friendly names, enabled status)
- If paginated results appear, fetch all pages before proceeding
- Build the spreadsheet using openpyxl (Python), not pandas, so formatting is preserved
- Save all output files to the outputs folder
- Do not stop between tasks — complete everything before asking questions
```

---

## After Running the Prompt

Once Claude finishes all 8 tasks, you'll have:
- `zigbee_migration.xlsx` — your complete reference spreadsheet
- `ZHA_to_Z2M_Migration_Plan.docx` — your personalized migration plan

Follow the plan document for migration. After migrating each device, you can start a fresh Claude session and say:

> *"I've migrated [device name] to Z2M with friendly name '[exact ZHA name]'. Please restore all settings from this spreadsheet."*

Upload the spreadsheet and Claude will apply area assignments, custom icons, and disabled-entity settings — and for advanced devices like the Inovelli VZM31-SN, it can re-apply all configurable parameters automatically.

---

## Troubleshooting

**Claude stopped partway through**
Say "continue from where you left off" — Claude will resume without restarting.

**Rate limit hit**
Wait a few minutes, then say "continue from where you left off."

**"Entity not found" errors on ha_get_state**
This happens if you try to pass a list of entity IDs. The prompt already instructs Claude to call once per entity — if it still happens, remind Claude: "Call ha_get_state once per entity, not with a list."

**ha_eval_template returns empty results**
This can happen if ZHA is named differently in your HA instance. Try: `integration_entities('zha')` — if that returns nothing, check what your ZHA integration entry is called in Settings → Devices & Services.
