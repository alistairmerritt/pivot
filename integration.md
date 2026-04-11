---
layout: page
title: Integration
permalink: /integration/
---

The Pivot HA integration provisions all required entities for a Pivot device, handles button toggle natively, and in Blueprint mode installs blueprint files for optional announcement and timer automations.

Install via HACS from [alistairmerritt/pivot-integration](https://github.com/alistairmerritt/pivot-integration).

---

Pivot installs blueprint files into `/config/blueprints/` automatically. Button toggle, knob control, and entity sync all work out of the box — blueprints are optional extras for announcements and timers.

> **Advanced:** You can ignore blueprints entirely and build your own automations using Pivot's events. See [Custom Automations](#custom-automations) below.

---

## Entities

### Number entities

| Entity | Purpose |
| --- | --- |
| `number.{device_suffix}_bank_0_value` | Bank 1 value (0–100%) |
| `number.{device_suffix}_bank_1_value` | Bank 2 value (0–100%) |
| `number.{device_suffix}_bank_2_value` | Bank 3 value (0–100%) |
| `number.{device_suffix}_bank_3_value` | Bank 4 value (0–100%) |
| `number.{device_suffix}_active_bank` | Active bank (1–4) |

### Switch entities

| Entity | Purpose |
| --- | --- |
| `switch.{device_suffix}_control_mode` | Control Mode vs Normal (voice) Mode |
| `switch.{device_suffix}_show_control_value` | Keep gauge LEDs permanently visible in control mode |
| `switch.{device_suffix}_dim_when_idle` | Dim gauge LEDs to 50% after 2 s of inactivity (requires Show Control Value) |
| `switch.{device_suffix}_announcements` | System Announcements — enable/disable bank-change and triple-press TTS announcements |
| `switch.{device_suffix}_mute_announcements` | Temporarily mute all spoken announcements (bank-change, value, and timer) without changing other settings |
| `switch.{device_suffix}_bank_0_mirror_light` | Bank 1 — mirror assigned RGB light colour |
| `switch.{device_suffix}_bank_1_mirror_light` | Bank 2 — mirror assigned RGB light colour |
| `switch.{device_suffix}_bank_2_mirror_light` | Bank 3 — mirror assigned RGB light colour |
| `switch.{device_suffix}_bank_3_mirror_light` | Bank 4 — mirror assigned RGB light colour |
| `switch.{device_suffix}_bank_0_announce_value` | Bank 1 — announce the entity value via TTS after the knob settles |
| `switch.{device_suffix}_bank_1_announce_value` | Bank 2 — announce the entity value via TTS after the knob settles |
| `switch.{device_suffix}_bank_2_announce_value` | Bank 3 — announce the entity value via TTS after the knob settles |
| `switch.{device_suffix}_bank_3_announce_value` | Bank 4 — announce the entity value via TTS after the knob settles |

### Text entities

| Entity | Purpose |
| --- | --- |
| `text.{device_suffix}_bank_0_entity` | Entity assigned to Bank 1 |
| `text.{device_suffix}_bank_1_entity` | Entity assigned to Bank 2 |
| `text.{device_suffix}_bank_2_entity` | Entity assigned to Bank 3 |
| `text.{device_suffix}_bank_3_entity` | Entity assigned to Bank 4 |
| `text.{device_suffix}_tts_entity` | TTS service used by Announce and Timer blueprints (diagnostic, written from integration settings) |
| `text.{device_suffix}_media_player_entity` | Speaker used by Announce and Timer blueprints (diagnostic, written from integration settings) |

> **Reserved value:** Setting a bank entity to `timer` (lowercase) marks that bank as a timer bank for use with the [Pivot Timer blueprint](/pivot/timer/). The knob has no effect on a timer bank — the LED gauge is managed entirely by the blueprint. Any other value is treated as a Home Assistant entity ID.

> **TTS and media player entities** are written automatically from the values you configure in the integration settings (**Settings → Devices & Services → Pivot → your device → Configure**). The Announce and Timer blueprints read from these entities at runtime — you only need to set TTS and speaker once, in the configure dialog, and all blueprints pick them up automatically.

### Binary sensor entities

| Entity | Purpose |
| --- | --- |
| `binary_sensor.{device_suffix}_bank_0_passive` | On when Bank 1 entity is a scene, script, switch, or input_boolean (knob disabled) |
| `binary_sensor.{device_suffix}_bank_1_passive` | On when Bank 2 entity is a scene, script, switch, or input_boolean (knob disabled) |
| `binary_sensor.{device_suffix}_bank_2_passive` | On when Bank 3 entity is a scene, script, switch, or input_boolean (knob disabled) |
| `binary_sensor.{device_suffix}_bank_3_passive` | On when Bank 4 entity is a scene, script, switch, or input_boolean (knob disabled) |

### Timer entities

These entities are provisioned per device but **disabled by default**. Enable them individually in the HA entity registry if you want to use the [Pivot Timer](/pivot/timer/) feature.

| Entity | Purpose |
| --- | --- |
| `number.{device_suffix}_timer_duration` | Timer duration in minutes (1–60, default 25) |
| `select.{device_suffix}_timer_state` | Timer state — idle, running, or paused |
| `text.{device_suffix}_timer_end` | Internal — stores the countdown end time while the timer is running |

> **Do not rename Pivot entity IDs.** The firmware and integration use your `device_suffix` to build entity IDs at runtime. Renaming any of these entities in Home Assistant will break the connection between the firmware and the integration. If you need to label entities more clearly, change the entity's **Name** — not its **Entity ID**.

---

## Mirror light colour

Each bank has an optional **Mirror light colour** switch (`switch.{device_suffix}_bank_N_mirror_light`). When enabled for a bank that is assigned to an RGB light, the bank's LED ring colour will match the current colour of that light instead of the fixed default bank colour.

- **Mirror on, knob idle** — LED ring uses the light's current RGB colour, updating whenever the light changes.
- **Mirror on, knob turning (value change)** — the gauge arc shows the light's current colour as it updates the brightness or volume, just as at idle.
- **Mirror on, bank switching (press + turn)** — the Bank Indicator quadrant shows the bank's configured colour (Blue/Orange/Green/Purple or your custom colour from the picker), not the light colour, so you can tell which bank you are on even when all your lights are the same colour (e.g. white).
- **Mirror on, light off** — LED ring stays at the last mirrored colour until the light comes back on.
- **Mirror off** — LED ring shows the colour set in the bank's colour picker. If you haven't changed it, this will be the default bank colour (Blue, Orange, Green, or Purple).

This is a persistent per-bank setting, not a temporary effect. It only applies to RGB lights — if the assigned entity is not a compatible RGB light, the bank falls back to its default colour automatically.


---

## Dim LEDs when idle

When **Dim LEDs When Idle** is enabled, the control mode gauge dims to 50% brightness after 2 seconds of no interaction and returns to full brightness the moment you touch the device again.

- **Idle** means no knob turns, button presses, or bank switches for 2 seconds.
- **Waking** is instant — LEDs snap back to full brightness on the first interaction.
- **Fading** to dim is smooth — a 1.5 second transition so there is no sudden flicker.
- This is a **pure brightness modifier** — it does not change colours, values, or any other behaviour.

> **Requires Show Control Value to be on.** Dim when idle has no effect if the gauge is not permanently visible. Enable **Show Control Value** first, then enable **Dim LEDs When Idle**.

---

## Announcements

Pivot can speak the name of the active bank and the current value of a bank's entity via TTS. Both are handled by a single **Pivot — Announce** blueprint, installed automatically in Blueprint mode.

### Pivot — Announce blueprint

The blueprint handles three announcement types in one automation:

- **Bank change** — speaks the assigned entity's name whenever you switch banks (Control Mode only). Requires the **System Announcements** switch to be on.
- **Triple press** — re-announces the current bank's entity name. Requires **System Announcements** to be on.
- **Value announcement** — speaks the entity's current value after the knob settles (~600 ms debounce). Each bank has a dedicated **Announce Value** switch (`switch.{device_suffix}_bank_N_announce_value`) — only banks with this switch on will announce.

The blueprint takes one input: **Device Suffix**. The TTS service and media player are read automatically from the values configured in Pivot's integration settings — no per-blueprint configuration needed. All other entity IDs are derived from the suffix.

Supported domains and what is spoken for value announcements:

| Domain | Announcement |
| --- | --- |
| `climate` | *"Temperature 22 degrees"* (reads `temperature` attribute) |
| `cover` | *"50 percent open"* (reads `current_position` attribute) |
| `light` | *"Brightness 60 percent"* (knob value) |
| `media_player` | *"Volume 40 percent"* (knob value) |
| `fan` | *"Speed 60 percent"* (knob value) |
| `number` | *"Set to 25 watts"* (entity state + unit) |

Value announcements only fire when the knob is physically turned — changes made by external automations, dashboards, or voice commands are not announced.

### Muting announcements

**Mute Announcements** (`switch.{device_suffix}_mute_announcements`) silences all TTS from the announce and timer blueprints without changing any other setting. Useful for quiet hours or when the device is in a shared space.

---

## Supported entity domains

When an assigned entity is changed externally — by a voice command, another dashboard, an automation, or a physical switch — Pivot automatically syncs the new state back into the bank value. The LED gauge will update to reflect the change without any additional configuration.

| Domain | Knob (Control Mode) | Button press |
| --- | --- | --- |
| `light` | Brightness % | Toggle on/off |
| `media_player` | Volume (0–100%) | Play/pause |
| `fan` | Speed % | Toggle on/off |
| `climate` | Temperature (16–30°C) | Toggle on/off |
| `cover` | Position % | Toggle open/close |
| `input_number` / `number` | Value scaled to entity min–max | — |
| `switch` / `input_boolean` | — | Toggle |
| `scene` | — | Activate |
| `script` | — | Run |

> **Passive banks show no gauge.** When a bank is assigned to a passive domain (switch, input_boolean, scene, or script), the LEDs turn off — there is no value for the knob to control, so nothing is shown. The bank colour ring still appears briefly while pressing and turning to switch banks as normal.

---

## Events

Pivot fires the following events on the HA event bus regardless of setup mode.

### `pivot_bank_changed`

Fired whenever the active bank changes.

| Field | Description |
| --- | --- |
| `suffix` | Device suffix |
| `bank` | New active bank (1–4) |
| `bank_entity` | Entity assigned to the new active bank (empty string if unassigned) |

---

### `pivot_knob_turn`

Fired whenever the knob is turned in Control Mode.

| Field | Description |
| --- | --- |
| `suffix` | Device suffix |
| `bank` | Active bank (1–4) |
| `bank_entity` | Entity assigned to the active bank |
| `value` | New value (0–100) |
| `delta` | Change since last turn (positive = clockwise) |

### `pivot_button_press`

Fired on every button press regardless of mode.

| Field | Description |
| --- | --- |
| `suffix` | Device suffix |
| `bank` | Active bank (1–4) |
| `bank_entity` | Entity assigned to the active bank |
| `press_type` | `single_press`, `double_press`, `triple_press`, or `long_press` |
| `control_mode` | `true` if in Control Mode |

> See [Custom Automations](#custom-automations) below for examples using these events.

---

## Built-in press behaviours

| Press type | Built-in behaviour |
| --- | --- |
| `single_press` | Toggles or activates the active bank's assigned entity (requires entity assigned; handled natively by the integration) |
| `double_press` | Toggles Control Mode on/off |
| `triple_press` | Announces the active bank's assigned entity name via TTS (if announcements configured) |
| `long_press` | No built-in behaviour |

Custom automations triggered by these events always stack on top of built-in behaviour — they do not replace it. If you want a press that only runs your automation:

- Use `long_press` — no built-in behaviour regardless of entity assignment
- Leave the bank entity field empty and use `single_press` — with no entity assigned, Pivot has nothing to toggle

→ [Custom Automations](#custom-automations) — examples and patterns for building your own.

---

## Custom Automations

In Manual mode, Pivot creates all the required entities and fires events on the HA event bus, but does not create any scripts or automations itself — you build everything from scratch.

In Blueprint mode, custom automations are an optional layer you can add on top of the installed blueprints. Because Pivot fires events on every interaction regardless of mode, you can trigger your own automations in response to things the built-in behaviour does not handle — like using a button press on a specific bank to trigger a shell command or run a scene.

Pivot events carry enough context to let automations be as specific or as broad as you need:

- which Pivot device was used
- which bank was active
- which entity is assigned to that bank
- which button press type occurred (`single_press`, `long_press`, etc.)
- the new value and direction of a knob turn

---

### Example: Controlling a light in Manual mode

In Manual mode you need to handle both the knob and the button yourself. The example below sets a light's brightness from the knob value and toggles it with a single press, for a device with suffix `ha_voice_lounge` and Bank 1 assigned to `light.living_room`.

#### Knob — set brightness

{% raw %}
```yaml
alias: Pivot Manual - Living Room Brightness

triggers:
  - trigger: event
    event_type: pivot_knob_turn
    event_data:
      suffix: ha_voice_lounge
      bank: 1

actions:
  - action: light.turn_on
    target:
      entity_id: light.living_room
    data:
      brightness_pct: "{{ trigger.event.data.value }}"

mode: single
```
{% endraw %}

#### Button — toggle on/off

{% raw %}
```yaml
alias: Pivot Manual - Living Room Toggle

triggers:
  - trigger: event
    event_type: pivot_button_press
    event_data:
      suffix: ha_voice_lounge
      bank: 1
      press_type: single_press

actions:
  - action: light.toggle
    target:
      entity_id: light.living_room

mode: single
```
{% endraw %}

Create one pair of automations like this per bank. You can use any entity domain and any action — the pattern is the same regardless of what you are controlling.

> **Available event fields** — see the [Events](#events) section above for the full list of fields available on `pivot_knob_turn` and `pivot_button_press` events.

---

### Example: Play/Pause computer media while Bank 1 is assigned to computer volume

This example adds extra behaviour on top of the integration's built-in toggle. Bank 1 is already controlling computer volume via an `input_number` helper — this automation adds a play/pause shell command to the same button press, but only fires it when that specific helper is assigned to Bank 1. Any other bank behaves normally.

The helper entity is `input_number.example_computer_volume` and the shell command is `shell_command.music_playpause`.

{% raw %}
```yaml
alias: Pivot Example - Computer Play Pause

triggers:
  - trigger: state
    entity_id: event.example_vpe_button_press
    not_from:
      - unavailable
    not_to:
      - unavailable
      - unknown

conditions:
  - condition: state
    entity_id: event.example_vpe_button_press
    attribute: event_type
    state: single_press

  - condition: numeric_state
    entity_id: number.ha_voice_lounge_active_bank
    above: 0
    below: 2

  - condition: template
    value_template: >
      {{ states('text.example_pivot_bank_0_entity') == 'input_number.example_computer_volume' }}

actions:
  - action: shell_command.music_playpause

mode: single
```
{% endraw %}

