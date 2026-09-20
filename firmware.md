---
layout: page
title: Firmware
permalink: /firmware/
---

Pivot firmware is a custom ESPHome configuration for the Home Assistant Voice Preview Edition (VPE) that adds four colour-coded control banks, knob-turn handling, LED feedback, and event firing to Home Assistant.

The firmware source is at [alistairmerritt/pivot-firmware](https://github.com/alistairmerritt/pivot-firmware).

> **Installing custom firmware is safe and reversible.** Pivot firmware is based on the official Home Assistant Voice PE firmware and has been tested extensively, but as with any custom firmware there is a small element of risk. If anything goes wrong, you can always restore the original stock firmware by visiting [esphome.github.io/home-assistant-voice-pe](https://esphome.github.io/home-assistant-voice-pe/) and flashing it from your browser – no tools required.

---

## Substitutions

All device-specific configuration lives in the substitutions block at the top of the YAML. You only need to edit this section when setting up a new device.

```yaml
substitutions:
  # ESPHome device name (use dashes not underscores)
  device_name: home-assistant-voice-lounge

  # Friendly name shown in HA and ESPHome
  device_friendly_name: Lounge VPE

  # Pivot device suffix – unique per device, no spaces or dashes
  # Must match exactly what you enter in the Pivot integration
  device_suffix: ha_voice_lounge

  # WiFi credentials
  wifi_ssid: "YourWiFiName"
  wifi_password: "YourWiFiPassword"

  # API encryption key – generate one at:
  # https://esphome.io/components/api.html#configuration-variables
  api_encryption_key: "your_generated_key_here"

  # OTA update password – REQUIRED, min 12 characters. One password can be
  # shared by all your Pivot devices. Generate with:  openssl rand -hex 16
  ota_password: "your_generated_ota_password"

  # LED orientation – set based on how your device is mounted:
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

It determines all entity IDs – for example `ha_voice_lounge` produces `number.ha_voice_lounge_active_bank`, `text.ha_voice_lounge_bank_1_entity`, etc.

---

## Multiple devices

Each VPE needs its own config file in ESPHome with a unique `device_suffix`. The recommended approach uses ESPHome's `packages:` feature – each device has a small per-device file that pulls the full shared firmware from GitHub automatically. You never have to copy or maintain separate full YAML files per device.

**Per-device config (paste into ESPHome as a new device):**

```yaml
substitutions:
  # =======================================================================
  # PIVOT DEVICE CONFIGURATION – fill in these values for each device
  # =======================================================================

  # ESPHome device name (use dashes not underscores)
  device_name: home-assistant-voice-lounge

  # Friendly name shown in HA and ESPHome
  device_friendly_name: Lounge VPE

  # Pivot device suffix – unique per device, no spaces or dashes
  # Must match exactly what you enter in the Pivot integration
  device_suffix: ha_voice_lounge

  # WiFi credentials – add these lines to your ESPHome secrets.yaml:
  #   wifi_ssid: "Your Network Name"
  #   wifi_password: "Your Password"
  wifi_ssid: !secret wifi_ssid
  wifi_password: !secret wifi_password

  # API encryption key – generate a unique one per device at:
  # https://esphome.io/components/api.html#configuration-variables
  api_encryption_key: "generate-a-unique-key-here"

  # OTA update password – REQUIRED, min 12 characters. One password can be
  # shared by all your Pivot devices. Generate with `openssl rand -hex 16`
  # and add it to secrets.yaml:
  #   pivot_ota_password: "a1b2c3d4e5f6a7b8c9d0e1f2"
  ota_password: !secret pivot_ota_password

  # LED orientation – set based on how your device is mounted:
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

When a new version of Pivot firmware is released, open each device in ESPHome and click **Install → Wirelessly** – ESPHome pulls the latest from GitHub and flashes it OTA. No manual YAML changes required.

| Device | `device_suffix` |
| --- | --- |
| Lounge VPE | `ha_voice_lounge` |
| Bedroom VPE | `ha_voice_bedroom` |
| Study VPE | `ha_voice_study` |

### Devices on stock firmware

If a VPE is currently running Nabu Casa's stock firmware:

- **If it appears in your ESPHome dashboard** (amber or green dot) – its API key and WiFi are already there. Create a new device entry using the per-device config above, then click **Install → Wirelessly**.
- **If it has never been in ESPHome** (set up via the HA onboarding UI only) – you won't have the API key, so the first flash needs to be done via USB. After that, all future updates are OTA.

---

## Updating the firmware

**HA does not notify you when a new version of Pivot firmware is available.** Updates are always manual – you initiate them from ESPHome Device Builder.

When a new firmware version is released:

1. Open **ESPHome Device Builder** in Home Assistant
2. Find the device you want to update
3. Click **Install → Wirelessly**
4. ESPHome fetches the latest firmware from GitHub, compiles it, and flashes it over WiFi

That's it – no USB, no copying YAML, no manual steps. Each device takes 2–3 minutes.

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

You can change bank colours from within Home Assistant using the light entities the integration creates for each bank (`light.{device_suffix}_bank_1_color_light` etc.).

---

## What the firmware does and does not do

Pivot firmware is built on the official Home Assistant Voice PE configuration, with Pivot's controls added on top. Compared with the upstream file it is based on, **1,338 lines have been added, 146 changed, and no upstream blocks removed**.

Most of those additions are what make Pivot work: the four-bank logic, LED ring behaviour, and handling for the dial and button.

### The voice and audio path is unchanged

Pivot does not change how the microphone or voice pipeline works.

Wake word detection still runs locally on the Voice PE using `micro_wake_word`, and audio is only streamed to Home Assistant after a wake word is detected or the button is pressed. It uses the same ESPHome connection as the official firmware.

Pivot adds behaviour around the LEDs, dial and button. It does not replace or reroute the audio path.

### Nothing extra leaves your network

Pivot uses the same core network components as the upstream firmware: the ESPHome API, OTA updates and Wi-Fi.

There is no `mqtt:`, `http_request:` or `web_server:` configuration, and Pivot does not add any cloud service, telemetry or analytics.

The only network audio functionality is the same `audio_http` media player used by the stock firmware for TTS and media playback from Home Assistant.

### Connections are authenticated

The connection to Home Assistant is protected using your `api_encryption_key`, and wireless firmware updates require your `ota_password`.

More detail is available in [SECURITY.md](https://github.com/alistairmerritt/pivot-firmware/blob/main/SECURITY.md).

### One external component, pinned to a specific version

Pivot uses the `voice_kit` component and device sounds from the official [esphome/home-assistant-voice-pe](https://github.com/esphome/home-assistant-voice-pe) repository.

They are pinned to commit `0579e7b` from 7 July 2026. This means the build does not silently start using a newer upstream version — the same configuration continues to build against the same known version.

Neither the component nor the device sounds have changed upstream since that commit, as of 30th September 2026.

Changes to the upstream Voice PE configuration are reviewed and brought into Pivot individually rather than being pulled in automatically. The most recent example was the updated LED ring timings included in Pivot firmware v0.0.28.

### You can check it yourself

The firmware is a single readable YAML file, so you can compare it directly with the official Home Assistant Voice PE configuration:

```bash
curl -O https://raw.githubusercontent.com/alistairmerritt/pivot-firmware/main/home-assistant-voice.yaml
curl -o upstream.yaml https://raw.githubusercontent.com/esphome/home-assistant-voice-pe/0579e7b9d8504264719c593474c85447253c9dc1/home-assistant-voice.yaml
diff upstream.yaml home-assistant-voice.yaml
```

For the Home Assistant side — including what the Pivot integration creates, reads and writes — see the [Architecture](/pivot/architecture/) page.

---

## Safety and rollback

Pivot firmware is based on the official Home Assistant Voice PE ESPHome configuration and is designed to be reversible. As with any custom firmware, there are a few trade-offs to be aware of: the first flash may require USB, updates are manual through ESPHome, and upstream VPE features or fixes may not appear in Pivot immediately. In uncommon cases, a flash or update may need to be retried, or the device may need to be reflashed or returned to stock firmware. If anything goes wrong, or if you simply change your mind, you can restore the original stock firmware from your browser at any time via the official [Home Assistant Voice PE recovery page](https://esphome.github.io/home-assistant-voice-pe/), completely removing Pivot firmware from your VPE.

### Updates and pinning

The per-device config uses `ref: main`, which means clicking **Install** always gives you the latest firmware from the repository. `refresh: 1d` controls ESPHome's local cache of the downloaded source – it does **not** auto-flash. Your device only ever updates when you click **Install** in ESPHome Device Builder. Nothing happens silently or automatically.

If you want to pin to a specific version, you can replace `ref: main` with a version tag (e.g. `ref: v0.0.23`). Available tags are listed on the [pivot-firmware](https://github.com/alistairmerritt/pivot-firmware) repository on GitHub.

---

## Flashing

> **Requires ESPHome Device Builder 2026.5.0 or later.** Pivot firmware depends on components that were merged into ESPHome core in this release. Older versions will fail to compile with a confusing error. Update the ESPHome Device Builder add-on before flashing.

The easiest way to flash is using the **ESPHome Device Builder application** (formerly called the ESPHome add-on). Install it via **Settings → Applications → Add Application** and search for ESPHome Device Builder.

**Taking control of your VPE in ESPHome Device Builder**

Taking control simply imports the device into ESPHome Device Builder so you can manage and flash its configuration. Once again, this can be undone at any time by restoring the original stock firmware at [esphome.github.io/home-assistant-voice-pe](https://esphome.github.io/home-assistant-voice-pe/).

When you open ESPHome Device Builder, your VPE may appear hidden under Discovered Devices – click **Show** in the top right corner if you don't see it. Click **Take Control**, give it a name, then click **Install**. This may take 3–5 minutes over Wi-Fi.

Once you have taken control, replace the stock YAML with your per-device config (from [`devices/example.yaml`](https://github.com/alistairmerritt/pivot-firmware/blob/main/devices/example.yaml)), then click **Install**. Do not paste the full `home-assistant-voice.yaml` – the per-device config fetches it from GitHub automatically.

**Flashing via USB**

Use a good quality USB cable for the initial flash rather than OTA – it is more reliable. OTA is fine for subsequent updates once the device is running Pivot firmware.

> **Tip:** There is a small switch inside the VPE case labelled **USB SELECT** with two positions: **ESP32** and **XU316**. It should be in the **ESP32** position by default, and your computer should detect the USB port it's connected to. If your device is not being detected when connected via USB, open the case and check this switch. Follow [Step 1 of the Nabu Casa disassembly guide](https://support.nabucasa.com/hc/en-us/articles/25938306296605-Disassembling-the-enclosure-of-Home-Assistant-Voice-Preview-Edition) to access it – you do not need to go further than Step 1 unless you have a custom case.

Click **Install** in the top right corner of ESPHome Device Builder. This could take 5–10 minutes. ESPHome will tell you if and why it fails.

> **After flashing for the first time**, disconnect your VPE from power completely, wait a few seconds, then reconnect. The device will then reconnect to Home Assistant automatically.

Via CLI (if you have ESPHome installed locally):
```bash
esphome run your-device.yaml
```
Replace `your-device.yaml` with your per-device config file — not `home-assistant-voice.yaml`, which contains unfilled substitutions and will fail to compile directly.

---

## Before you flash – note these down

Before flashing, make a note of these three values somewhere safe. You will need them during setup and potentially again later:

| Value | Where it's used |
| --- | --- |
| `device_suffix` | Required when adding your device in the Pivot integration |
| `api_encryption_key` | Required if you ever need to re-add the device to Home Assistant |
| `ota_password` | Used for wireless updates. Lose it and you can still update using the `api_encryption_key`; lose both and it's a USB reflash |

### `ota_password`

Required, minimum 12 characters. One password can be shared by all your Pivot devices. Make one up yourself, or generate one by running `openssl rand -hex 16` in a terminal.

Once a device is running Pivot firmware, updates can arrive over the air (a USB cable still works too). Without a password, that endpoint accepts *plaintext* uploads from anyone who can reach the device on your network.

It is not the only credential, though. On current ESPHome a device built with an `api_encryption_key` also accepts encrypted OTA uploads authenticated by that key, and those skip the password check – so protect the key as carefully as the password. ESPHome removes the plaintext path in 2027.3.0, and Pivot will move to key-authenticated updates before then, at which point the separate OTA password goes away.

Because a missing OTA password cannot be detected once the device is running, Pivot checks at **build time**. Omitting it, leaving it empty, or using fewer than 12 characters stops the build:

```
error: static assertion failed: ota_password must be at least 12 characters - see SECURITY.md
```

You're free to make up your own password rather than generate one – just keep it at least 12 characters and stick to letters, digits, hyphens and underscores. Only the length is actually checked at build time; the value is embedded in a C++ string literal during the build, so a quote or backslash could break the build or silently produce a different password than you typed, with no warning either way.

**Upgrading a device that has no OTA password yet:** the first upload still works without authentication, because the firmware currently on the device has no password to check against. Enforcement starts from the following update.

**If you lose the password:** you can still update the device wirelessly using its `api_encryption_key`, because encrypted uploads do not check the password. Only losing both means a USB reflash.

**Changing an existing password** is a two-stage process — the configured password both authenticates the upload and sets the new firmware's password, so you cannot simply swap the value. See [`SECURITY.md`](https://github.com/alistairmerritt/pivot-firmware/blob/main/SECURITY.md) in the firmware repository.

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
- Hold+turn in Normal mode retains stock behaviour – changes the LED ring colour (hue cycling)
- Secondary scroll behaviour removed
- Passive banks (scene, script, switch, input_boolean, open/close-only cover) ignore the knob. Scene and script banks turn the LEDs off. Banks with an on/off state show it instead: a full ring in the bank colour when on or open, nothing when off or closed. Bank Indicator still fires normally during bank switching.

All standard VPE functionality (voice assistant, wake word, mute button, LED colour change, etc.) remains intact.

## Upstream Voice Preview Edition changes

Home Assistant’s Voice Preview Edition firmware continues to evolve, and new upstream features or fixes may be added over time. Where relevant, I will aim to review those changes and carry useful updates across into Pivot firmware where possible.

Because Pivot is a custom firmware branch, upstream Voice Preview Edition changes will not always appear in Pivot immediately, and some updates may require additional work or testing before they are adopted.
