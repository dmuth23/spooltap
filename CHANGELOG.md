# Changelog

All notable changes to SpoolTap are recorded here.

## [Unreleased]

### Added
- Initial public scaffold: README, SETUP guide, `secrets.example.yaml`,
  `.gitignore`, MIT license, and the `packages/` + `dashboards/` layout.

- `packages/spooltap.yaml` — the full system, parameterized: reads 100% from
  `secrets.yaml`, no install values baked in (except the documented `TOPOLOGY`
  trigger lists). Adversarially reviewed (3 independent passes) before release.
- `input_select.spooltap_mode` is now defined in the package (was a UI-created
  helper) so the dashboard's mode switcher works on a fresh install.
- `dashboards/spooltap.yaml` — the SpoolTap control surface (Assign / Bind /
  Modify / Inventory), exported and parameterized. The one install-specific
  reference (print-status sensor) reads from `secrets.yaml` like everything
  else. Includes an Apply Config button on the Inventory tab.
- `dashboards/README.md` — dashboard install (YAML-mode + paste alternative),
  HACS card prerequisites, and the storage-vs-full-YAML resource rules.

### Fixed
- Reconciler one-spool-two-slots oscillation. If a spool was registered in two
  Bambuddy slot rows at once (an AMS slot plus a stale `external` ghost), the
  auto-heal loop flapped Spoolman's `nfc_active_tray` between the two slots
  every cycle, so the printer slot kept dropping to Empty and a manual change
  "wouldn't stick." The reconciler now enforces one-spool-one-slot before the
  diff: it keeps the AMS slot over an `external` ghost (chosen structurally by
  `ams_id`, not the unreliable tray sensor), converges Bambuddy + Spoolman to
  that one slot, and freezes with a warning on a genuinely ambiguous spool in
  two AMS slots rather than guessing. Fully unattended; validated by 5/5
  behavioral acceptance cases in a sandbox HA.
