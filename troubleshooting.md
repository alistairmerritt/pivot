---
layout: page
title: Troubleshooting
permalink: /troubleshooting/
---

---

## The device won't appear when connected via USB for flashing

First make sure you are using a good quality USB cable — some cables are charge-only and do not support data transfer. Try a different cable if you have one.

> **Tip:** There is a small switch inside the VPE case labelled **USB SELECT** with two positions: **ESP32** and **XU316**. It should be in the **ESP32** position by default. If your device is still not being detected, open the case and check this switch. Follow [Step 1 of the Nabu Casa disassembly guide](https://support.nabucasa.com/hc/en-us/articles/25938306296605-Disassembling-the-enclosure-of-Home-Assistant-Voice-Preview-Edition) to access it — you do not need to go further than Step 1 unless you have a custom case.

---

## Something went wrong with the firmware — how do I recover?

You can always restore the original stock Home Assistant Voice PE firmware by visiting [esphome.github.io/home-assistant-voice-pe](https://esphome.github.io/home-assistant-voice-pe/) in your browser and following the on-screen instructions. No tools or ESPHome installation required.

---

## Nothing happens after installing everything

Start here before anything else:

1. Go to **Settings → Devices & Services → ESPHome → your VPE → ⚙️ Configure** and confirm **Allow the device to perform Home Assistant actions** is ticked. Without this, the firmware cannot call scripts or send events to Home Assistant.
2. Go to **Settings → Devices & Services → Pivot → your device → Configure** and confirm at least one bank has an entity assigned.
3. Make sure you are in **Control Mode** — double press the button to toggle it on. The LED ring should change to show the active bank colour.

---

## My VPE doesn't appear in the ESPHome application

If your device isn't showing up in ESPHome, you may need to take control of it first. Follow the [ESPHome getting started guide](https://esphome.io/guides/getting_started_hassio/) which walks through adding a device to the ESPHome application.

---

## My VPE just has revolving blue lights after installing the firmware

The revolving blue light pattern means the device is trying to connect but hasn't succeeded yet. There are two likely causes:

**Encryption key mismatch** — check **Settings → Notifications** in Home Assistant. If there is an alert asking you to reconfigure a device, open it and enter the `api_encryption_key` from your firmware YAML's substitutions block.

**WiFi credentials issue** — if there is no notification in Home Assistant, the device may not be reaching your network at all. Open your firmware YAML in ESPHome and double-check that your `wifi_ssid` and `wifi_password` are correct, then reflash.

---

## The device won't connect to Home Assistant after flashing

After flashing the Pivot firmware for the first time, **fully power cycle your VPE** — disconnect it from power completely, wait a few seconds, then reconnect. A simple restart is not always enough. The device should then reconnect and appear in Home Assistant automatically.

---

## Home Assistant asks for an encryption key

When Home Assistant asks for an encryption key, open your firmware YAML file and copy the `api_encryption_key` value from the substitutions block — paste that directly into the box Home Assistant is showing you.

---

## The integration says "Invalid handler specified" when I try to add it

This usually means HA has loaded stale cached files. Fix it from the UI:

1. Go to **Settings → System → Restart** and do a full restart (not just a quick reload)
2. Try adding the integration again

If it still fails after a full restart, the cache may need to be cleared manually via SSH:
```bash
rm -rf /config/custom_components/pivot/__pycache__
```
Then restart Home Assistant again.

---

## My entities are all disabled

The simplest fix is to remove and re-add the Pivot integration from **Settings → Devices & Services** — your bank assignments are stored in the text entities and will be restored automatically.

If you would prefer not to re-add the integration, you can re-enable the entities manually:

1. Go to **Settings → Devices & Services → Pivot → your device**
2. Click on each disabled entity and use the toggle to re-enable it

If entities are disabled at the config entry level rather than individually, that requires editing an internal HA storage file via SSH. Only proceed if you are comfortable with that — a clean reinstall is safer.

---

## The knob turns but nothing happens

Work through these in order:

1. **Check bank assignment** — go to **Settings → Devices & Services → Pivot → your device → Configure** and confirm the active bank has an entity assigned.
2. **Check the entity domain** — the entity must be a supported type: light, fan, media player, climate, or cover. Scenes and scripts are passive (knob does nothing, button only).
3. **Check Control Mode is on** — go to **Settings → Devices & Services → Pivot → your device** and check that the **Control Mode** switch is on. You can also toggle it with a double press on the button.
4. **Check the bank toggle script exists** (Automatic mode only) — go to **Developer Tools → Template** and enter `{% raw %}{{ states('script.{suffix}_bank_toggle') }}{% endraw %}`. If it returns `unknown`, the script is missing. Go to **Configure** on the integration and save to re-trigger the file write.

---

## The button press does nothing

1. **Check bank assignment** — go to **Settings → Devices & Services → Pivot → your device → Configure** and confirm the active bank has an entity assigned.
2. **Check the script entity ID** (Automatic mode only) — go to **Settings → Devices & Services → Pivot → your device** and look for a script entity. Its entity ID must be `script.{suffix}_bank_toggle` where `{suffix}` matches your `device_suffix` exactly. If the ID looks wrong, the suffix in your firmware YAML may not match what you entered during integration setup.
3. **Check the script exists** — go to **Settings → Scripts** and search for your device suffix. If it's not there, go to **Configure** on the integration and save to re-trigger the file write.

---

## The button press stopped working after editing scripts.yaml

If you deleted or modified `pivot/pivot_{suffix}_bank_toggle.yaml` or the `!include` line Pivot added to your `scripts.yaml`, the bank toggle script will stop working.

The quickest fix is to go to **Settings → Devices & Services → Pivot → your device → Configure** and save — this will recreate the file and re-add the `!include` line if it is missing.

> **Warning:** If the `!include` line appears more than once in `scripts.yaml`, HA will throw a duplicate key error and fail to load scripts entirely. Go to **Studio Code Server** or another file editor and remove the duplicate line, then reload scripts from **Developer Tools → YAML → Scripts**. Once HA is running again, Pivot will automatically correct any remaining inconsistencies on its next load — you do not need to do anything further.

---

## Announcements are not working

1. Go to **Settings → Devices & Services → Pivot → your device** and check that the **Announcements** switch is on.
2. Go to **Configure** on the integration and confirm a text-to-speech service and speaker are selected.
3. Test your TTS service independently — go to **Developer Tools → Actions**, find `tts.speak`, select your TTS entity and media player, and send a test message. If this doesn't work, the issue is with your TTS setup rather than Pivot.
4. In Automatic mode, check the announcements automation exists — go to **Settings → Automations** and search for your device suffix. If it's missing, go to **Configure** on the integration and save to re-trigger the file write.

---

## The timer blueprint triggers when I turn the knob on a bank with a real entity

This happens when the timer blueprint is set up on a bank that also has a real entity assigned. The blueprint now requires the bank's entity field to be set to the reserved keyword `timer` before it will respond — this prevents it from interfering with normally-assigned banks.

To fix: go to **Settings → Devices & Services → Pivot → your device → Configure**, set the bank entity for your timer bank to `timer` (lowercase), and save. If the timer bank has a real entity assigned, remove it first.

---

## The timer gauge (LED ring) does not update while the timer is running

1. Make sure all three timer entities are enabled: `number.{suffix}_timer_duration`, `select.{suffix}_timer_state`, and `text.{suffix}_timer_end`. All three must be enabled for the blueprint to work.
2. Check the bank entity for the timer bank is set to `timer` (not left blank or set to a real entity).
3. Confirm the timer automation is enabled — go to **Settings → Automations** and check it is toggled on.
4. The gauge updates every 30 seconds, not continuously. A brief delay before the first update is normal.

---

## ESPHome build fails with "'.' is an invalid character for names"

This happens when the `device_name` substitution in your device YAML includes the `.yaml` file extension. ESPHome device names cannot contain dots.

Open your device YAML in ESPHome and find the `substitutions:` block. The `device_name` value should be the name only — no extension, lowercase, using hyphens or underscores:

```yaml
substitutions:
  device_name: ha-voice-kitchen        # ✓ correct
  device_name: ha-voice-kitchen.yaml   # ✗ causes this error
```

Remove the `.yaml` suffix, save, and retry the build.

---

## The device suffix mismatch — entities have wrong IDs

The `device_suffix` in your firmware YAML must match exactly what you entered in the integration setup. If they don't match, the firmware and integration will use mismatched entity IDs and won't communicate.

To check: go to **Settings → Devices & Services → Pivot → your device** and look at the entity IDs. They should all start with your `device_suffix`. If they don't match what the firmware expects, the easiest fix is to remove and re-add the Pivot integration using the correct suffix.

---

## Restoring from a Pivot backup

Before Pivot appends its `!include` line to your `scripts.yaml` or `automations.yaml` for the first time, it automatically creates a backup:

- `scripts.yaml.pivot_backup`
- `automations.yaml.pivot_backup`

These backups reflect the state of your files before Pivot ever touched them. To restore via SSH:

```bash
cp /config/scripts.yaml.pivot_backup /config/scripts.yaml
cp /config/automations.yaml.pivot_backup /config/automations.yaml
```
