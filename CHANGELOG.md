# Changelog

All notable changes to SpoolTap are recorded here.

## [Unreleased]

### Added
- Initial public scaffold: README, SETUP guide, `secrets.example.yaml`,
  `.gitignore`, MIT license, and the `packages/` + `dashboards/` layout.

- `packages/spooltap.yaml` — the full system, parameterized: reads 100% from
  `secrets.yaml`, no install values baked in (except the documented `TOPOLOGY`
  trigger lists). Adversarially reviewed (3 independent passes) before release.

### In progress
- Dashboard YAML export.
