# SpoolTap — Setup Guide

This guide takes you from a working HA + Bambu printer to a fully
operational tap-to-track filament system. Expect ~1 hour of setup plus
however long it takes to sticker your spool collection.

## Prerequisites

| Requirement | Notes |
|---|---|
| Home Assistant OS/Supervised | **2024.11 or newer** (the dashboard uses the Sections view + `perform-action`); tested on 2026.x. Needs `packages:` support in configuration.yaml |
| [HACS](https://hacs.xyz) + 3 frontend cards | [Mushroom](https://github.com/piitaya/lovelace-mushroom), [card-mod](https://github.com/thomasloven/lovelace-card-mod), [auto-entities](https://github.com/thomasloven/lovelace-auto-entities) — the dashboard needs all three (see Part 2) |
| [Spoolman](https://github.com/Donkie/Spoolman) addon + the Spoolman HA integration | Your spools/filaments entered with vendor, material, name, weights (tare!) |
| [Bambuddy](https://bambuddy.dev) addon | Third-party, actively-developed addon. **Linked to your Bambu account** (cloud login) — this is how custom profile codes resolve — and connected to your printer + Spoolman. Its REST API is the one external contract SpoolTap depends on; if a Bambuddy update changes an endpoint, SpoolTap surfaces a ⚠ rather than silently misbehaving |
| [ha-bambulab](https://github.com/greghesp/ha-bambulab) integration | Local MQTT mode is fine |
| HA Companion app on **Android** | iOS background NFC is more restrictive; Android is the tested path |
| NTAG215 stickers | One per AMS slot + one per spool. 25 mm round stickers work well |
| Bambu Studio | With **custom filament profiles** created for your third-party vendor lines (cloud account) |

## Part 1 — Install the package

1. Copy `packages/spooltap.yaml` into your HA `packages/` directory (create it
   and add `homeassistant: packages: !include_dir_named packages/` to
   configuration.yaml if you don't have one).
2. **Configure your values.** Copy `secrets.example.yaml` to `secrets.yaml`
   (next to your `configuration.yaml`) and fill in the keys below. The package
   reads every install-specific value from `secrets.yaml` — you never edit the
   package file itself, and your values never leave your machine.

| `secrets.yaml` key | What it is | How to find yours |
|---|---|---|
| `spooltap_spoolman_base` | Spoolman add-on URL (`http://host:7912`) | Add-on page → Hostname |
| `spooltap_bambuddy_base` | Bambuddy add-on URL (`http://host:8000`) | Add-on page → Hostname |
| `spooltap_spoolman_config_entry_id` | Spoolman HA integration config-entry id | Developer Tools → Actions → `homeassistant.reload_config_entry` → pick **Spoolman** as the target → toggle to **YAML mode**: the `entry_id:` shown is the value (this is the exact action SpoolTap calls, so it's guaranteed to be the right id) |
| `spooltap_printer_id` | Your printer's id in Bambuddy | `GET /api/v1/printers/` on the Bambuddy API |
| `spooltap_ams_prefix` | Your AMS tray-sensor stem (the part before `left_tray_1`) | Developer Tools → States, search `tray_1` |
| `spooltap_external_sensor` | Your external-spool sensor | Developer Tools → States, search `external` |
| `spooltap_print_status_sensor` | Your printer's print-status sensor | Developer Tools → States, search `print_status` |
| `spooltap_notify_target` | Your phone's notify service | Developer Tools → Actions, search `notify.mobile_app` |
| `spooltap_nozzle` | Your nozzle diameter (setting_ids are per-nozzle) | e.g. `0.4` |

3. Restart Home Assistant (first install creates many new entities). On first
   start the config helpers **seed from your `secrets.yaml` automatically**.
   Later, any time you edit `secrets.yaml`: reload the YAML config (or restart)
   first so `!secret` re-resolves, **then** run **SpoolTap: Apply Config** — the
   button on the dashboard's Inventory tab (Part 2), or **Developer Tools →
   Actions → `script.spooltap_apply_config`** if you haven't installed the
   dashboard yet.

> All of SpoolTap's on/off toggles (Registration Mode, the BB Reconciler
> auto-heal, the notification switches) ship **OFF** — that's the intended
> resting state. Turn Registration Mode on only while binding tags.

> **AMS topology (one hand-edit needed).** SpoolTap ships configured for a
> dual-AMS layout (`left_1…left_4`, `right_1…right_4`, plus `external`). Two
> spots in `spooltap.yaml` reference your AMS tray sensors by **literal name**
> (Home Assistant forbids templating trigger entity-ids) — find the comments
> marked `TOPOLOGY` and replace the placeholder sensor names with your real
> ones. **Critical:** the names there must match `spooltap_ams_prefix` exactly
> (the part before `left_tray_1`) — a mismatch silently breaks empty-slot
> assignment. Running a single AMS? Also delete the `right_*` lines there.

## Part 2 — Install the dashboard

The dashboard is the SpoolTap control surface (Assign / Bind / Modify /
Inventory) — install it once the package is in and HA has restarted.

1. **Install the three HACS frontend cards** (HACS → Frontend): Mushroom,
   card-mod, auto-entities. Without them you'll see red "Custom element doesn't
   exist" errors — this is the #1 reason a borrowed dashboard looks broken.
2. Copy `dashboards/spooltap.yaml` into `config/dashboards/` and register it in
   `configuration.yaml` (a YAML-mode dashboard with its own sidebar entry), then
   restart. Full instructions — including the storage-mode-vs-full-YAML resource
   rules and a no-`configuration.yaml`-edit paste alternative — are in
   [dashboards/README.md](dashboards/README.md).
3. Open **SpoolTap** from the sidebar. It lands on the **Assign** tab; the four
   top buttons switch modes. On the **Inventory** tab, tap **Apply Config** once.

> Custom (HACS) cards render in the browser, so the only thing a config check
> can't prove is that they *paint*. After install, open the dashboard once: if
> every tab renders and the mode buttons light up, you're set.

## Part 3 — One-time Spoolman fields

Create these **extra fields** in Spoolman (Settings → Extra Fields), all
type *text* unless noted:

- Spool: `nfc_tag`, `nfc_active_tray`
- Filament: `nfc_tray_idx`, `nfc_setting_id`, `nfc_temp_min` (integer),
  `nfc_temp_max` (integer)

Then run **Sync Filament Profiles** from the dashboard once: it matches
your filaments to your Bambu Studio cloud profiles (codes + setting_ids)
and materializes the spool fields. Review the Profile Sync card for any
unmatched filaments and map them with the offered candidate codes.

> **Why filament-level profile fields?** Spoolman embeds the full filament
> record (extras included) in every spool read — map a vendor line once
> and every spool of it, present and future, inherits the mapping.

## Part 4 — Tags

1. **Create tag identities first** (HA → Settings → Tags → Add): one per
   slot ("AMS Left Tray 1"… + "External Spool") and a batch for spools
   ("Spool 01"… — names are permanent, spool association is not).
2. **Write the physical stickers** from the Companion app (Settings →
   Tags → tap a tag → Write). Stick slot tags next to their slots; keep
   spool tags in a pile for sticker-time.
3. **Register slot tags**: paste each slot tag's id into its
   `input_text.nfc_slot_*` helper (or scan it in Registration Mode and use
   Bind to AMS Slot).
4. **Bind spool tags** as you sticker each spool: Registration Mode ON →
   scan the tag → pick the spool in the faceted picker → Bind. Status
   confirms; tag recycling (off archived/dead spools) is automatic.

## Part 5 — Custom profiles & the printer

For each third-party vendor line you own, create a **custom filament
profile in Bambu Studio** (e.g. "SUNLU PLA+ Gen 2") while signed in — it
syncs to your Bambu cloud, where Bambuddy reads it.

**Critical, easy-to-miss step:** the *printer* must learn each brand-new
profile once before pushes referencing it stick. Either briefly connect
the printer to your Bambu account (let it sync, then return to LAN mode),
or assign the profile to any tray from Bambu Studio while connected.
Profiles the printer has seen before need nothing, ever again.

## Part 6 — Verify

1. Scan a slot tag → status: *"AMS slot … scanned"*, chip: **Awaiting
   Spool Scan**.
2. Scan a bound spool's tag → status ends **`✓ printer confirmed`**, the
   tray shows the real filament name, and the filament appears in your
   slicer's device list.
3. Print something. Watch Spoolman's remaining-weight drop on its own.

## Daily reference

Each dashboard tab carries inline hint cards (what to scan, what each
status means), so you can operate SpoolTap day-to-day from the dashboard
alone. The full written manual (`NFC_FILAMENT_WORKFLOW.md`) — the Modify
Spool inventory workbench, Profile Sync nuances, notification toggles, the
Cancel button, and every status message explained — is being prepared and
will land in this repo.

## Troubleshooting quick hits

| Symptom | Cause / fix |
|---|---|
| Chip says ⚠ "printer did NOT confirm" | AMS was busy or tray empty — re-scan the pair once settled (it's idempotent) |
| Tray tile looks right but tap-in says empty / slicer doesn't list it | The profile code didn't stick → the printer doesn't know that profile yet (Part 5's learn step) |
| Status says GENERIC fallback | Filament has no mapped profile — run Profile Sync, check the card |
| A scan does nothing | Same tag scanned twice in a row — tap Cancel Assignment, scan again |
| New spool not recognized after binding | Wait ~15 s (sensor poll) or run Profile Sync (materializes new spools) |
| Inventory spool grid is empty | The Spoolman **HA integration** isn't set up, or its `sensor.spoolman_spool_*` entities are named differently on your install — confirm the integration is added and those sensors exist (Dev Tools → States). The grid and tag-scan resolution rely on the Spoolman integration's current entity/attribute schema |
