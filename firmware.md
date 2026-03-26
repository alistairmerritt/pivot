---
layout: page
title: Firmware
permalink: /firmware/
---

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

  # LED orientation — set based on how your device is mounted:
  #   6  = flat on a surface, cable facing away (LEDs start at bottom)
  #   0  = upright on a stand, cable at the bottom (LEDs start at top)
  led_offset: '6'
```

### `led_offset`

Set to `'6'` if your device is **flat on a surface** with the cable facing away from you. Set to `'0'` if your device is **upright on a stand** with the cable at the bottom. This controls which physical LED is treated as position 0 on the ring, so the bank colour indicators and gauges appear in the correct position relative to how you're looking at it.

---

### `device_suffix`

This is the most important field. It must be:
- Unique across all your Pivot devices
- Lowercase, no spaces, no dashes (underscores are fine)
- Identical to what you enter in the Pivot HA integration setup

It determines all entity IDs — for example `ha_voice_lounge` produces `number.ha_voice_lounge_active_bank`, `text.ha_voice_lounge_bank_0_entity`, etc.

---

## Multiple devices

Each VPE needs its own config file in ESPHome with a unique `device_suffix`. The recommended approach uses ESPHome's `packages:` feature — each device has a small per-device file that pulls the full shared firmware from GitHub automatically. You never have to copy or maintain separate full YAML files per device.

**Per-device config (paste into ESPHome as a new device):**

```yaml
substitutions:
  # =======================================================================
  # PIVOT DEVICE CONFIGURATION — fill in these values for each device
  # =======================================================================

  # ESPHome device name (slug, no spaces or dashes)
  device_name: home_assistant_voice_lounge

  # Friendly name shown in HA and ESPHome
  device_friendly_name: Lounge VPE

  # Pivot device suffix — unique per device, no spaces or dashes
  # Must match exactly what you enter in the Pivot integration
  device_suffix: ha_voice_lounge

  # WiFi credentials — add these lines to your ESPHome secrets.yaml:
  #   wifi_ssid: "Your Network Name"
  #   wifi_password: "Your Password"
  wifi_ssid: !secret wifi_ssid
  wifi_password: !secret wifi_password

  # API encryption key — generate a unique one per device at:
  # https://esphome.io/components/api.html#configuration-variables
  api_encryption_key: "generate-a-unique-key-here"

  # LED orientation — set based on how your device is mounted:
  #   '6'  flat on a surface, cable facing away (LEDs start at bottom)
  #   '0'  upright on a stand, cable at the bottom (LEDs start at top)
  led_offset: '6'

  # =======================================================================

packages:
  pivot:
    url: https://github.com/alistairmerritt/pivot-firmware
    ref: main
    file: home-assistant-voice.yaml
    refresh: 1d
