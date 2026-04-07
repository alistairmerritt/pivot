---
layout: page
title: Integration
permalink: /integration/
---

The Pivot HA integration provisions all required entities for a Pivot device and, depending on setup mode, can create the required scripts and automations automatically.

Install via HACS from [alistairmerritt/pivot-integration](https://github.com/alistairmerritt/pivot-integration).

---

## Setup modes

| Mode | What Pivot does |
| --- | --- |
| **Automatic** | Writes files to `/config/pivot/` and adds a single `!include` line to your `scripts.yaml` and `automations.yaml`. Fully managed — created and removed automatically. |
| **Blueprints** | Copies blueprint files into `/config/blueprints/`. You create the automations yourself from the HA UI. |
| **Manual** | Pivot does not touch any YAML files. Bank control and event firing still work — see [Custom Automations](#custom-automations) below for a guide to building your own. |

The setup mode can be changed at any time from the integration's **Configure** menu. Switching away from Automatic will remove the files Pivot created.

---

## Entities

### Number entities

| Entity | Purpose |
| --- | --- |
| `number.{suffix}_bank_0_value` | Bank 1 value (0–100%) |
| `number.{suffix}_bank_1_value` | Bank 2 value (0–100%) |
| `number.{suffix}_bank_2_value` | Bank 3 value (0–100%) |
| `number.{suffix}_bank_3_value` | Bank 4 value (0–100%) |
| `number.{suffix}_active_bank` | Active bank (1–4) |

### Switch entities

| Entity | Purpose |
| --- | --- |
| `switch.{suffix}_control_mode` | Control Mode vs Normal (voice) Mode |
| `switch.{suffix}_show_control_value` | Keep gauge LEDs permanently visible in control mode |
| `switch.{suffix}_dim_when_idle` | Dim gauge LEDs to 50% after 2 s of inactivity (requires Show Control Value) |
| `switch.{suffix}_announcements` | Enable/disable spoken announcements |
| `switch.{suffix}_bank_0_mirror_light` | Bank 1 — mirror assigned RGB light colour |
| `switch.{suffix}_bank_1_mirror_light` | Bank 2 — mirror assigned RGB light colour |
| `switch.{suffix}_bank_2_mirror_light` | Bank 3 — mirror assigned RGB light colour |
| `switch.{suffix}_bank_3_mirror_light` | Bank 4 — mirror assigned RGB light colour |

### Text entities

| Entity | Purpose |
| --- | --- |
| `text.{suffix}_bank_0_entity` | Entity assigned to Bank 1 |
| `text.{suffix}_bank_1_entity` | Entity assigned to Bank 2 |
| `text.{suffix}_bank_2_entity` | Entity assigned to Bank 3 |
| `text.{suffix}_bank_3_entity` | Entity assigned to Bank 4 |

> **Reserved value:** Setting a bank entity to `timer` (lowercase) marks that bank as a timer bank for use with the [Pivot Timer blueprint](/pivot/timer/). The knob has no effect on a timer bank — the LED gauge is managed entirely by the blueprint. Any other value is treated as a Home Assistant entity ID.

### Binary sensor entities

| Entity | Purpose |
| --- | --- |
| `binary_sensor.{suffix}_bank_0_passive` | On when Bank 1 entity is a scene, script, switch, or input_boolean (knob disabled) |
| `binary_sensor.{suffix}_bank_1_passive` | On when Bank 2 entity is a scene, script, switch, or input_boolean (knob disabled) |
| `binary_sensor.{suffix}_bank_2_passive` | On when Bank 3 entity is a scene, script, switch, or input_boolean (knob disabled) |
| `binary_sensor.{suffix}_bank_3_passive` | On when Bank 4 entity is a scene, script, switch, or input_boolean (knob disabled) |

### Timer entities

These entities are provisioned per device but **disabled by default**. Enable them individually in the HA entity registry if you want to use the [Pivot Timer](/pivot/timer/) feature.

| Entity | Purpose |
| --- | --- |
| `number.{suffix}_timer_duration` | Timer duration in minutes (1–60, default 25) |
| `select.{suffix}_timer_state` | Timer state — idle, running, or paused |
| `text.{suffix}_timer_end` | Internal — stores the countdown end time while the timer is running |

> **Do not rename Pivot entity IDs.** The firmware and integration use your `device_suffix` to build entity IDs at runtime. Renaming any of these entities in Home Assistant will break the connection between the firmware and the integration. If you need to label entities more clearly, change the entity's **Name** — not its **Entity ID**.

---

## Mirror light colour

Each bank has an optional **Mirror light colour** switch (`switch.{suffix}_bank_N_mirror_light`). When enabled for a bank that is assigned to an RGB light, the bank's LED ring colour will match the current colour of that light instead of the fixed default bank colour.

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

Pivot fires two events on the HA event bus regardless of setup mode.

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
| `single_press` | Toggles or activates the active bank's assigned entity (Automatic mode only, requires entity assigned) |
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

If you use Automatic mode, custom automations are an optional layer you can add on top. Because Pivot fires events on every interaction regardless of mode, you can trigger your own automations in response to things the built-in behaviour does not handle — like using a button press on a specific bank to trigger a shell command or run a scene.

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

This example adds behaviour on top of Automatic mode. Bank 1 is already controlling computer volume via an `input_number` helper — this automation adds a play/pause shell command to the same button press, but only fires it when that specific helper is assigned to Bank 1. Any other bank behaves normally.

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
    entity_id: event.example_pivot_button_press
    attribute: event_type
    state: single_press

  - condition: numeric_state
    entity_id: number.example_pivot_active_bank
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

---

## How Automatic mode manages files

Pivot writes its own dedicated files to a `/config/pivot/` subfolder — they do not appear alongside your other config files in `/config/`.

| File | Description |
| --- | --- |
| `pivot/pivot_{suffix}_bank_toggle.yaml` | Bank toggle script |
| `pivot/pivot_{suffix}_announcements.yaml` | Announcements automation |

It adds a single `!include` line for each file into your `scripts.yaml` and `automations.yaml`. These are the only changes Pivot makes to those files — it never touches any other content.

On every load, Pivot rewrites its own `!include` lines from scratch: any stale, duplicate, or broken entries for its keys are removed and replaced with a single correct line. This means the integration is self-healing — if an include line ever becomes inconsistent (e.g. after a manual edit or a failed update), it will be corrected automatically without any user action.

On removal or mode change, Pivot removes the files it created and its `!include` lines from `scripts.yaml` and `automations.yaml`.

Before modifying `scripts.yaml` or `automations.yaml` for the first time, Pivot creates a one-time backup:
- `scripts.yaml.pivot_backup`
- `automations.yaml.pivot_backup`

Pivot also validates its generated YAML before writing — if anything is invalid, it aborts and logs an error rather than writing a broken file.
