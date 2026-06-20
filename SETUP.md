# SpoolTap — Setup Guide

This guide takes you from a working HA + Bambu printer to a fully
operational tap-to-track filament system. Expect ~1 hour of setup plus
however long it takes to sticker your spool collection.

## Prerequisites

| Requirement | Notes |
|---|---|
| Home Assistant OS/Supervised | Tested on 2026.x; needs `packages:` support in configuration.yaml |
| [Spoolman](https://github.com/Donkie/Spoolman) addon + the Spoolman HA integration | Your spools/filaments entered with vendor, material, name, weights (tare!) |
| [Bambuddy](https://bambuddy.dev) addon | **Linked to your Bambu account** (cloud login) — this is how custom profile codes resolve — and connected to your printer + Spoolman |
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
| `spooltap_spoolman_config_entry_id` | Spoolman HA integration config-entry id | Settings → Devices & Services → Spoolman → ⋮ → the id in the page URL |
| `spooltap_printer_id` | Your printer's id in Bambuddy | `GET /api/v1/printers/` on the Bambuddy API |
| `spooltap_ams_prefix` | Your AMS tray-sensor stem (the part before `left_tray_1`) | Developer Tools → States, search `tray_1` |
| `spooltap_external_sensor` | Your external-spool sensor | Developer Tools → States, search `external` |
| `spooltap_notify_target` | Your phone's notify service | Developer Tools → Actions, search `notify.mobile_app` |
| `spooltap_nozzle` | Your nozzle diameter (setting_ids are per-nozzle) | e.g. `0.4` |

3. Restart Home Assistant (first install creates many new entities), then tap
   **SpoolTap: Apply Config** on the dashboard to push your `secrets.yaml`
   values into the system. Re-run it any time you change `secrets.yaml`.

> **AMS topology:** SpoolTap ships configured for a dual-AMS layout
> (`left_1…left_4`, `right_1…right_4`, plus `external`). If you run a single
> AMS, delete the `right_*` entries from the helper options and slot maps —
> look for the comments marked `TOPOLOGY` in `spooltap.yaml`.

## Part 2 — One-time Spoolman fields

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

## Part 3 — Tags

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

## Part 4 — Custom profiles & the printer

For each third-party vendor line you own, create a **custom filament
profile in Bambu Studio** (e.g. "SUNLU PLA+ Gen 2") while signed in — it
syncs to your Bambu cloud, where Bambuddy reads it.

**Critical, easy-to-miss step:** the *printer* must learn each brand-new
profile once before pushes referencing it stick. Either briefly connect
the printer to your Bambu account (let it sync, then return to LAN mode),
or assign the profile to any tray from Bambu Studio while connected.
Profiles the printer has seen before need nothing, ever again.

## Part 5 — Verify

1. Scan a slot tag → status: *"AMS slot … scanned"*, chip: **Awaiting
   Spool Scan**.
2. Scan a bound spool's tag → status ends **`✓ printer confirmed`**, the
   tray shows the real filament name, and the filament appears in your
   slicer's device list.
3. Print something. Watch Spoolman's remaining-weight drop on its own.

## Daily reference

The full user manual (`NFC_FILAMENT_WORKFLOW.md`) — day-to-day flows, the
Modify Spool inventory workbench, Profile Sync, notification toggles, the
Cancel button, and every status message explained — is being prepared and
will land in this repo.

## Troubleshooting quick hits

| Symptom | Cause / fix |
|---|---|
| Chip says ⚠ "printer did NOT confirm" | AMS was busy or tray empty — re-scan the pair once settled (it's idempotent) |
| Tray tile looks right but tap-in says empty / slicer doesn't list it | The profile code didn't stick → the printer doesn't know that profile yet (Part 4's learn step) |
| Status says GENERIC fallback | Filament has no mapped profile — run Profile Sync, check the card |
| A scan does nothing | Same tag scanned twice in a row — tap Cancel Assignment, scan again |
| New spool not recognized after binding | Wait ~15 s (sensor poll) or run Profile Sync (materializes new spools) |
