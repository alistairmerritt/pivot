---
layout: page
title: Getting Started
permalink: /getting-started/
---

This guide walks you through everything needed to get Pivot running on a Home Assistant Voice Preview Edition device.

## What you need

- A Home Assistant Voice Preview Edition (VPE)
- Home Assistant with the ESPHome integration installed
- The ESPHome application installed in Home Assistant
- HACS installed in Home Assistant
- Your WiFi credentials 


---

## Step 1. Set up your VPE in Home Assistant

Your VPE must already be connected to your WiFi and added to Home Assistant via the ESPHome integration before you can flash Pivot firmware.

If you haven't done this yet, follow [Home Assistant's official VPE setup guide](https://www.home-assistant.io/voice_control/voice_remote_local_assistant/) first, then come back here.

If your VPE is already showing up in Home Assistant under ESPHome, you're ready to continue.

> **Your VPE's voice and listening functionality is not affected by Pivot.** Wake word detection, voice commands, and all standard VPE features continue to work exactly as before. Pivot adds a new layer of control — it does not replace or interfere with anything already set up.


---

## Step 2. Enable Home Assistant actions

While you're in the ESPHome integration, enable this now before flashing the firmware:

1. Go to **Settings → Devices & Services → ESPHome**
2. Find your VPE and click **⚙️ Configure**
3. Enable **Allow the device to perform Home Assistant actions**

This is required for the Pivot firmware to call scripts and send events to Home Assistant.


---

## Step 3. Flash the Pivot firmware

> **Installing custom firmware is safe and reversible.** Pivot firmware is based on the official Home Assistant Voice PE firmware and has been tested extensively, but as with any custom firmware there is a small element of risk. If anything goes wrong, you can always restore the original stock firmware by visiting [esphome.github.io/home-assistant-voice-pe](https://esphome.github.io/home-assistant-voice-pe/) and flashing it from your browser — no tools required.

<ol>
<li><p><strong>Take control of your VPE in ESPHome Device Builder.</strong><br>
If you don't already have ESPHome Device Builder installed, install it via <strong>Settings → Applications → Add Application</strong>.</p>

<p>When you open ESPHome Device Builder, your VPE may appear hidden under Discovered Devices — click <strong>Show</strong> in the top right corner if you don't see it.</p>

<blockquote><strong>Not sure which device is yours?</strong> If you have multiple VPEs, ESPHome names them by the last 6 characters of their MAC address (e.g. <code>Home-Assistant-Voice-052B5D</code>). To find the MAC address of a specific device, go to <strong>Settings → Devices &amp; Services → ESPHome</strong>, select the device, and look in the left column.</blockquote>

<p>Click <strong>Take Control</strong> and give it a name. ESPHome will offer to install its own firmware — you can skip this as it will be replaced by Pivot firmware in the next step.</p>

<blockquote><strong>Save your encryption key.</strong> When you take control, ESPHome shows the device's API encryption key. Copy this and keep it somewhere safe — you'll need it in the next step. Alternatively, generate a fresh one at <a href="https://esphome.io/components/api.html#configuration-variables">esphome.io</a>.</blockquote></li>

<li><p><strong>Create your per-device config.</strong><br>
Copy the template from <a href="https://github.com/alistairmerritt/pivot-firmware/blob/main/devices/example.yaml"><code>devices/example.yaml</code></a> in the firmware repo and paste it into ESPHome, replacing the stock YAML. Fill in your device-specific values:</p>

<pre><code>substitutions:
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

  # API encryption key — use the one from Take Control, or generate one at:
  # https://esphome.io/components/api.html#configuration-variables
  api_encryption_key: "your-key-here"

  # LED orientation:
  #   '6'  flat on a surface, cable facing away (default)
  #   '0'  upright on a stand, cable at the bottom
  led_offset: '6'

  # =======================================================================

packages:
  pivot:
    url: https://github.com/alistairmerritt/pivot-firmware
    ref: main
    file: home-assistant-voice.yaml
    refresh: 1d
</code></pre>

<p>The <code>packages:</code> block at the bottom tells ESPHome to fetch the full firmware from GitHub automatically — do not paste the contents of <code>home-assistant-voice.yaml</code> directly.</p>

<p>The <code>device_suffix</code> must be unique per device with no spaces or dashes. You will enter this exact value when setting up the Pivot integration.</p>

<blockquote><strong>WiFi credentials:</strong> The config uses <code>!secret</code> to keep credentials out of the YAML. Add <code>wifi_ssid</code> and <code>wifi_password</code> to your ESPHome <strong>Secrets</strong> file (the key icon in ESPHome Device Builder). If you prefer, you can paste the values directly as plain text instead.</blockquote>

<blockquote><strong>Tip:</strong> Note down your <code>device_suffix</code> and <code>api_encryption_key</code> somewhere safe. You will need both — the suffix when adding the Pivot integration, and the key if Home Assistant ever asks for it.</blockquote></li>

<li><p><strong>Connect your VPE via USB</strong> for the initial flash — use a good quality cable. OTA (wireless) is used for all future updates once the device is running Pivot firmware.</p>

<blockquote><strong>Tip:</strong> There is a small switch inside the VPE case labelled <strong>USB SELECT</strong> with two positions: <strong>ESP32</strong> and <strong>XU316</strong>. It should be in the <strong>ESP32</strong> position by default. If your device is not detected when connected via USB, open the case and check this switch. Follow <a href="https://support.nabucasa.com/hc/en-us/articles/25938306296605-Disassembling-the-enclosure-of-Home-Assistant-Voice-Preview-Edition">Step 1 of the Nabu Casa disassembly guide</a> to access it — you do not need to go further than Step 1.</blockquote></li>

<li>Click <strong>Install → Plug into this computer</strong>. ESPHome will fetch the firmware from GitHub, compile it, and flash it. This may take 5–10 minutes the first time.</li>

<li>Once flashed, <strong>fully power cycle your VPE</strong> — disconnect from power, wait a few seconds, then reconnect. The device will reconnect to Home Assistant automatically.</li>
</ol>

> **Future updates** — once your device is running Pivot firmware and is online, all future updates are wireless. When a new firmware version is released, just open the device in ESPHome Device Builder and click **Install → Wirelessly**. No USB required.


---

## Step 4. Install the Pivot integration

1. In HACS, go to **Integrations → ⋮ → Custom repositories**
2. Add `https://github.com/alistairmerritt/pivot-integration`, category: **Integration**
3. Search for **Pivot** and install
4. Restart Home Assistant


---

## Step 5. Add your device in Pivot

1. Go to **Settings → Devices & Services → Add Integration** and search for **Pivot**
2. Select your VPE from the dropdown
3. Confirm the firmware and enter the `device_suffix` you used in the firmware YAML
4. Choose a **setup mode**:
   - **Automatic** — Pivot creates and manages the required scripts and automations for you. Recommended for most users.
   - **Blueprints** — Pivot installs blueprint files. You create the automations yourself from the HA UI.
   - **Manual** — Pivot does not create any files. Use the fired events to build your own automations.
5. Optionally configure announcements — select a text-to-speech service (TTS) and speaker to have Pivot speak the active bank name when you switch


> **Do not rename Pivot entity IDs.** The firmware and integration use your `device_suffix` to build entity IDs (e.g. `number.{suffix}_bank_0_value`). Renaming these entities in Home Assistant will break the connection between the firmware and the integration. If you need to label entities more clearly, change the entity's **Name** — not its **Entity ID**.

---

## Step 6. Assign entities to banks

1. Go to **Settings → Devices & Services → Pivot → your device → Configure**
2. Assign a Home Assistant entity to each bank
3. Save

You’re done. Turn the knob to control the active bank’s assigned entity, press the button to toggle or activate it, and hold the knob while turning to switch banks.

> **Bank entity chooser not appearing?** Occasionally the entity assignment screen is skipped during initial setup. If this happens, you can assign entities at any time by going to **Settings → Devices & Services → Pivot**, clicking the **⚙️ Configure** icon on your device, and stepping through the setup screens. The bank entity chooser appears at the end — just click through the earlier screens (setup mode, announcements) to reach it. Alternatively, you can assign entities directly by editing the `text.{suffix}_bank_N_entity` text entities from **Developer Tools → States**.


---

## Next steps

- [Firmware reference](/pivot/firmware/) — full substitutions reference and multi-device setup
- [Integration reference](/pivot/integration/) — full entity reference, setup modes, and custom automations
- [Troubleshooting](/pivot/troubleshooting/) — common issues and fixes
