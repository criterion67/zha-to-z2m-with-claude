# Tips, Tricks & Troubleshooting

---

## Getting the Best Results from Claude

**Give it time.** With 50+ devices, Claude makes hundreds of individual tool calls to query your HA instance. Let it run — don't interrupt. Typical runtime is 10–20 minutes for a large setup.

**If it hits a rate limit**, say "continue from where you left off" and it will pick up exactly where it stopped without re-doing completed work.

**Use Sonnet or Opus.** Haiku is faster but less reliable for complex multi-step tasks involving lots of tool calls and data synthesis. Claude Sonnet hits the right balance of speed and reliability.

**Don't edit your ZHA setup mid-run.** Avoid adding, removing, or renaming devices in HA while Claude is auditing. The data it collects needs to be consistent.

---

## Entity ID Strategy — Common Mistakes

**Capitalization matters.** If your ZHA device is named "Bedroom Ceiling Light", your Z2M Friendly Name must be exactly "Bedroom Ceiling Light" — not "bedroom ceiling light" or "Bedroom ceiling light".

**Spaces and special characters.** HA converts these to underscores in the entity ID. The Friendly Name in Z2M should use the same spacing/capitalization as the ZHA device name — HA handles the conversion to entity ID format.

**Duplicate entity IDs.** If Z2M generates a new entity ID that collides with an existing one (from another integration), HA appends `_2`, `_3`, etc. This shouldn't happen if you follow the naming strategy, but check for it if entity IDs look wrong.

---

## Zigbee Mesh Considerations

**Migrate routers first.** Zigbee devices are either "routers" (mains-powered, extend the mesh) or "end devices" (usually battery-powered, connect through routers). Migrating your router devices to Z2M first builds the Z2M mesh before end devices need to join it.

**Smart plugs make great routers.** If you have smart plugs, migrate them early — they act as Zigbee routers and strengthen your Z2M network for the devices that follow.

**Bulbs as routers.** Most smart bulbs act as Zigbee routers. Migrating lights before sensors generally builds a more robust mesh.

---

## `zha_event` Automations

ZHA-specific device triggers (`zha_event`) won't fire after a device leaves ZHA. Your migration plan identifies which automations need updating.

**How to find them manually:**
Settings → Automations → search for "zha_event" in the filter

**How to update them:**
Edit the automation, remove the `zha_event` trigger, add a new trigger using Device → select your device from Z2M → choose the action.

Z2M device triggers behave identically to ZHA device triggers for most remotes and buttons.

---

## Switch-as-X Helpers

If you use `switch_as_x` helpers that wrap ZHA switch entities (for example, wrapping a smart plug as a lock or cover), the helper will break when the underlying entity moves to Z2M because it's linked to the ZHA entity ID.

**To fix:** After the device migrates to Z2M, edit the helper to point to the new Z2M entity. The entity ID should be the same if you followed the naming strategy — but verify it.

Settings → Helpers → [your helper] → Edit → update the entity reference

---

## Devices That Need Special Handling

**Philips Hue bulbs:** Need to be factory-reset before re-pairing. The easiest method is using a Hue Dimmer switch (5 button presses hold). Alternative: 5x power cycle (on/off at the wall, ~1 second off each time). See [devices/philips-hue.md](../devices/philips-hue.md).

**Aqara curtain motors:** Can be re-paired without losing calibration in most cases. See [devices/aqara.md](../devices/aqara.md).

**IKEA remotes/dimmers:** These use `zha_event` — make sure to update automations after migration. See [devices/ikea.md](../devices/ikea.md).

**Devices with Zigbee bindings:** Any switch bound directly to a light (at the Zigbee level, not just an HA automation) must be migrated last — after its target devices are in Z2M and Z2M groups are created. See [devices/inovelli-vzm31-sn.md](../devices/inovelli-vzm31-sn.md) for an example.

---

## Restoring Parameters with Claude

After migrating a device with many configurable parameters (like an Inovelli switch), start a new Claude session:

1. Upload your `zigbee_migration.xlsx`
2. Say: *"I've migrated [device name] to Z2M with friendly name '[name]'. Please restore all parameters from the [Tab Name] tab of this spreadsheet."*
3. Claude will call `ha_call_service` for each parameter entity to restore the saved values

This is much faster than clicking through the HA UI for 40+ parameters.

---

## After the Migration

**Don't remove ZHA immediately.** Give yourself a day or two with Z2M running to verify everything works before removing ZHA. Keep your HA backup handy.

**Check your Zigbee mesh health.** Z2M's frontend has a network map view — use it to verify devices are routing through expected paths.

**Take a final backup.** After the migration is complete and verified, take a full HA backup as your new baseline.
