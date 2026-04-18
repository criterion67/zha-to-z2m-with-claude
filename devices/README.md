# Device-Specific Migration Guides

This folder contains migration notes for specific device types. Most devices follow the standard procedure in [docs/migration-guide.md](../docs/migration-guide.md) — these guides cover devices with special requirements.

| Device Type | Guide | Key Considerations |
|-------------|-------|--------------------|
| Philips Hue bulbs | [philips-hue.md](philips-hue.md) | Must be factory-reset before re-pairing |
| Aqara sensors, buttons, curtain motors | [aqara.md](aqara.md) | Curtain motors can preserve calibration |
| IKEA TRADFRI bulbs and remotes | [ikea.md](ikea.md) | 6 power cycles to reset; remotes need automation updates |
| Generic smart plugs | [generic-plugs.md](generic-plugs.md) | Migrate early — they act as Zigbee routers |
| Inovelli VZM31-SN dimmer switch | [inovelli-vzm31-sn.md](inovelli-vzm31-sn.md) | 42 parameters to capture; Zigbee bindings; migrate last |

Have a device type not listed here? [Contributions welcome](../CONTRIBUTING.md).
