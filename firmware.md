---
layout: page
title: Firmware
permalink: /firmware/
---

# Pivot Firmware

Pivot firmware is a custom ESPHome configuration for the Home Assistant Voice Preview Edition (VPE) that adds four colour-coded control banks, knob-turn handling, LED feedback, and event firing to Home Assistant.

The firmware source is at [alistairmerritt/pivot-firmware](https://github.com/alistairmerritt/pivot-firmware).

---

## Substitutions

All device-specific configuration lives in the substitutions block at the top of the YAML. You only need to edit this section when setting up a new device.

```yaml
substitutions:
  # ESPHome device name (slug, no spaces or dashes)
  device_name: home_assistant_voice_lounge

  # Friendly name shown in HA and ESPHome
  device_friendly_name: Lounge VPE

  # Pivot device suffix — unique per device, no spaces or dashes
  # Must match exactly what you enter in the Pivot integration
  device_suffix: ha_voice_lounge

  # WiFi credentials
  wifi_ssid: "YourWiFiName"
  wifi_password: "YourWiFiPassword"

  # API encryption key — generate one at:
  # https://esphome.io/components/api.html#configuration-variables
  api_encryption_key: "your_generated_key_here"
```

### `device_suffix`

This is the most important field. It must be:
- Unique across all your Pivot devices
- Lowercase, no spaces, no dashes (underscores are fine)
- Identical to what you enter in the Pivot HA integration setup

It determines all entity IDs — for example `ha_voice_lounge` produces `number.ha_voice_lounge_active_bank`, `text.ha_voice_lounge_bank_0_entity`, etc.

---

## Multiple devices

Flash a separate copy of the firmware for each VPE with a unique `device_suffix` for each one. The integration will create a completely separate set of entities for each device.

| Device | `device_suffix` |
| --- | --- |
| Lounge VPE | `ha_voice_lounge` |
| Bedroom VPE | `ha_voice_bedroom` |
| Study VPE | `ha_voice_study` |

---

## Bank colours

The LED ring colour for each bank is controlled by text entities created by the integration. Default colours:

| Bank | Default Colour |
| --- | --- |
| 1 | Blue `#2889FF` |
| 2 | Orange `#FF7D19` |
| 3 | Green `#97FF3D` |
| 4 | Purple `#C800FF` |

You can change bank colours from within Home Assistant using the light entities the integration creates for each bank (`light.{suffix}_bank_1_color_light` etc.).

---

## Flashing

The easiest way to flash is using the **ESPHome add-on for Home Assistant**. If you haven't installed it yet, go to **Settings → Add-ons → Add-on Store** and search for ESPHome. Once installed, open the ESPHome dashboard, create a new device entry, paste in your edited YAML, and flash from there — all from within Home Assistant.

Via CLI (if you have ESPHome installed locally):
```bash
esphome run home-assistant-voice.yaml
```

For subsequent updates you can use OTA (over-the-air) flashing — no USB connection needed once the device is on your network.

> **After flashing for the first time**, disconnect your VPE from power completely, wait a few seconds, then reconnect. This ensures the device boots cleanly with the new firmware and reconnects to Home Assistant properly.

---

## Before you flash — note these down

Before flashing, make a note of these two values somewhere safe. You will need them during setup and potentially again later:

| Value | Where it's used |
| --- | --- |
| `device_suffix` | Required when adding your device in the Pivot integration |
| `api_encryption_key` | Required if you ever need to re-add the device to Home Assistant |

---

## Differences from stock firmware

Pivot firmware is based on the official Home Assistant Voice PE ESPHome configuration with the following additions:

- `device_suffix` substitution used for all Pivot entity IDs
- 12 colour globals (`bank_r/g/b_0-3`) with default RGB values per bank
- 4 `text_sensor` entries watching the bank colour text entities in HA
- LED animation lambdas read from colour globals
- Active bank sent as `bank + 1` to HA (1-based) to match the integration's number entity range
- Triple press retains sound; double press sound removed

All standard VPE functionality (voice assistant, wake word, mute button, etc.) remains intact.
