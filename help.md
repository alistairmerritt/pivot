---
layout: page
title: Help
permalink: /help/
---

## On this page

**[FAQ](#faq)**
 –  [General](#general) · [Setup & Installation](#setup--installation) · [Behaviour & Controls](#behaviour--controls) · [Firmware & Updates](#firmware--updates) · [Advanced](#advanced) · [Timer](#timer)

**[Troubleshooting](#troubleshooting)**
 –  [Device & flashing](#device--flashing) · [Connection](#connection) · [Integration & entities](#integration--entities) · [Controls](#controls) · [Announcements & timer](#announcements--timer) · [File & config issues](#file--config-issues)

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

You can continue using voice as normal – Pivot simply adds a physical control layer alongside it.

---

**Does Pivot change how voice is triggered?**
Only while Control Mode is enabled. When Control Mode is on, single press toggles or activates the assigned entity instead of starting the voice assistant. When Control Mode is off (double press to toggle it), single press returns to starting the voice assistant exactly as it does in the stock firmware.

Wake word activation is always available regardless of Control Mode. If you want a manual voice trigger while staying in Control Mode, long press is left open for this purpose – see [Can I use long press to start a voice conversation?](#can-i-use-long-press-to-start-a-voice-conversation) in the Advanced section.

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
- **Switches and garage doors** → a full ring when on or open, off when off or closed
- **Scenes and scripts** → briefly show the bank colour, then turn off

Each bank's colour can be customised from within Home Assistant, so you can choose colours that make sense for your setup.

---

**Why do the LEDs turn off on some banks?**
If a bank is assigned to a scene or script, there is no value or state to display, so the LED ring briefly shows the bank colour when selected, then turns off.

Switches, input_booleans and open/close-only covers (such as most garage doors) are different: the ring shows their state – a full circle in the bank colour when on or open, and off when off or closed. So on those banks, a dark ring means the entity is off or closed. This needs integration v0.0.90 and firmware v0.0.27 – with older firmware, these banks also turn the LEDs off.

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

**How can I trigger voice from a button press in Control Mode?**
Long press is intentionally left open – it is the one press type Pivot does not consume natively, so you can wire it up however you like.

A common use is starting a voice conversation on the device without speaking a wake word. To do this, create a Home Assistant automation using the example below, substituting your own device entity IDs.

> **Note:** There is a short pause between the long press and the microphone opening. This is because `assist_satellite.start_conversation` requires a `start_message` – the device speaks it before it starts listening. Even a very short message adds a small TTS round-trip delay. If you leave `start_message` blank, HA will reject the service call. The delay is typically less than a second but is noticeable.

```yaml
alias: Pivot – Long Press Start Conversation
description: Long press on any Pivot device starts a voice conversation on that device.
mode: parallel
max: 10

trigger:
  - platform: state
    entity_id:
      - event.your_device_button_press
      # add one line per device

condition:
  - condition: template
    value_template: "{{ trigger.to_state.attributes.event_type == 'long_press' }}"

variables:
  satellite_map:
    event.your_device_button_press: assist_satellite.your_device_assist_satellite
    # keep this in sync with the trigger list above
  satellite: "{{ satellite_map[trigger.entity_id] }}"

action:
  - service: assist_satellite.start_conversation
    target:
      entity_id: "{{ satellite }}"
    data:
      start_message: " "
```

The trigger uses a plain state change (not filtered by `attribute`/`to`) because ESPHome event entities update their timestamp on every press – if you filter by `to: long_press`, the trigger only fires the first time and ignores all subsequent long presses.

> **Tip:** If you use the Timer feature, long press already cancels a running or paused timer via the Timer blueprint. Both automations will fire simultaneously on a long press while a timer is active. To avoid this, add a condition to the conversation automation checking that the timer is not running or paused.

---

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
The timer feature is optional and uses additional entities and automations provided by Pivot. It requires a small amount of setup – see the [Timer](/pivot/timer/) page.

---

## Troubleshooting

### Device & flashing

### The device won't appear when connected via USB for flashing

First make sure you are using a good quality USB cable – some cables are charge-only and do not support data transfer. Try a different cable if you have one.

> **Tip:** There is a small switch inside the VPE case labelled **USB SELECT** with two positions: **ESP32** and **XU316**. It should be in the **ESP32** position by default. If your device is still not being detected, open the case and check this switch. Follow [Step 1 of the Nabu Casa disassembly guide](https://support.nabucasa.com/hc/en-us/articles/25938306296605-Disassembling-the-enclosure-of-Home-Assistant-Voice-Preview-Edition) to access it – you do not need to go further than Step 1 unless you have a custom case.

---

### Something went wrong with the firmware – how do I recover?

You can always restore the original stock Home Assistant Voice PE firmware by visiting [esphome.github.io/home-assistant-voice-pe](https://esphome.github.io/home-assistant-voice-pe/) in your browser and following the on-screen instructions. No tools or ESPHome installation required.

---

### ESPHome build fails with "'.' is an invalid character for names"

This happens when the `device_name` substitution in your device YAML includes the `.yaml` file extension. ESPHome device names cannot contain dots.

Open your device YAML in ESPHome and find the `substitutions:` block. The `device_name` value should be the name only – no extension, lowercase, using hyphens or underscores:

```yaml
substitutions:
  device_name: ha-voice-kitchen        # ✓ correct
  device_name: ha-voice-kitchen.yaml   # ✗ causes this error
```

Remove the `.yaml` suffix, save, and retry the build.

---

### ESPHome build fails with "ota_password must be at least 12 characters"

Or `Set a unique ota_password in your device YAML`. Both mean the same thing: your device YAML has no usable `ota_password`.

This is intentional, not a bug. Once a device runs Pivot firmware, updates can arrive wirelessly, and an OTA endpoint without a password accepts firmware from anyone who can reach the device on your network. A missing password cannot be detected after the device is running, so Pivot checks while building instead.

Run this in a terminal to generate one, or make up your own following the same length and character rules below:

```bash
openssl rand -hex 16
```

```yaml
substitutions:
  ota_password: !secret pivot_ota_lounge   # ✓ from your ESPHome secrets file
  ota_password: "a1b2c3d4e5f6a7b8c9d0e1f2" # ✓ or inline
  ota_password: ""                         # ✗ empty is rejected
  ota_password: "hunter2"                  # ✗ under 12 characters
```

If using `!secret`, add the matching entry to your ESPHome **Secrets** file (the key icon in ESPHome Device Builder), using a different name for each device.

Only the length is checked (12+ characters) — the build does not inspect content, so a value with quotes or a backslash will not necessarily be caught here. It is embedded in a C++ string literal during the build though, so such a character could break the build or silently produce a different password than you typed. Stick to the hexadecimal output above, or letters/digits/hyphens/underscores, to avoid that risk.

**Updating a device that never had one:** this first upload still works without authentication, because the firmware currently on the device has no password to check. Enforcement begins with the next update. Save the password somewhere safe — if you lose it, the only way back in is a USB reflash.

---

### Connection

### My VPE just has revolving blue lights after installing the firmware

The revolving blue light pattern means the device is trying to connect but hasn't succeeded yet. There are two likely causes:

**Encryption key mismatch** – check **Settings → Notifications** in Home Assistant. If there is an alert asking you to reconfigure a device, open it and enter the `api_encryption_key` from your firmware YAML's substitutions block.

**WiFi credentials issue** – if there is no notification in Home Assistant, the device may not be reaching your network at all. Open your firmware YAML in ESPHome and double-check that your `wifi_ssid` and `wifi_password` are correct, then reflash.

---

### The device won't connect to Home Assistant after flashing

After flashing the Pivot firmware for the first time, **fully power cycle your VPE** – disconnect it from power completely, wait a few seconds, then reconnect. A simple restart is not always enough. The device should then reconnect and appear in Home Assistant automatically.

---

### My VPE doesn't appear in the ESPHome application

If your device isn't showing up in ESPHome, you may need to take control of it first. Follow the [ESPHome getting started guide](https://esphome.io/guides/getting_started_hassio/) which walks through adding a device to the ESPHome application.

---

### Home Assistant asks for an encryption key

When Home Assistant asks for an encryption key, open your firmware YAML file and copy the `api_encryption_key` value from the substitutions block – paste that directly into the box Home Assistant is showing you.

---

### The device suffix mismatch – entities have wrong IDs

The `device_suffix` in your firmware YAML must match exactly what you entered in the integration setup. If they don't match, the firmware and integration will use mismatched entity IDs and won't communicate.

**How it shows up:** the button still works – a press toggles the assigned entity – but turning the knob does nothing, and settings you change on the Pivot device page don't reach the device. The knob and settings find Pivot's entities by suffix, while button presses are matched to the device itself, which is why one works without the other. (The opposite – knob works, button doesn't – has a different cause: see [The button press does nothing](#the-button-press-does-nothing).)

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
3. Make sure you are in **Control Mode** – double press the button to toggle it on. The LED ring should change to show the active bank colour.

---

### Controls

### The knob turns but nothing happens

Work through these in order:

1. **Check bank assignment** – go to **Settings → Devices & Services → Pivot → your device → Configure** and confirm the active bank has an entity assigned.
2. **Check the entity domain** – the entity must be a supported type: light, fan, media player, climate, number, input_number, or a cover that accepts a position. Scenes, scripts, switches and open/close-only covers such as most garage doors are passive (knob does nothing, button only).
3. **Check Control Mode is on** – go to **Settings → Devices & Services → Pivot → your device** and check that the **Control Mode** switch is on. You can also toggle it with a double press on the button.
4. **If the button works but the knob doesn't** – the `device_suffix` in your firmware YAML probably doesn't match the integration. See [The device suffix mismatch](#the-device-suffix-mismatch--entities-have-wrong-ids).

---

### The button press does nothing

1. **Check bank assignment** – go to **Settings → Devices & Services → Pivot → your device → Configure** and confirm the active bank has an entity assigned.
2. **Check Control Mode is on** – the button only toggles entities in Control Mode. Double press to toggle it on.
3. **If the knob works but the button doesn't** – the Pivot entry is probably linked to an old copy of your VPE. This happens when the VPE is added to Home Assistant again: re-adopting it in ESPHome, re-adding it after a reset, or first adding it by IP address and later by name. The knob keeps working because it finds Pivot's entities by suffix, but button presses are matched to the specific device the Pivot entry was set up with – and that device no longer exists. Triple-press announcements go quiet for the same reason. Nothing warns you.

   **To check:** go to **Settings → Devices & Services → ESPHome**. If your VPE appears twice, the copy whose entities are all unavailable is the old one. You can also open **Developer Tools → States**, find your VPE's `event.…_button_press` entity and press the button: if its time updates but nothing toggles, this is the cause.

   **To fix:** make a note of your bank assignments, then delete your device's entry from the Pivot integration and add it again, choosing the copy of the VPE whose entities are available. At the suffix step, enter the same `device_suffix` as your firmware YAML – the field is pre-filled from the ESPHome name, which may be different. If you're sure the unavailable copy is old, delete it from ESPHome first so it can't be picked by mistake.
4. **Check the integration is up to date** – button toggle is handled natively by the integration. Update via HACS and restart Home Assistant if you are not on the latest version.
5. **Check firmware is up to date** – open your device in ESPHome Device Builder and click **Install → Wirelessly** to get the latest firmware.

---

### Announcements & timer

### Announcements are not working

1. Go to **Settings → Devices & Services → Pivot → your device** and check that the **Announcements** switch is on.
2. Go to **Configure** on the integration and confirm a text-to-speech service and speaker are selected, and that **Enable spoken announcements** is ticked.
3. Test your TTS service independently – go to **Developer Tools → Actions**, find `tts.speak`, select your TTS entity and media player, and send a test message. If this doesn't work, the issue is with your TTS setup rather than Pivot.

---

### The timer blueprint triggers when I turn the knob on a bank with a real entity

This happens when the timer blueprint is set up on a bank that also has a real entity assigned. The blueprint requires the bank to be set as a timer bank before it will respond – this prevents it from interfering with normally-assigned banks.

To fix: go to **Settings → Devices & Services → Pivot → your device → Configure**, step through to the **Bank Entity Assignment** screen, select the correct bank under **Timer bank (optional)**, and save. This clears the entity assignment for that bank automatically.

---

### The timer gauge (LED ring) does not update while the timer is running

1. Make sure all three timer entities are enabled: `number.{device_suffix}_timer_duration`, `select.{device_suffix}_timer_state`, and `text.{device_suffix}_timer_end`. All three must be enabled for the blueprint to work.
2. Check the bank entity for the timer bank is set to `timer` (not left blank or set to a real entity).
3. Confirm the timer automation is enabled – go to **Settings → Automations** and check it is toggled on.
4. The gauge updates every 30 seconds, not continuously. A brief delay before the first update is normal.

---

### File & config issues

### I didn't receive the blueprint import notification

When you first add a Pivot device, Pivot sends a one-time Home Assistant notification with links to import the optional timer blueprints from GitHub. If you missed it:

1. Go to **Settings → Devices & Services → Pivot → your device** and click the ⋮ menu → **Reload**. This re-sends the notification if it hasn't been dismissed.
2. If the notification still doesn't appear, you can import the blueprints manually from the [Timer](/pivot/timer/) page – the import links are listed there.

---

### The `device_suffix` field is greyed out or pre-filled with the wrong value

The config flow pre-fills `device_suffix` based on the ESPHome device name. If the pre-filled value does not match your firmware's actual suffix, clear the field and type the suffix manually – it must match the `device_suffix` substitution in your ESPHome YAML exactly.

---

## Diagnostic flowchart

Use this to quickly narrow down where a problem is.

```
Is the VPE showing a revolving blue LED pattern?
├── Yes → Connection problem
│   ├── Check HA for a notification asking for an encryption key
│   │   └── Yes → Enter the api_encryption_key from your firmware YAML
│   └── No notification → WiFi credentials issue
│       └── Check wifi_ssid / wifi_password in your ESPHome YAML and reflash
│
└── No (solid or pulsing colour, or off)
    │
    Is the VPE showing the active bank colour on the LED ring?
    ├── No → Control Mode is off
    │   └── Double press the button to enable Control Mode
    │
    └── Yes → Control Mode is on
        │
        Does turning the knob change the entity value?
        ├── No
        │   ├── Check bank has an entity assigned (Settings → Pivot → Configure)
        │   ├── Check the entity type is supported (light, fan, media player, climate, positionable cover, number, input_number)
        │   ├── Check "Allow device to perform HA actions" is enabled in ESPHome integration
        │   └── Button still works? → device_suffix mismatch (see Connection)
        │
        └── Yes – knob works
            │
            Does pressing the button toggle the entity?
            ├── No
            │   ├── Pivot entry linked to an old copy of the VPE (see "The button press does nothing", step 3)
            │   ├── Update integration via HACS
            │   └── Update firmware via ESPHome Device Builder
            │
            └── Yes – everything is working
```
