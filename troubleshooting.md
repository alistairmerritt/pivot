---
layout: page
title: Troubleshooting
permalink: /troubleshooting/
---

# Troubleshooting

---

## Nothing happens after installing everything

The most common cause is that **Allow the device to perform Home Assistant actions** is not enabled. Without this, the firmware cannot call scripts or send events to Home Assistant.

Go to **Settings → Devices & Services → ESPHome → your VPE → ⚙️ Configure** and make sure it is ticked.

---

## The device won't connect to Home Assistant after flashing

After flashing the Pivot firmware for the first time, you need to **fully power cycle your VPE** — disconnect it from power completely, wait a few seconds, then reconnect. A simple restart is not always enough. The device should then reconnect and appear in Home Assistant automatically.

---

## Home Assistant asks for an encryption key that doesn't match

The API encryption key in your firmware YAML must match exactly what Home Assistant expects. If you reused an existing key or generated a new one after the device was already added to HA, they will be out of sync.

To fix: remove the device from the ESPHome integration in HA, update the key in your firmware YAML to match, reflash, then re-add the device.

> **Tip:** When setting up a new device, note down your `device_suffix` and `api_encryption_key` somewhere safe before flashing. You will need both — the suffix when adding the Pivot integration, and the key if you ever need to re-add the device to Home Assistant.

---

## The integration says "Invalid handler specified" when I try to add it

This usually means HA loaded stale cached Python files. Fix:

1. In SSH or Terminal: `rm -rf /config/custom_components/pivot/__pycache__`
2. Restart Home Assistant fully (Settings → System → Restart)
3. Try adding the integration again

---

## My entities are all disabled

If the Pivot config entry was previously disabled, the entities will be disabled too. Fix via SSH:

```bash
sed -i 's/"disabled_by":"user","discovery_keys":{},"domain":"pivot"/"disabled_by":null,"discovery_keys":{},"domain":"pivot"/g' /config/.storage/core.config_entries
ha core restart
```

---

## The knob turns but nothing happens

Check the following:

1. The bank has an entity assigned — go to Settings → Devices & Services → Pivot → your device → **Configure**
2. The entity is a supported domain (light, fan, media_player, climate, cover)
3. Control Mode is on — check `switch.{suffix}_control_mode` is enabled
4. In Automatic mode, the bank toggle script exists — check `grep -i pivot /config/scripts.yaml`

---

## The button press does nothing

In Automatic mode, `single_press` requires the bank toggle script to exist. Check:

```bash
ls /config/pivot_*_bank_toggle.yaml
grep -i pivot /config/scripts.yaml
```

If the files are missing, go to the integration's Configure menu and save — this will re-trigger the file write.

---

## Announcements are not working

1. Check `switch.{suffix}_announcements` is on
2. Check a TTS entity and media player are configured — Settings → Devices & Services → Pivot → your device → **Configure**
3. In Automatic mode, check the automation file exists: `ls /config/pivot_*_announcements.yaml`
4. Check the TTS entity is working by testing it independently in Developer Tools → Services


---

## The device suffix mismatch — entities have wrong IDs

The `device_suffix` in your firmware YAML must match exactly what you entered in the integration setup. If they don't match, the firmware and integration will create mismatched entity IDs and won't communicate.

To fix: reflash the firmware with the correct `device_suffix`, or delete and re-add the integration entry with the matching suffix.

---

## GitHub or HACS shows an old version

Each time you push updates, make sure you:
1. Update the version in `manifest.json`
2. Create a new git tag matching the version
3. Push the tag: `git push origin vX.X.X`
4. Create a GitHub release for that tag

HACS uses GitHub releases to detect updates — without a release, HACS won't offer the update.
