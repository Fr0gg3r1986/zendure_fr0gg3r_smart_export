# Changelog

> **Disclaimer:** This is a personal, hobbyist project — see the [README](README.md#disclaimer) for the full disclaimer. Not affiliated with Zendure, Gielz1986, or gast777. Use at your own risk.

All notable changes to Zendure Fr0gg3r Smart Export are documented here. The package and dashboard are versioned independently (they don't always ship together).

## Package (`zendure_fr0gg3r_smart_export.yaml`)

### 2.16.0 (2026-08-21)
- Added a grid-spike export block: `binary_sensor.zendure_grid_spike_detected` trips when `sensor.smart_export_p1_active_power` (the P1 wrapper sensor) exceeds `input_number.zendure_grid_spike_threshold_w` (default 300W), catching sudden heavy consumption from any appliance — smart or not (e.g. a non-smart electric stove) — since it watches the grid meter rather than individual devices.
- Uses native `delay_on` (default 15s, `input_number.zendure_grid_spike_entry_seconds`) to ignore brief transients, and `delay_off` (default 5min, `input_number.zendure_grid_spike_clear_minutes`) so it rides through an appliance's normal heating-element on/off cycling instead of flapping the mode every time it cycles off.
- Blocks/reverts export the same way the aircon and low-solar blocks already do, reverting to whichever mode `input_select.zendure_revert_mode` is currently set to.

### 2.15.0 (2026-08-21)
- **Bug fix:** `binary_sensor.zendure_aircon_reserve_needed`'s temperature check was being silently bypassed whenever export was armed. The "away" condition incorrectly included `input_boolean.zendure_export_armed` (the same toggle required for export to run at all), so whenever export could actually happen, the temperature-based block was disabled — only an actively-cooling AC unit could still block, not a hot room on its own. "Away" now only means vacation mode, as originally intended, so a hot room correctly blocks export whenever armed and not on vacation, regardless of the AC's current state.

### 2.14.0 (2026-08-21)
- Added solar forecast accuracy tracking. A new daily automation (`smart_export_snapshot_solar_accuracy`, 23:55) captures each plane's actual yield and Forecast.Solar's predicted total for that day into 6 new input_numbers, right before the daily yield sensors reset.
- Added `sensor.smart_export_solar_actual_total_yesterday`, `sensor.smart_export_solar_predicted_total_yesterday`, and `sensor.smart_export_solar_forecast_accuracy` (%) for the dashboard's actual-vs-predicted comparison card and histogram.

### 2.13.0 (2026-08-21)
- Added `input_text.smart_export_p1_meter_entity_prefix` and 11 wrapper template sensors (`sensor.smart_export_p1_active_power`, `_l1/l2/l3`, `active_voltage_l1/l2/l3`, `active_current_l1/l2/l3`, `wifi_strength`). The dashboard now references these fixed names instead of a raw HomeWizard P1 entity ID directly — set the prefix once (Export Settings tab) and everything follows automatically. Removes the need to find-and-replace P1 entity IDs in the dashboard YAML, and keeps the public repo free of any hardware-specific identifiers.

### 2.12.0 (2026-08-21)
- Added a low-solar-tomorrow export block: `sensor.zendure_solar_forecast_tomorrow` sums Forecast.Solar's per-plane "tomorrow" estimate across all 3 real solar systems (Huawei south, SolaX X1 east, SolaX X2 west). `binary_sensor.zendure_low_solar_tomorrow` trips when that sum falls below `input_number.zendure_low_solar_threshold_kwh` (default 5 kWh — see [TODO](README.md#known-todos--testing-notes)). When tripped, evening export is blocked the same way the aircon reserve block already works — preventing the battery being drained by price-chasing export on an evening before a low-sun day, when it may not fully recharge. Overridable per-day via `input_boolean.zendure_ignore_low_solar_forecast`.
- Requires the Forecast.Solar integration (or compatible) configured per solar plane.

### 2.11.0 (2026-08-20)
- Added `sensor.zendure_true_total_state_of_charge` — a capacity-weighted SOC across the two proxy units (5.28 kWh + 2.4 kWh), correcting Gielz1986's raw `sensor.zendure_total_state_of_charge`, which is a naive (unweighted) average and was found to overstate available energy by ~6-9 SOC points in an asymmetric multi-unit setup.
- Repointed all 7 reserve/export decision-logic checks and both notification messages to the corrected sensor.
- Package renamed from `zendure_custom_evening_export` to Zendure Smart Export ahead of publishing publicly; added author credit and repo link. No functional changes from this rename.

### 2.10.1 (2026-07-08)
- Prior baseline. Consolidated final version predating this changelog — see the project writeup for earlier 2.x release history (SOC floor clamping fix, choose-block revert logic fix, indirect entity reference fixes, versioned dashboard split, vacation mode, arm/disarm notifications with action buttons).

---

## Dashboard (`dashboard_fr0gg3r_smart_export.yaml`)

### 2.19.0
- Added a "Grid Spike Blocking" tile to Export Control, and a new "Grid Spike Settings" section on Export Settings: current grid power, spike threshold, entry delay, clear window, live block status. Companion to package v2.16.0's grid-spike export block.

### 2.18.0
- Added a "Disclaimer" view: the full README disclaimer rendered as alert cards directly in the dashboard UI, not just in YAML comments.

### 2.17.0
- Added a "Forecast Accuracy" section to the Solar view: per-inverter actual-vs-predicted comparison (yesterday), a combined accuracy %, and a 30-day actual-vs-predicted histogram. Companion to package v2.14.0's daily solar accuracy snapshot automation.

### 2.16.0
- P1 meter cards now reference `sensor.smart_export_p1_*` wrapper sensors (from package v2.13.0) instead of a raw HomeWizard entity ID directly — set `input_text.smart_export_p1_meter_entity_prefix` once (new "P1 Meter Settings" section on Export Settings) and every P1 card follows automatically. No more manual find-and-replace needed when reusing this dashboard on a different system.

### 2.15.0
- Added a Forecast section to the Solar view: combined + per-plane today/this-hour/remaining-today figures, peak production times, and a clearly flagged display of an unmatched 4th Forecast.Solar device (appears to be an empty/misconfigured entry — not used in any totals).
- Added a "Low Solar Blocking" tile to Export Control, and a new "Solar Reserve Settings" section on Export Settings: combined tomorrow forecast, threshold, live block status, per-day override toggle. Companion to package v2.12.0's low-solar-tomorrow export block.
- Removed two leftover empty artifacts (a stray "New section" heading on Export Control, an empty trailing grid on Export Settings).

### 2.13.1
- Added "Package vX.X.X · Dashboard vX.X.X · by Fr0gg3r1986" footer to the bottom of the Live view, matching the existing footer on Custom Export.
- Dashboard renamed ahead of publishing publicly; added author credit. No functional changes.

### 2.13.0
- Added a new glance card under Home Energy on the Live view showing per-unit detail: Unit 1 SOC, Unit 1 Power, Unit 2 SOC, Unit 2 Power.

### 2.12.0
- All 6 combined-SOC displays repointed from the raw `sensor.zendure_total_state_of_charge` to the corrected `sensor.zendure_true_total_state_of_charge`. "SOC %" label relabeled to "Real SOC %".

### 2.11.0
- Added the "Proxy Details" view: gast777 proxy sensors (per-unit power/SOC/mode/serial, active device, offgrid outlets), proxy control switches (no-sleep, all-active, equal-mode, dual damper), and per-battery-pack temperature/SOC/cell-balance rows.

### 2.10.1
- Prior baseline predating this changelog.
