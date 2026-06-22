# dashboards/

`spooltap.yaml` is the complete SpoolTap control surface — the Assign / Bind /
Modify / Inventory workbench, plus a Notifications tab for muting/unmuting the
phone-alert categories. It is **part of SpoolTap, not an optional extra**:
the package and the dashboard are designed together.

The file is fully parameterized. The only install-specific value it ever
touches (your printer's print-status sensor) is read through the same
`secrets.yaml` helper the package uses — so this file is identical for everyone.

---

## Prerequisites (do these first)

1. **Install the package and restart HA.** The dashboard references dozens of
   `input_*`, `script.*`, and `sensor.*` entities that only exist once
   `packages/spooltap.yaml` is installed and HA has restarted. Install the
   dashboard *after* the package or you'll see "entity not found" placeholders.
   See [../SETUP.md](../SETUP.md) Part 1.

2. **Install the three HACS frontend cards** (HACS → Frontend → search):
   - [Mushroom](https://github.com/piitaya/lovelace-mushroom) — the mode
     switcher and status chips
   - [card-mod](https://github.com/thomasloven/lovelace-card-mod) — the
     lit/greyed active-tab styling
   - [auto-entities](https://github.com/thomasloven/lovelace-auto-entities) —
     the live spool grid on the Inventory tab

   Without these you'll get red **"Custom element doesn't exist"** errors where
   those cards should be. This is the single most common reason a borrowed
   dashboard "doesn't work" — don't skip it.

---

## Install — recommended (YAML-mode dashboard)

This keeps the dashboard as a file you can re-pull when SpoolTap updates, and
adds its own entry in your sidebar without touching any of your existing
dashboards.

1. Copy `spooltap.yaml` into your HA config directory, e.g.
   `/config/dashboards/spooltap.yaml`.

2. Add this to your `configuration.yaml` (merge into your existing `lovelace:`
   block if you have one — otherwise add the whole thing):

   ```yaml
   lovelace:
     dashboards:
       spooltap-dashboard:          # the URL slug — MUST contain a hyphen
         mode: yaml
         title: SpoolTap
         icon: mdi:nfc-variant
         show_in_sidebar: true
         filename: dashboards/spooltap.yaml
   ```

   > Home Assistant **requires the dashboard key to contain a `-`** —
   > `spooltap-dashboard` is fine, `spooltap` alone is rejected at startup.

3. **Restart Home Assistant** (newly-registered YAML dashboards need a restart,
   not just a reload). "SpoolTap" now appears in your sidebar.

### Resources — does it "just work"?

The dashboard needs the three HACS cards above to be *registered as Lovelace
resources*. Whether that's automatic depends on how you run Lovelace:

- **Default / storage mode** (you manage dashboards through the UI — the common
  case, and true even after adding the `dashboards:` block above): HACS
  registers those cards as resources for you, and they load for the SpoolTap
  YAML dashboard automatically. **Nothing to do.**

- **Full YAML-mode Lovelace** (you have `lovelace: mode: yaml` at the top
  level): storage-registered resources are ignored — you must list them
  yourself under `lovelace: resources:`:

  ```yaml
  lovelace:
    mode: yaml
    resources:
      - url: /hacsfiles/lovelace-mushroom/mushroom.js
        type: module
      - url: /hacsfiles/lovelace-card-mod/card-mod.js
        type: module
      - url: /hacsfiles/lovelace-auto-entities/auto-entities.js
        type: module
    dashboards:
      spooltap-dashboard:
        mode: yaml
        title: SpoolTap
        icon: mdi:nfc-variant
        show_in_sidebar: true
        filename: dashboards/spooltap.yaml
  ```

  The exact `/hacsfiles/...` paths depend on your HACS install — copy them from
  **Settings → Dashboards → ⋮ → Resources**.

---

## Install — alternative (no configuration.yaml edit)

If you'd rather not edit `configuration.yaml`:

1. **Settings → Dashboards → Add Dashboard → New dashboard from scratch.** Give
   it a title and icon.
2. Open it, then **⋮ → Edit Dashboard → ⋮ → Raw configuration editor.**
3. Paste the contents of `spooltap.yaml` and **Save.** (You can keep or drop the
   top-level `title:` line — you already named the dashboard in step 1; the
   `views:` block is what matters.)

Because this is a storage-mode dashboard, the HACS resources auto-load. The
trade-off: updates mean re-pasting, and it isn't a file you version-control.

---

## After install

The dashboard opens on the **Assign** tab. The four buttons across the top
switch modes (they drive `input_select.spooltap_mode`, which ships in the
package). The **Inventory** tab has a **Configuration → Apply Config** button —
tap it once after first install, and again any time you edit `secrets.yaml`.

> **One honest caveat:** Home Assistant renders custom (HACS) cards in the
> browser, so a config can pass every structural check and still need a real
> browser to confirm the custom cards paint. After installing, open the
> dashboard once: if every tab renders and the mode buttons light up, you're
> done. If you see "Custom element doesn't exist", a HACS card above is missing
> — install it and refresh.
