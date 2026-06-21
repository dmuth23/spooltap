# SpoolTap

**Tap a sticker, load a spool, and your whole filament stack just… knows.**

SpoolTap is a phone-NFC filament tracking system for Bambu Lab printers,
built entirely on Home Assistant + [Spoolman](https://github.com/Donkie/Spoolman)
+ [Bambuddy](https://bambuddy.dev) + the
[ha-bambulab](https://github.com/greghesp/ha-bambulab) integration. No
custom hardware, no NFC reader — your phone and a pack of NTAG215 stickers
are the entire input device.

## What it does

**Loading filament becomes two taps.** Stick a permanent NFC tag next to
each AMS slot (and the external feed) and one on each spool. When you load
a spool: scan the slot's tag, scan the spool's tag, done. Home Assistant
then automatically:

- **Tells the printer the truth** — the AMS slot shows the *real* filament
  identity ("SUNLU PLA+ Gen 2", "Overture Matte PLA"…) with the right
  color and temps, not "Generic PLA". Custom Bambu Studio profiles are
  pushed with their cloud `setting_id` — the same mechanism the slicer
  uses — so they appear in your slicer's installed-filament list, ready to
  pick.
- **Enables automatic usage tracking** — Bambuddy's slot-assignment table
  is synced, which is the signal that makes it deduct every print's usage
  from the right Spoolman spool. Non-RFID spools get the
  weight-tracking experience Bambu reserves for its own spools.
- **Keeps Spoolman's records straight** — which spool is in which slot,
  with stale claims cleaned up automatically.
- **Verifies and reports honestly** — every printer push is read back and
  retried; a dashboard chip shows Working → Awaiting Spool Scan →
  Success ✓ / Failed ⚠, and your phone gets a notification you can
  trust. A Cancel button bails out of anything, any time.

**Inventory day becomes a scan-and-weigh loop.** The Modify Spool
workbench lets you load any spool (scan its tag, pick its tag name, or
filter by brand/type), weigh it on a kitchen scale (it computes
remaining = gross − tare and shows you the *expected* weight so drift is
obvious), fix names and materials, or archive empties — which frees their
tags for recycling onto new spools.

**New filament? One profile, one sync.** Create a custom filament profile
in Bambu Studio; the nightly Profile Sync (or one dashboard tap) matches
it to your Spoolman filament and resolves everything the printer needs.
Ambiguous names land on a workbench card with copyable candidate codes —
no guessing, no silent wrong answers.

## The stack

| Piece | Role |
|---|---|
| Home Assistant | Orchestration — one package YAML of helpers, scripts, automations |
| HA Companion app (Android) | The NFC reader (tag scans fire HA events) |
| Spoolman | Inventory database + source of truth for spool↔profile mapping |
| Bambuddy | Printer bridge: usage tracking, slot table, cloud profile library, and the profile push |
| ha-bambulab | Printer telemetry (tray sensors used for push verification) |
| NTAG215 stickers | ~$10 for more than you'll ever need |

## Design principles (learned the hard way)

- **A scan-pair is a declaration of physical reality** — you scan while
  you load. Slot tags are permanent; spool tags are recycled when spools
  die.
- **Records first, printer last** — the fast, reliable bookkeeping always
  completes; the slow physical push comes last and is verified, retried,
  and honestly reported.
- **Never trust a fire-and-forget push** — the printer silently drops
  custom profile codes it doesn't have cached; only a `setting_id`-bearing
  push (via Bambuddy) sticks reliably.
- **Never reload an integration on the hot path** — entity storms take
  down websockets and starve MQTT. Sensors are pre-materialized once;
  values refresh by polling.
- **When the matcher isn't sure, it asks** — ambiguity goes to a human
  with copy-paste candidates, never a silent guess that ends up on your
  printer forever.

## What's in this repo

- [`packages/spooltap.yaml`](packages/) — the whole system (helpers, scripts,
  automations, template sensors). Parameterized: reads every install value from
  your `secrets.yaml`.
- [`dashboards/spooltap.yaml`](dashboards/) — the control surface (Assign /
  Bind / Modify / Inventory). Part of SpoolTap, not optional.
- [`secrets.example.yaml`](secrets.example.yaml) — copy to `secrets.yaml`, fill
  in your nine values.

## Documentation

- [SETUP.md](SETUP.md) — prerequisites, installation, and the `secrets.yaml`
  values for your own install.
- [dashboards/README.md](dashboards/README.md) — installing the dashboard and
  its HACS card prerequisites.
- `NFC_FILAMENT_WORKFLOW.md` — the user manual (registration runbook,
  day-to-day flows, inventory workbench, Profile Sync). *Coming soon.*
- `FILAMENT_SYSTEM_ARCHITECTURE.md` — internals: every component, every
  data flow, and a decision log of everything that bit us so it doesn't
  bite you. *Coming soon.*

## Credit

Sparked by a fellow tinkerer's reddit post about NFC spool tracking —
this is that idea, run all the way to the finish line. Built
collaboratively with Claude (Anthropic) over several intense days of
live debugging on a real print farm.
