# Smart Plugs and Outlets

Smart plugs are mains-powered and act as Zigbee routers — they extend your mesh for other devices. Migrate these early in your migration order.

---

## General Migration Procedure

1. Factory reset the plug (hold button 5–10 seconds, device blinks)
2. Enable pairing mode in Z2M
3. The plug pairs to Z2M
4. Set Friendly Name to match original ZHA device name
5. Verify the outlet turns on/off from HA

---

## Single Outlet Plugs

Most basic smart plugs expose:
- `switch.` — main switch entity
- `sensor.` — power, energy, voltage, current (where available)
- `select.` — start-up behavior

No special considerations. Follow the standard procedure.

---

## Dual Outlet / Multi-Channel Plugs

Plugs with two independently-controlled outlets appear as multiple switch entities. After migration:
- Verify both outlets appear in HA
- Check both switch entities are controllable
- Verify energy monitoring (if applicable) shows data for both channels

Some multi-channel plugs use a single device with multiple endpoint entities (e.g., `switch.device_name` and `switch.device_name_2`). In Z2M these will appear under a single device entry with multiple exposes.

---

## Power Monitoring Plugs

Plugs with energy monitoring expose additional sensors. In ZHA, some of these sensors are disabled by default (e.g., summation delivered, instantaneous demand). After migration to Z2M, check which sensors are enabled vs. disabled and adjust to match your preferences.

**Common energy monitoring entities:**
- `sensor.` power (watts)
- `sensor.` energy (kWh summation delivered)
- `sensor.` voltage
- `sensor.` current (amps)
- `sensor.` power factor
- `sensor.` AC frequency

---

## Start-Up Behavior

If you had specific start-up behavior configured (e.g., plug always turns on when power is restored), re-configure this in Z2M after migration:
- In Z2M frontend → device → Exposes → Start-up behavior
- Or via the `select.` entity in HA

---

## Switch-as-X Helpers

If you used a `switch_as_x` helper to represent a plug as a different entity type (e.g., a plug controlling a pet door represented as a `lock`), the helper must be updated after migration.

After the device migrates to Z2M with the same Friendly Name, the underlying entity ID should be identical — but verify the helper is pointing to the correct entity.

Settings → Helpers → [helper name] → Edit → verify entity reference.

---

## Notes

**Third Reality dual plugs:** Use a single device entry with two switch endpoints. Both channels appear in Z2M under the same device.

**Sonoff S31 / S40:** Popular Zigbee plugs that work well in Z2M. Factory reset with button hold.

**IKEA TRADFRI outlets:** See [ikea.md](ikea.md).

**Plug as Zigbee router:** Once migrated to Z2M, the plug immediately starts acting as a Z2M router, strengthening the mesh for end devices in its area.
