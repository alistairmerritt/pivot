---
layout: page
title: Getting Started
permalink: /getting-started/
---

# Getting Started

This guide walks you through everything needed to get Pivot running on a Home Assistant Voice Preview Edition device.

## What you need

- A Home Assistant Voice Preview Edition (VPE)
- Home Assistant with the ESPHome integration installed
- HACS installed in Home Assistant
- Your WiFi credentials and a generated ESPHome API encryption key

---

## Step 1 — Set up your VPE in Home Assistant

Your VPE must already be connected to your WiFi and added to Home Assistant via the ESPHome integration before you can flash Pivot firmware.

If you haven't done this yet, follow [Home Assistant's official VPE setup guide](https://www.home-assistant.io/voice_control/voice_remote_local_assistant/) first, then come back here.

If your VPE is already showing up in Home Assistant under ESPHome, you're ready to continue.

> **Your VPE's voice and listening functionality is not affected by Pivot.** Wake word detection, voice commands, and all standard VPE features continue to work exactly as before. Pivot adds a new layer of control — it does not replace or interfere with anything already set up.

---

## Step 2 — Enable Home Assistant actions

While you're in the ESPHome integration, enable this now before flashing the firmware:

1. Go to **Settings → Devices & Services → ESPHome**
2. Find your VPE and click **⚙️ Configure**
3. Enable **Allow the device to perform Home Assistant actions**

This is required for the Pivot firmware to call scripts and send events to Home Assistant.

---

## Step 3 — Flash the Pivot firmware

1. Download `home-assistant-voice.yaml` from the [pivot-firmware](https://github.com/alistairmerritt/pivot-firmware) repository
2. Open the file and fill in the substitutions block at the top:

```yaml
substitutions:
  device_name: home_assistant_voice_lounge
  device_friendly_name: Lounge VPE
  device_suffix: ha_voice_lounge
  wifi_ssid: "YourWiFiName"
  wifi_password: "YourWiFiPassword"
  api_encryption_key: "your_generated_key_here"
```

The `device_suffix` must be unique per device with no spaces or dashes. You will enter this exact value when setting up the integration. Generate an API key at [esphome.io](https://esphome.io/components/api.html#configuration-variables).

3. Flash the firmware using the **ESPHome add-on** (recommended) — go to **Settings → Add-ons → Add-on Store**, install ESPHome, open the dashboard, and flash from there. Alternatively run:
```bash
esphome run home-assistant-voice.yaml
```

4. Once flashed, **fully power cycle your VPE** — disconnect it from power, wait a few seconds, then reconnect. The device will then reconnect to Home Assistant automatically.

---

## Step 4 — Install the Pivot integration

1. In HACS, go to **Integrations → ⋮ → Custom repositories**
2. Add `https://github.com/alistairmerritt/pivot-integration`, category: **Integration**
3. Search for **Pivot** and install
4. Restart Home Assistant

---

## Step 5 — Add your device in Pivot

1. Go to **Settings → Devices & Services → Add Integration** and search for **Pivot**
2. Select your VPE from the dropdown
3. Confirm the firmware and enter the `device_suffix` you used in the firmware YAML
4. Choose a **setup mode**:
   - **Automatic** — Pivot creates and manages the required scripts and automations for you. Recommended for most users.
   - **Blueprints** — Pivot installs blueprint files. You create the automations yourself from the HA UI.
   - **Manual** — Pivot does not create any files. Use the fired events to build your own automations.
5. Optionally configure announcements — select a TTS engine and speaker to have Pivot speak the active bank name when you switch

---

## Step 6 — Assign entities to banks

1. Go to **Settings → Devices & Services → Pivot → your device → Configure**
2. Assign a Home Assistant entity to each bank
3. Save

That's it. Turn the knob to control the active bank's assigned entity. Press the button to toggle or activate it. Hold the knob and turn to switch banks.

---

## Next steps

- [Firmware reference](/pivot/firmware/) — full substitutions reference and multi-device setup
- [Integration reference](/pivot/integration/) — full entity reference, setup modes, and custom automations
- [Troubleshooting](/pivot/troubleshooting/) — common issues and fixes
