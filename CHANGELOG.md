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
