# Zendure Fr0gg3r Smart Export

Price-aware evening/vacation export control for Zendure batteries in Home Assistant, built as a companion package on top of [Gielz1986's Zendure-HA-zenSDK](https://github.com/Gielz1986/Zendure-HA-zenSDK) integration.

Automates grid export during peak Nordpool price windows, with configurable SOC reserve, vacation mode, aircon-load blocking, arm/disarm notifications, and a full dashboard — including a fix for combined-SOC accuracy on multi-unit setups.

**Author:** Fr0gg3r1986

---

## What this is (and isn't)

This is **not** a replacement for Gielz1986's integration — it's an add-on package that sits alongside it. It never edits or modifies his files; it only reads and controls entities he creates, by entity ID. This means:

- Updating Gielz1986's integration never breaks this package.
- Updating this package never touches his files.
- You need his integration installed and working first — this package has nothing to automate without it.

## Features

- **Price-based export control** — automatically arms/disarms grid export based on Nordpool's cheapest/most expensive price windows for the day.
- **Configurable SOC reserve** — set a minimum battery reserve you always want to keep, e.g. to get through the night.
- **Vacation mode** — auto-arms export daily without requiring manual confirmation, for when you're away.
- **Aircon-load reserve blocking** — optionally pause export if any of up to 3 rooms are running their AC, so the battery isn't drained by price-chasing while cooling/heating needs power.
- **Arm/disarm prompts** — daily notification with action buttons to confirm or postpone export, plus a "custom start time tonight" override.
- **Force-stop button** — one tap to immediately stop export and disarm, with confirmation.
- **Full dashboard** — Live view, Custom Export controls, Settings, Health, History, Proxy Details (for multi-unit setups), and Releasenotes.
- **Multi-unit SOC fix** — see [Known issue](#known-issue--combined-soc-not-capacity-weighted-on-multi-unit-setups) below.

## Requirements

| Dependency | Required? | Notes |
|---|---|---|
| [Gielz1986's Zendure-HA-zenSDK](https://github.com/Gielz1986/Zendure-HA-zenSDK) | **Always** | Base integration. Tested against the Global (EN) Integration, v20260716. |
| [gast777's Zendure-zenSDK-proxy](https://github.com/gast777/Zendure-zenSDK-proxy) | **Only for multi-unit setups** | Needed if you're running 2+ physical Zendure hubs behind Gielz1986's integration. Single-unit setups can skip this entirely. |
| Home Assistant `packages` support | **Always** | See install steps below. |
| A Nordpool sensor (HACS) | **Always** | For price-window detection. |

## Installation

1. Make sure Gielz1986's integration is installed and working first.
2. If you're running multiple Zendure units, set up gast777's proxy first too.
3. Copy `zendure_fr0gg3r_smart_export.yaml` into your `packages/` folder.
4. Make sure your `configuration.yaml` has:
   ```yaml
   homeassistant:
     packages: !include_dir_named packages
   ```
5. Restart Home Assistant.
6. Add `dashboard_fr0gg3r_smart_export.yaml` as a new dashboard (Settings → Dashboards → Add Dashboard → Edit in YAML, paste the contents).
7. Go to the **Custom Export Settings** tab and configure your reserve SOC, notification phone(s), and (optionally) your aircon sensors.

## Known issue — combined SOC not capacity-weighted on multi-unit setups

If you're running multiple Zendure units of **different capacities** behind gast777's proxy (e.g. one unit with a battery expansion pack, one without), Gielz1986's combined `sensor.zendure_total_state_of_charge` appears to be a **simple average** across units rather than weighted by each unit's actual capacity.

On a real asymmetric setup (5.28 kWh + 2.4 kWh), this was found to overstate the true combined SOC by roughly **6-9 percentage points** — which matters directly for reserve/export decisions, since the smaller unit can hit its real 0%/100% well before the reported average reflects it.

This has been reported upstream: [gast777/Zendure-zenSDK-proxy#26](https://github.com/gast777/Zendure-zenSDK-proxy/issues/26).

**This package works around it independently**, without touching Gielz1986's files: `sensor.zendure_true_total_state_of_charge` computes the correct capacity-weighted value from the individual per-unit proxy sensors, and all reserve/export logic in this package uses that corrected sensor instead of the raw one.

If you're on a single unit, or units of matched capacity, this workaround is a no-op — Gielz1986's raw sensor is already correct in those cases, and you don't need to do anything.

## Configuration reference

| Helper | Purpose |
|---|---|
| `input_boolean.zendure_export_armed` | Master arm switch for price-based export |
| `input_boolean.zendure_vacation_mode` | Auto-arm daily without confirmation prompt |
| `input_number.zendure_night_reserve_soc` | Minimum SOC reserve to protect |
| `input_number.zendure_export_not_before_hour` | Don't start exporting before this hour |
| `input_datetime.zendure_export_reminder_time` | Daily arm-prompt notification time |
| `input_number.zendure_aircon_temp_threshold` | Temperature above which aircon blocking kicks in |
| `input_text.zendure_bedroom1/2/3_temp_sensor` + `_airco` | Room temp sensor / AC entity pairs (leave blank to skip a room) |
| `input_text.zendure_notify_phone1/2` | Notify service names for push notifications |

## Credits

- [Gielz1986](https://github.com/Gielz1986/Zendure-HA-zenSDK) — the base Zendure Home Assistant integration this package builds on.
- [gast777](https://github.com/gast777/Zendure-zenSDK-proxy) — the multi-unit proxy that makes running multiple Zendure hubs possible.

## License

MIT — see [LICENSE](LICENSE).