```

A fully annotated template is available at [`devices/example.yaml`](https://github.com/alistairmerritt/pivot-firmware/blob/main/devices/example.yaml) in the firmware repo.

When a new version of Pivot firmware is released, open each device in ESPHome and click **Install** — ESPHome pulls the latest from GitHub and flashes it OTA. No manual YAML copying required.

| Device | `device_suffix` |
| --- | --- |
| Lounge VPE | `ha_voice_lounge` |
| Bedroom VPE | `ha_voice_bedroom` |
| Study VPE | `ha_voice_study` |

### Devices on stock firmware

If a VPE is currently running Nabu Casa's stock firmware:

- **If it appears in your ESPHome dashboard** (amber or green dot) — its API key and WiFi are already there. Create a new device entry using the per-device config above, then click **Install → Wirelessly**.
- **If it has never been in ESPHome** (set up via the HA onboarding UI only) — you won't have the API key, so the first flash needs to be done via USB. After that, all future updates are OTA.

---

## Updating the firmware

**HA does not notify you when a new version of Pivot firmware is available.** Updates are always manual — you initiate them from ESPHome Device Builder.

When a new firmware version is released:

1. Open **ESPHome Device Builder** in Home Assistant
2. Find the device you want to update
3. Click **Install → Wirelessly**
4. ESPHome fetches the latest firmware from GitHub, compiles it, and flashes it over WiFi

That's it — no USB, no copying YAML, no manual steps. Each device takes 2–3 minutes.

**USB is only ever needed for:**
- The very first flash on a device that has never had ESPHome on it
- Recovery if a device has lost its WiFi connection and can't be reached OTA

Once a device is running Pivot firmware and is online, every future update is OTA.

> **Note:** HA's Updates page (Settings → Updates) will notify you of new Pivot **integration** releases via HACS, but not firmware changes. Check the [changelog](/pivot/changelog/) to see what's changed.

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

## Safety and rollback

Pivot firmware is based on the official Home Assistant Voice PE ESPHome configuration and has been tested, but as with any custom firmware there is a small element of risk. If anything goes wrong, you can always restore the original stock firmware by visiting [esphome.github.io/home-assistant-voice-pe](https://esphome.github.io/home-assistant-voice-pe/) and flashing it from your browser — no tools or ESPHome installation required.

---

## Flashing

The easiest way to flash is using the **ESPHome Device Builder application** (formerly called the ESPHome add-on). Install it via **Settings → Applications → Add Application** and search for ESPHome Device Builder.

**Taking control of your VPE in ESPHome Device Builder**

Taking control simply imports the device into ESPHome Device Builder so you can manage and flash its configuration. Once again, this can be undone at any time by restoring the original stock firmware at [esphome.github.io/home-assistant-voice-pe](https://esphome.github.io/home-assistant-voice-pe/).

When you open ESPHome Device Builder, your VPE may appear hidden under Discovered Devices — click **Show** in the top right corner if you don't see it. Click **Take Control**, give it a name, then click **Install**. This may take 3–5 minutes over Wi-Fi.

Once you have taken control, replace the stock YAML with your per-device config (from [`devices/example.yaml`](https://github.com/alistairmerritt/pivot-firmware/blob/main/devices/example.yaml)), then click **Install**. Do not paste the full `home-assistant-voice.yaml` — the per-device config fetches it from GitHub automatically.

**Flashing via USB**

Use a good quality USB cable for the initial flash rather than OTA — it is more reliable. OTA is fine for subsequent updates once the device is running Pivot firmware.

> **Tip:** There is a small switch inside the VPE case labelled **USB SELECT** with two positions: **ESP32** and **XU316**. It should be in the **ESP32** position by default, and your computer should detect the USB port it's connected to. If your device is not being detected when connected via USB, open the case and check this switch. Follow [Step 1 of the Nabu Casa disassembly guide](https://support.nabucasa.com/hc/en-us/articles/25938306296605-Disassembling-the-enclosure-of-Home-Assistant-Voice-Preview-Edition) to access it — you do not need to go further than Step 1 unless you have a custom case.

Click **Install** in the top right corner of ESPHome Device Builder. This could take 5–10 minutes. ESPHome will tell you if and why it fails.

> **After flashing for the first time**, disconnect your VPE from power completely, wait a few seconds, then reconnect. The device will then reconnect to Home Assistant automatically.

Via CLI (if you have ESPHome installed locally):
```bash
esphome run home-assistant-voice.yaml
```

---

## Before you flash — note these down

Before flashing, make a note of these two values somewhere safe. You will need them during setup and potentially again later:

| Value | Where it's used |
| --- | --- |
| `device_suffix` | Required when adding your device in the Pivot integration |
| `api_encryption_key` | Required if you ever need to re-add the device to Home Assistant |

---

## Differences from stock firmware

Pivot firmware is based on the official Home Assistant Voice PE ESPHome configuration with the following additions and changes:

- `device_suffix` substitution used for all Pivot entity IDs
- 12 colour globals (`bank_r/g/b_0-3`) with default RGB values per bank
- 4 `text_sensor` entries watching the bank colour text entities in HA
- LED animation lambdas read from colour globals
- Active bank sent as `bank + 1` to HA (1-based) to match the integration's number entity range
- Triple press retains sound; double press sound removed
- Control Mode added: hold+turn switches bank, turn adjusts the active bank's assigned entity value
- Hold+turn in Normal mode retains stock behaviour — changes the LED ring colour (hue cycling)
- Secondary scroll behaviour removed
- Passive banks (scene, script, switch, input_boolean) show no gauge — LEDs turn off rather than showing a misleading full ring. Bank Indicator still fires normally during bank switching.

All standard VPE functionality (voice assistant, wake word, mute button, LED colour change, etc.) remains intact.
