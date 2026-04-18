# Advanced: Dual Coordinator Migration

Running ZHA and Zigbee2MQTT simultaneously with two separate Zigbee coordinators lets you migrate devices one at a time while keeping your smart home fully functional. This is more complex to set up but eliminates the need for any "all at once" cutover.

This approach is what the original Claude prompt was designed around, but it's not required for a successful migration. Most users can do a sequential single-coordinator migration instead — see [migration-guide.md](migration-guide.md).

---

## When Dual Coordinator Makes Sense

- You have a large number of devices (50+) and want to minimize risk
- You have critical automations that can't be down for extended periods
- You already have a second coordinator available

---

## Hardware Requirements

You need two separate Zigbee coordinators, each managed by a different integration:

| Integration | Example Hardware |
|-------------|-----------------|
| ZHA (keeping for now) | Home Assistant SkyConnect (ZBT-1) |
| Zigbee2MQTT (migrating to) | Smlight SLZB-06M, Sonoff Zigbee 3.0 USB Dongle Plus, or similar |

The two coordinators must be on **non-overlapping Zigbee channels**.

---

## Zigbee Channel Separation

Zigbee uses channels 11–26. The three most commonly used "non-overlapping" channels are **11, 15, and 25** (or 11, 20, 26 in some recommendations). Your two coordinators must be on different channels from this list.

To check your current ZHA channel: Settings → Devices & Services → ZHA → Configure

To set your Z2M channel: In `configuration.yaml` for Z2M, add:
```yaml
advanced:
  channel: 25
```

> **Important:** WiFi (2.4 GHz) overlaps with some Zigbee channels. Channel 25 is generally the safest choice for Z2M if your WiFi uses channels 1, 6, or 11.

---

## Setup

1. Connect both coordinators to your HA host
2. Configure Z2M with the second coordinator's serial port
3. Set Z2M to a different channel than ZHA
4. Verify both ZHA and Z2M are running simultaneously in Settings → Integrations
5. Run the Claude prompt (it will audit ZHA and detect both networks)

---

## Migration Flow

With dual coordinators, you can migrate at your own pace:

1. Run the Claude audit prompt to capture all ZHA device data
2. Migrate low-risk devices first to test your process
3. Migrate remaining devices following your plan's priority order
4. ZHA and Z2M run in parallel throughout — your unmigrated ZHA devices stay fully functional
5. When all devices are migrated and verified, remove the ZHA integration

---

## Detecting Duplicate Devices

Occasionally a device may accidentally pair to both networks. If this happens:
- The device will appear in both ZHA and Z2M with the same IEEE address
- To remove from Z2M: use the Z2M frontend → device → Remove
- Or via MQTT: publish to `zigbee2mqtt/bridge/request/device/remove` with payload `{"id": "[device name]", "force": true}`

This is an edge case and doesn't happen in normal operation.
