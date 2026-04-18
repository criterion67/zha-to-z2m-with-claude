# IKEA TRADFRI Devices

IKEA makes a wide range of Zigbee-compatible smart home devices including bulbs, outlets, motion sensors, and remotes. Most work well in Z2M with full feature parity vs. ZHA.

---

## IKEA Bulbs (TRADFRI, Styrbar targets, etc.)

### Factory Reset
IKEA bulbs require 6 power cycles to factory reset:
1. Start with the bulb ON
2. Turn off and on 6 times, with ~1 second off each time
3. After the 6th cycle, the bulb blinks to confirm reset

### Pairing
After reset, the bulb enters pairing mode and should appear in Z2M within 30 seconds.

### Entities in Z2M
- `light.` — main light entity
- Supports brightness (all IKEA bulbs)
- Supports color temperature (warm-to-cool models)
- Color: only TRADFRI color bulbs

---

## TRADFRI Remotes and Dimmers

IKEA remotes and dimmers (5-button remote, Shortcut button, Rodret dimmer, Styrbar remote) use `zha_event` in ZHA automations. **These automations must be updated after migration.**

### Pairing
- Pair button is typically on the back inside the battery compartment
- Hold reset button for 5+ seconds to reset and enter pairing mode

### After Migration — Update Automations
In ZHA, button press events come as `zha_event` with a command (e.g., `toggle`, `move_to_level`). In Z2M, these become device-specific triggers.

To update automations:
1. Open the automation in HA
2. Remove the `zha_event` trigger
3. Add a new trigger: Device → select the remote from Z2M → select the action

Z2M device triggers for IKEA remotes expose the same logical actions (button pressed, held, released) as ZHA.

### TRADFRI Shortcut Button
Single-button remote. Sends a `toggle` action in ZHA and a similar device trigger in Z2M. Update any automations after migration.

### Rodret Dimmer / Somrig Shortcut
Two-button dimmers. Work similarly — update `zha_event` automations after migration.

---

## TRADFRI Outlets / Plugs

Smart plugs that act as Zigbee routers. Migrate these early to build your Z2M mesh.

### Pairing
Hold the button on the outlet for 5+ seconds to reset.

### Entities in Z2M
- `switch.` — main switch
- Some models expose `sensor.` for power monitoring

---

## TRADFRI Motion Sensors

### Pairing
Small reset button — hold 5+ seconds to reset.

### Entities in Z2M
- `binary_sensor.` — occupancy/motion
- `sensor.` — battery, illuminance (where applicable)

### Automation Updates
ZHA motion sensor events work the same in Z2M via state changes on the `binary_sensor.` entity. Most motion-based automations don't need updating (they trigger on entity state change, not on ZHA-specific events).

---

## Notes

**IKEA DIRIGERA hub:** If your IKEA devices are paired to a DIRIGERA hub rather than ZHA, they need to be removed from DIRIGERA first, then factory-reset, then paired to Z2M.

**Firmware updates:** IKEA offers firmware updates for some devices. Z2M can push these updates via the `update.` entity — check the Z2M device page after migration.

**TRADFRI signal repeaters:** These are Zigbee routers — migrate early to extend your mesh.
