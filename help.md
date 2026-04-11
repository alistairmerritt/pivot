---
layout: page
title: Help
permalink: /help/
---

## On this page

**[FAQ](#faq)**
— [General](#general) · [Setup & Installation](#setup--installation) · [Behaviour & Controls](#behaviour--controls) · [Firmware & Updates](#firmware--updates) · [Advanced](#advanced) · [Timer](#timer)

**[Troubleshooting](#troubleshooting)**
— [Device & flashing](#device--flashing) · [Connection](#connection) · [Integration & entities](#integration--entities) · [Controls](#controls) · [Announcements & timer](#announcements--timer) · [File & config issues](#file--config-issues)

---

## FAQ

### General

**What is Pivot?**
Pivot is a firmware and Home Assistant integration that turns the Home Assistant Voice Preview Edition into a four-bank physical control dial.

Each bank represents a different control, allowing you to adjust or activate an assigned entity using a simple turn and press interaction.

---

**What is a "bank"?**
A bank is one of four control slots on the dial.

Each bank is assigned to a single entity, script, or scene, and you switch between them using press + turn.

---

**What can I control with Pivot?**
Pivot is designed to work with controllable Home Assistant entities.

Common examples include:
- Lights
- Media players
- Fans
- Covers
- Climate entities
- Input numbers
- Scripts
- Scenes

It can also be extended further through automations.

---

**Does Pivot replace voice control?**
No. Pivot preserves the core voice functionality of the Voice Preview Edition, including wake word activation.

You can continue using voice as normal — Pivot simply adds a physical control layer alongside it.

---

**Does Pivot change how voice is triggered?**
Yes. In the stock VPE firmware, a single press starts the voice input. In Pivot, that press is used for physical control instead.

Voice functionality is still preserved, including wake word activation, but the stock press-to-talk shortcut is no longer the default behaviour while Pivot's control mode is enabled.

If you would prefer a manual trigger as well, a good option is to use long press to run a Home Assistant automation that starts a conversation on the device. This is not built in by default, but can be configured manually.

---

**Can I switch between Pivot behaviour and the stock Voice Preview Edition behaviour?**
Yes. Double press toggles Pivot control mode on or off.

When control mode is enabled, the dial controls your assigned entities using Pivot's bank system. When control mode is disabled, the device behaves like a standard Voice Preview Edition again, including the default button actions (with the exception of double press).

This makes it easy to switch between Pivot and the stock experience at any time.

---

### Setup & Installation

**Do I need to flash custom firmware?**
Yes. Pivot requires flashing custom firmware to the Voice Preview Edition using ESPHome.

After the initial flash, updates can be installed wirelessly.

---

**Can I go back to the original firmware?**
Yes. You can re-flash the device with the stock Home Assistant firmware at any time.

---

**Do I need experience with ESPHome?**
No. The Getting Started guide walks through the process step-by-step.

---

---

### Behaviour & Controls

**How do I use the dial?**
- Turn → adjust the active bank
- Press → activate or toggle
- Press + turn → switch banks

---

**How do I know which bank I'm on?**
The LED ring shows the active bank using its configured colour, and banks can optionally announce the assigned entity when you switch to them.

You can also triple press the button to have Pivot announce the current bank and its assigned entity.

---

**How do the LEDs work?**
Pivot uses the LED ring to communicate both identity and value:

- **Bank colour** → shows which bank is active
- **Value display** → shows the current value of adjustable entities
- **RGB lights** → reflect the light's colour
- **Non-RGB entities** → use the bank colour as the value indicator
- **Passive entities** → briefly show the bank colour, then turn off

Each bank's colour can be customised from within Home Assistant, so you can choose colours that make sense for your setup.

---

**Why do the LEDs turn off on some banks?**
If a bank is assigned to a passive entity (such as a switch, scene, or script), there is no adjustable value to display. In this case, the LED ring will briefly show the bank colour when selected, then turn off.

---

**What does "mirror light colour" mean?**
When enabled, the LED ring will match the colour of the assigned light.

When disabled, the LEDs use the bank's configured colour instead.

---

### Firmware & Updates

**How do I update the firmware?**
In Home Assistant:
- Go to **ESPHome Device Builder**
- Select your device
- Click **Install → Wirelessly**

---

**What's the difference between firmware and the integration?**
- **Firmware** controls the physical behaviour (dial input, LEDs, bank switching)
- **Integration** controls how Pivot interacts with Home Assistant (entities, automations, timers)

---

### Advanced

**Can I use multiple Pivot VPE devices?**
Yes. Each device uses a unique identifier, allowing multiple VPE devices to run independently.

---

**Does Pivot work without internet?**
Yes. Pivot runs entirely locally within Home Assistant and ESPHome.

---

**What happens if Home Assistant restarts?**
Pivot restores its state and continues working once Home Assistant is available again.

---

### Timer

**Can I use Pivot as a timer?**
Yes. A bank can be assigned to a timer, allowing you to:
- Set duration with the dial
- Start/pause with a press
- Receive alerts when the timer finishes

See the [Timer](/pivot/timer/) page for setup instructions.

---

**Is the timer built-in?**
The timer feature is optional and uses additional entities and automations provided by Pivot. It requires a small amount of setup — see the [Timer](/pivot/timer/) page.

---

## Troubleshooting

### Device & flashing

### The device won't appear when connected via USB for flashing

First make sure you are using a good quality USB cable — some cables are charge-only and do not support data transfer. Try a different cable if you have one.

> **Tip:** There is a small switch inside the VPE case labelled **USB SELECT** with two positions: **ESP32** and **XU316**. It should be in the **ESP32** position by default. If your device is still not being detected, open the case and check this switch. Follow [Step 1 of the Nabu Casa disassembly guide](https://support.nabucasa.com/hc/en-us/articles/25938306296605-Disassembling-the-enclosure-of-Home-Assistant-Voice-Preview-Edition) to access it — you do not need to go further than Step 1 unless you have a custom case.

---

### Something went wrong with the firmware — how do I recover?

You can always restore the original stock Home Assistant Voice PE firmware by visiting [esphome.github.io/home-assistant-voice-pe](https://esphome.github.io/home-assistant-voice-pe/) in your browser and following the on-screen instructions. No tools or ESPHome installation required.

---

### ESPHome build fails with "'.' is an invalid character for names"

This happens when the `device_name` substitution in your device YAML includes the `.yaml` file extension. ESPHome device names cannot contain dots.

Open your device YAML in ESPHome and find the `substitutions:` block. The `device_name` value should be the name only — no extension, lowercase, using hyphens or underscores:

```yaml
substitutions:
  device_name: ha-voice-kitchen        # ✓ correct
  device_name: ha-voice-kitchen.yaml   # ✗ causes this error
```

Remove the `.yaml` suffix, save, and retry the build.

---

### Connection

### My VPE just has revolving blue lights after installing the firmware

The revolving blue light pattern means the device is trying to connect but hasn't succeeded yet. There are two likely causes:

**Encryption key mismatch** — check **Settings → Notifications** in Home Assistant. If there is an alert asking you to reconfigure a device, open it and enter the `api_encryption_key` from your firmware YAML's substitutions block.

**WiFi credentials issue** — if there is no notification in Home Assistant, the device may not be reaching your network at all. Open your firmware YAML in ESPHome and double-check that your `wifi_ssid` and `wifi_password` are correct, then reflash.

---

### The device won't connect to Home Assistant after flashing

After flashing the Pivot firmware for the first time, **fully power cycle your VPE** — disconnect it from power completely, wait a few seconds, then reconnect. A simple restart is not always enough. The device should then reconnect and appear in Home Assistant automatically.

---

### My VPE doesn't appear in the ESPHome application

If your device isn't showing up in ESPHome, you may need to take control of it first. Follow the [ESPHome getting started guide](https://esphome.io/guides/getting_started_hassio/) which walks through adding a device to the ESPHome application.

---

### Home Assistant asks for an encryption key

When Home Assistant asks for an encryption key, open your firmware YAML file and copy the `api_encryption_key` value from the substitutions block — paste that directly into the box Home Assistant is showing you.

---

### The device suffix mismatch — entities have wrong IDs

The `device_suffix` in your firmware YAML must match exactly what you entered in the integration setup. If they don't match, the firmware and integration will use mismatched entity IDs and won't communicate.

To check: go to **Settings → Devices & Services → Pivot → your device** and look at the entity IDs. They should all start with your `device_suffix`. If they don't match what the firmware expects, the easiest fix is to remove and re-add the Pivot integration using the correct suffix.

---

### Integration & entities

### The integration says "Invalid handler specified" when I try to add it

This usually means HA has loaded stale cached files. Fix it from the UI:

1. Go to **Settings → System → Restart** and do a full restart (not just a quick reload)
2. Try adding the integration again

If it still fails after a full restart, the cache may need to be cleared manually via SSH:
```bash
rm -rf /config/custom_components/pivot/__pycache__
```
Then restart Home Assistant again.

---

### Nothing happens after installing everything

Start here before anything else:

1. Go to **Settings → Devices & Services → ESPHome → your VPE → ⚙️ Configure** and confirm **Allow the device to perform Home Assistant actions** is ticked. Without this, the firmware cannot call scripts or send events to Home Assistant.
2. Go to **Settings → Devices & Services → Pivot → your device → Configure** and confirm at least one bank has an entity assigned.
3. Make sure you are in **Control Mode** — double press the button to toggle it on. The LED ring should change to show the active bank colour.

---

### Controls

### The knob turns but nothing happens

Work through these in order:

1. **Check bank assignment** — go to **Settings → Devices & Services → Pivot → your device → Configure** and confirm the active bank has an entity assigned.
2. **Check the entity domain** — the entity must be a supported type: light, fan, media player, climate, or cover. Scenes and scripts are passive (knob does nothing, button only).
3. **Check Control Mode is on** — go to **Settings → Devices & Services → Pivot → your device** and check that the **Control Mode** switch is on. You can also toggle it with a double press on the button.

---

### The button press does nothing

1. **Check bank assignment** — go to **Settings → Devices & Services → Pivot → your device → Configure** and confirm the active bank has an entity assigned.
2. **Check Control Mode is on** — the button only toggles entities in Control Mode. Double press to toggle it on.
3. **Check the integration is up to date** — button toggle is handled natively by the integration (v0.0.59+). If you are on an older version, update via HACS and restart Home Assistant.
4. **Check firmware is up to date** — the integration handles the toggle from the `button_press_event` entity. If your firmware predates v0.0.15, reflash via ESPHome → Install → Wirelessly.

---

### Announcements & timer

### Announcements are not working

1. Go to **Settings → Devices & Services → Pivot → your device** and check that the **Announcements** switch is on.
2. Go to **Configure** on the integration and confirm a text-to-speech service and speaker are selected.
3. Test your TTS service independently — go to **Developer Tools → Actions**, find `tts.speak`, select your TTS entity and media player, and send a test message. If this doesn't work, the issue is with your TTS setup rather than Pivot.
4. Check the announce automation exists — go to **Settings → Automations** and search for your device suffix. If it's missing, create it from the **Pivot — Announce** blueprint — the only input required is your **Device Suffix**. See the [getting started guide](/pivot/getting-started/#announce-automation--optional) for instructions.

---

### The timer blueprint triggers when I turn the knob on a bank with a real entity

This happens when the timer blueprint is set up on a bank that also has a real entity assigned. The blueprint requires the bank to be set as a timer bank before it will respond — this prevents it from interfering with normally-assigned banks.

To fix: go to **Settings → Devices & Services → Pivot → your device → Configure**, step through to the **Bank Entity Assignment** screen, select the correct bank under **Timer bank (optional)**, and save. This clears the entity assignment for that bank automatically.

---

### The timer gauge (LED ring) does not update while the timer is running

1. Make sure all three timer entities are enabled: `number.{device_suffix}_timer_duration`, `select.{device_suffix}_timer_state`, and `text.{device_suffix}_timer_end`. All three must be enabled for the blueprint to work.
2. Check the bank entity for the timer bank is set to `timer` (not left blank or set to a real entity).
3. Confirm the timer automation is enabled — go to **Settings → Automations** and check it is toggled on.
4. The gauge updates every 30 seconds, not continuously. A brief delay before the first update is normal.

---

### File & config issues
