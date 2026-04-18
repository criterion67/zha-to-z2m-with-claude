# Philips Hue Bulbs

Philips Hue bulbs pair directly to Zigbee coordinators (ZHA, Z2M, or the Hue Bridge). When moving from ZHA to Z2M, they need to be factory-reset first because they won't respond to pairing while associated with another coordinator.

---

## Factory Reset Methods

### Method 1: Hue Dimmer Switch (Easiest)
If you have a Hue Dimmer remote:
1. Hold the Dimmer within 10 cm of the bulb
2. Press and hold the **Setup button** (or I/O button on newer dimmers) for 10+ seconds
3. The bulb will blink to confirm the reset

### Method 2: Power Cycle
1. Turn the bulb on
2. Turn off for ~1 second (use the wall switch or lamp switch)
3. Turn back on
4. Repeat 5 times total
5. After the 5th cycle, the bulb blinks to confirm reset

### Method 3: Hue App (if bulb is on the Hue Bridge)
If the bulb is currently on a Hue Bridge rather than ZHA:
1. Open the Hue app → Settings → Lights
2. Find the bulb → Delete
3. The bulb resets during deletion

---

## Pairing to Z2M

1. Enable pairing mode in Z2M (Permit Join)
2. Factory-reset the bulb using one of the methods above
3. The bulb should appear in Z2M within 30–60 seconds
4. Set the Friendly Name to exactly match the original ZHA device name

---

## Entity Matching After Migration

Hue bulbs in Z2M expose the same entity types as in ZHA:
- `light.` — the main light entity (supports brightness, color temperature, color)
- `button.` — identify button
- `select.` — start-up behavior
- `number.` — start-up color temperature, start-up brightness level
- `update.` — firmware update entity
- `sensor.` — RSSI and LQI (disabled by default)

The `light.` entity is the one used in automations and dashboards.

---

## Notes

**Color bulbs (LCA series):** Support full RGB and color temperature. All capabilities carry over to Z2M identically.

**White ambiance bulbs (LTA/LCT series):** Support color temperature only. Work the same in Z2M.

**Start-up behavior:** If you had custom start-up behavior configured in ZHA (e.g., last state, always on), you'll need to re-configure this in Z2M after migration. Your Entity Registry spreadsheet tab captures the start-up behavior values if you had custom settings.

**Hue Lightguide (gradient bulb / decorative):** Pairs and works the same as other Hue bulbs. Uses the `LCZ002` model in ZHA.
