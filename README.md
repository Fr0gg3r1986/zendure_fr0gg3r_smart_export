# Zendure Fr0gg3r Smart Export

Price-aware evening/vacation export control for Zendure batteries in Home Assistant, built as a companion package on top of [Gielz1986's Zendure-HA-zenSDK](https://github.com/Gielz1986/Zendure-HA-zenSDK) integration, with optional support for multi-unit setups via [gast777's Zendure-zenSDK-proxy](https://github.com/gast777/Zendure-zenSDK-proxy).

Automates grid export during peak Nordpool price windows, with configurable SOC reserve, vacation mode, aircon-load blocking, next-day solar forecast awareness, arm/disarm notifications, and a full dashboard — including a fix for combined-SOC accuracy on multi-unit setups.

**Author:** Fr0gg3r1986

**Status:** 🧪 Active testing on a single real setup (2× SF2400 AC+, asymmetric capacity, via gast777 proxy). Core export logic has been running in production for a while; the solar-forecast block is newer and still being tuned. See [Known TODOs / Testing Notes](#known-todos--testing-notes) below before assuming every default is final.

Full version history: [CHANGELOG.md](CHANGELOG.md).

---

## Disclaimer

- **This project is not affiliated with, endorsed by, or supported by Zendure, Gielz1986, or gast777.** It's an independent, unofficial add-on built by a user, for personal use, and shared publicly as-is.
- **Use entirely at your own risk.** This code reads from and writes to real battery hardware, and makes automated decisions about grid export based on price and forecast data. Bugs, misconfiguration, upstream API/firmware changes, or incorrect assumptions in this code could result in unexpected battery behavior, financial cost from incorrect export/import decisions, or hardware wear.
- **No warranty, no liability.** This software is provided "as is," with no guarantee it is fit for any particular purpose. The author accepts **no responsibility or liability whatsoever** for any hardware malfunction, damage, data loss, financial loss, or any other direct or indirect harm arising from installing, configuring, or using this code — see the [LICENSE](LICENSE) for the full legal terms.
- **Test carefully before trusting it unattended.** Review the automations, understand what they do, and monitor behavior closely — especially reserve/SOC-floor logic and anything that controls charge/discharge — before leaving it to run unsupervised.

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
- **Low-solar-tomorrow blocking** — optionally pause evening export if tomorrow's forecasted solar (via Forecast.Solar) is below a configurable threshold, so the battery isn't drained right before a low-sun day when it may not fully recharge. Per-day override available.
- **Arm/disarm prompts** — daily notification with action buttons to confirm or postpone export, plus a "custom start time tonight" override.
- **Force-stop button** — one tap to immediately stop export and disarm, with confirmation.
- **Full dashboard** — Overview, Export Control, Export Settings, Battery Health, History, Solar (incl. forecast), P1 Meter, Proxy Details (for multi-unit setups), and Releasenotes.
- **Multi-unit SOC fix** — see [Known issue](#known-issue--combined-soc-not-capacity-weighted-on-multi-unit-setups) below.

## Requirements

| Dependency | Required? | Notes |
|---|---|---|
| [Gielz1986's Zendure-HA-zenSDK](https://github.com/Gielz1986/Zendure-HA-zenSDK) | **Always** | Base integration. Tested against the Global (EN) Integration, v20260716. |
| [gast777's Zendure-zenSDK-proxy](https://github.com/gast777/Zendure-zenSDK-proxy) | **Only for multi-unit setups** | Needed if you're running 2+ physical Zendure hubs behind Gielz1986's integration. Single-unit setups can skip this entirely. |
| Home Assistant `packages` support | **Always** | See install steps below. |
| A Nordpool sensor (HACS) | **Always** | For price-window detection. |
| [Forecast.Solar](https://www.home-assistant.io/integrations/forecast_solar/) (built-in) | **Only for low-solar-tomorrow blocking** | One instance per solar plane. Set the threshold to 0 to disable this feature entirely without needing to remove anything. |

## Installation

1. Make sure Gielz1986's integration is installed and working first.
2. If you're running multiple Zendure units, set up gast777's proxy first too.
3. If you want low-solar-tomorrow blocking, set up Forecast.Solar (one instance per solar plane) first too.
4. Copy `zendure_fr0gg3r_smart_export.yaml` into your `packages/` folder.
5. Make sure your `configuration.yaml` has:
   ```yaml
   homeassistant:
     packages: !include_dir_named packages
   ```
6. Restart Home Assistant.
7. Add `dashboard_fr0gg3r_smart_export.yaml` as a new dashboard (Settings → Dashboards → Add Dashboard → Edit in YAML, paste the contents).
8. Go to the **Export Settings** tab and configure your reserve SOC, notification phone(s), aircon sensors, and (optionally) your low-solar threshold.

## Known TODOs / Testing Notes

This project is running in production on one real setup, but a few things are still open or under active observation rather than fully settled:

- **`input_number.zendure_low_solar_threshold_kwh` default (5 kWh) is an unvalidated starting guess**, not a calculated value. Rough math based on a 7.68 kWh combined capacity and ~30% reserve floor suggests something closer to **5.4-5.5 kWh** better matches the actual max exportable energy per evening session — but real-world behavior over several days/weeks hasn't been observed yet to confirm whether to match that number exactly or add a conservative buffer above it. Treat the default as provisional.
- **`switch.equal_mode_charging` (gast777 proxy) behavior with asymmetric unit capacities is unconfirmed.** "Equal mode" applies the same wattage to every active unit rather than scaling by capacity — on a matched-capacity setup this is fine, but on an asymmetric setup (e.g. 5.28 kWh + 2.4 kWh) it's not yet confirmed whether this causes uneven C-rate stress on the smaller unit, or whether the proxy's non-equal default already handles this more sensibly. Worth investigating before recommending either setting confidently for asymmetric setups.
- **The upstream SOC-weighting bug** ([gast777/Zendure-zenSDK-proxy#26](https://github.com/gast777/Zendure-zenSDK-proxy/issues/26)) is still open. This package's `sensor.zendure_true_total_state_of_charge` workaround is stable, but if/when it's fixed upstream, the workaround becomes redundant (harmless either way — it'll just agree with the raw sensor once weighted).
- **Only tested on one real hardware configuration** (2× SF2400 AC+, one with an AB3000L expansion, one without). Single-unit setups and other capacity combinations should work in principle (the code paths degrade gracefully — e.g. the low-solar block is a no-op if you don't configure Forecast.Solar), but haven't been verified firsthand.

Feedback, issue reports, and PRs on any of the above are welcome.

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
| `input_number.zendure_low_solar_threshold_kwh` | Combined tomorrow solar forecast (kWh) below which export is blocked. Set to 0 to disable. |
| `input_boolean.zendure_ignore_low_solar_forecast` | Per-day override for the low-solar block |
| `input_text.zendure_notify_phone1/2` | Notify service names for push notifications |

## Credits

- [Gielz1986](https://github.com/Gielz1986/Zendure-HA-zenSDK) — the base Zendure Home Assistant integration this package builds on.
- [gast777](https://github.com/gast777/Zendure-zenSDK-proxy) — the multi-unit proxy that makes running multiple Zendure hubs possible.

## License

MIT — see [LICENSE](LICENSE).
