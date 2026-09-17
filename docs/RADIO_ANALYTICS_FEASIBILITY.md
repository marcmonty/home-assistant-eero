# Eero radio analytics: offline feasibility and validation plan

Status: **research only**. No new endpoint, API request, Home Assistant entity or Atlas chart is implemented here. No live Eero/Home Assistant data was accessed during this audit.

## User-facing goal

For each physical Eero and supported radio, display measured 2.4 GHz, 5 GHz Low, 5 GHz High (or model-appropriate bands), with a six- and 24-hour history: total channel busyness, own-radio airtime, other-device interference, noise floor (dBm), channel, control channel, bandwidth, and channel changes. Distinguish per-radio RF measurements from per-client Wi-Fi channel/RSSI and traffic throughput. An app screenshot establishes that eero displays these measurements, **not** that its private API offers them to this integration.

Official descriptions and data limits: https://eero.com/support/articles/what-is-wifi-radio-analytics . Eero reports up to 48 hours of app history; exact availability to third-party integrations is **unverified**. This is described as an eero Plus feature; model and account access can vary.

## Verified by static inspection of this repository

- `custom_components/eero/api/__init__.py`: `EeroAPI.update()` retrieves account/network resources, Eero and client records, selected activity series, plus thread and some optional premium/network resources. `update_activity()` supports the configured activity mappings. It does not currently fetch a radio-analytics series.
- `custom_components/eero/api/const.py`: `ACTIVITY_MAP` contains network/eero/client data usage and adblock/blocked/inspected insights. It has no documented mapping for per-radio channel busyness, own-radio activity, interference or noise floor.
- `custom_components/eero/api/eero.py`: `EeroDevice` exposes status, firmware, client counts, location, LED, reboot and activity usage totals; no radio-history accessor.
- `custom_components/eero/api/client.py`, `binary_sensor.py`, and `device_tracker.py`: wireless **client** signal, channel, operating frequency, link width and serving Eero are partly available. Client channel readings must never be presented as the AP's own complete radio analytics.
- The integration is cloud-polling (manifest version 1.8.1). Existing configured scan interval is independent of any still-unverified radio-history cadence.

### Privacy and reliability concerns to address before any diagnostics

- **Do not turn on `save_responses`** to inspect radio fields. `EeroAPI.save_response` writes whole account/network API responses as JSON; responses can contain SSIDs, Wi-Fi passwords, device addresses and identifiers. Do not commit raw payloads, screenshots with identifiers, tokens/cookies or captured mobile traffic to GitHub or send them to an external service.
- A missing client usage object is currently returned as `0` by `EeroClient.usage_down`/`usage_up`. A proposed radio pipeline must preserve `null`/unavailable distinctly from a measured zero. The update routine can return previously held account data after an API error: propagate sample timestamp, source and stale state.
- The Eero cloud API used by this fork is not a documented third-party radio-analytics API. Avoid guessing/probing endpoint names, changing router settings, installing a packet interceptor or increasing polling just to make a graph appear.

## Safe next validation, when HA is next online and the owner opts in

1. Check the existing, already-retrieved Eero device/network response **key structure locally in memory**, without saving or transmitting values. Explicit allowlist only: keys that suggest per-radio measurements or links already present in documented `resources`. No cookie/token/password/SSID/client/IP/MAC/serial/URL output. No new network calls at this stage. If the key-only structure has no lead, record `source_unverified` and stop.
2. If a specific existing read-only resource reference is found, verify its permission/subscription conditions and schema with a minimal **redacted and locally inspected** sample. Only a verified source may become an opt-in read-only API request; use an explicit timeout, rate limit and graceful unavailable result. No guessed URLs and no raw response logger.
3. Design a typed, schema-validated adapter for radio identity (physical node + band), sample timestamp and cadence, percentage values [0,100], negative noise dBm, channel/width and channel-change events. Keep invalid/missing values absent; test future API changes, rate limits and unavailable subscriptions with synthetic fixtures.
4. Publish scoped Home Assistant entities or history only if measurements really exist. Atlas Home should receive a privacy-projected, administrator-only data feed. It must not expose network credentials or client identifiers. Plot measured 6h/24h histories only; derive mean/P99/max from observed samples with sample-window and missing-data warnings, or label Eero-provided aggregates as such.
5. If the source remains unavailable, ship **separately labelled** real alternatives (per-client signal trends, download/upload rates, Eero status/data-usage totals) and omit fake radio busyness, noise and precise indoor positioning.

## Go/no-go

Proceed with full radio graphs only after a read-only data source and its timestamps, identity, units, access and sample cadence have been verified. Until then the screenshot-style radio analytics are an unverified capability, not a deliverable. No live HA or router changes are required for this documentation-only change.
