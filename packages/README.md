# packages/

The SpoolTap package — `spooltap.yaml` — installs here.

**Install:** copy `spooltap.yaml` into your Home Assistant `config/packages/`
directory and make sure `configuration.yaml` includes:

```yaml
homeassistant:
  packages: !include_dir_named packages/
```

Every install-specific value (hostnames, ids, your AMS sensor names, your
phone) is read from your `secrets.yaml` — so this file is the same for
everyone. Copy `secrets.example.yaml` → `secrets.yaml`, fill it in, and follow
[../SETUP.md](../SETUP.md).

> `spooltap.yaml` reads 100% from `secrets.yaml` — no install values are baked
> in. The one exception is the AMS tray-sensor *trigger* lists (Home Assistant
> forbids templating those), marked `TOPOLOGY` — match them to your printer per
> [../SETUP.md](../SETUP.md).
