# Aqara Devices

Aqara makes a wide range of Zigbee sensors, buttons, and motorized devices. Most are battery-powered end devices. The key exception is motorized devices (curtain motors, roller shades) which require special handling to preserve calibration.

---

## General Aqara Pairing

Most Aqara devices have a small reset/pair button. The procedure is:
1. Press and hold for 5+ seconds until the LED blinks rapidly (factory reset)
2. Then the device enters pairing mode automatically
3. Alternatively, a short press (1 second) often enters pairing mode without resetting

Device-specific instructions are printed inside the battery compartment of most Aqara sensors.

---

## Temperature / Humidity / Pressure Sensors (WSDCGQ11LM, etc.)

**Pairing:**
1. Open battery cover, press the small button on the back
2. Hold for 5 seconds to reset, or 1 second to pair

**Entities in Z2M:**
- `sensor.` — temperature, humidity, pressure (where applicable), battery
- `sensor.` — RSSI, LQI (disabled by default)

No automation updates needed — these are passive sensors, not triggers.

---

## Door / Window Sensors (MCCGQ11LM, etc.)

**Pairing:**
1. Small button on the back — hold for 5 seconds to reset

**Entities in Z2M:**
- `binary_sensor.` — contact (open/closed)
- `sensor.` — battery
- `sensor.` — device temperature (on some models)

If you have automations triggered by door/window sensors, verify the entity IDs match after migration (they should, if you used the naming strategy).

---

## Wireless Mini Switches / Cube Controllers

These use `zha_event` in ZHA automations. After migration, update automations to use Z2M device triggers.

**Pairing:**
- Small button hold for reset/pair

**After migration:**
- Edit any automations that trigger on button press events
- Replace `zha_event` trigger with Device trigger in Z2M

---

## Curtain Motors (ZNCLDJ11LM / ZNCLDJ12LM / CM-M01)

Curtain motors are the trickiest Aqara devices to migrate because they store calibration data (motor limits, direction). However, the calibration is stored in the motor's firmware — **not** in ZHA — so re-pairing to Z2M in most cases **preserves calibration**.

### Migration Procedure
1. Do NOT factory-reset the curtain motor
2. Remove from ZHA: Settings → Devices & Services → ZHA → [motor device] → Delete Device
3. Immediately put the motor in pairing mode (press the small button once — do not hold)
4. The motor pairs to Z2M with calibration intact
5. Verify the motor opens/closes to the correct limits

### If Calibration is Lost
If the motor loses calibration during pairing:
1. In Z2M, go to the device → Exposes tab
2. Use the calibration controls to reset motor limits
3. Fully open → set upper limit → fully close → set lower limit

### Entities in Z2M
- `cover.` — main cover entity (open/close/set position)
- `sensor.` — battery
- `binary_sensor.` — running state (on some models)

---

## Notes

**All Aqara sensors** are Zigbee end devices (battery-powered). They connect through router devices. Make sure your Z2M mesh has enough router coverage in the areas where these sensors are before migrating them.

**Aqara E1 series** (newer models) behave the same as older models for pairing purposes.
