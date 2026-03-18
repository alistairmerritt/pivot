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
| **Automatic** | Writes `pivot_{suffix}_bank_toggle.yaml` and `pivot_{suffix}_announcements.yaml`, adds a single `!include` line to your `scripts.yaml` and `automations.yaml`. Fully managed — created and removed automatically. |
| **Blueprints** | Copies blueprint files into `/config/blueprints/`. You create the automations yourself from the HA UI. |
| **Manual** | Pivot does not touch any YAML files. Bank control and event firing still work — use the fired events to build your own automations. |

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

### Binary sensor entities

| Entity | Purpose |
| --- | --- |
| `binary_sensor.{suffix}_bank_0_passive` | On when Bank 1 entity is a scene or script (knob disabled) |
| `binary_sensor.{suffix}_bank_1_passive` | On when Bank 2 entity is a scene or script (knob disabled) |
| `binary_sensor.{suffix}_bank_2_passive` | On when Bank 3 entity is a scene or script (knob disabled) |
| `binary_sensor.{suffix}_bank_3_passive` | On when Bank 4 entity is a scene or script (knob disabled) |

---

## Mirror light colour

Each bank has an optional **Mirror light colour** switch (`switch.{suffix}_bank_N_mirror_light`). When enabled for a bank that is assigned to an RGB light, the bank's LED ring colour will match the current colour of that light instead of the fixed default bank colour.

- **Mirror on, light on** — LED ring uses the light's current RGB colour, updating whenever the light changes colour.
- **Mirror on, light off** — LED ring stays at the last mirrored colour until the light comes back on.
- **Mirror off** — LED ring returns to the colour set in the bank's colour picker. If you haven't changed it, this will be the default bank colour (Blue, Orange, Green, or Purple).

This is a persistent per-bank setting, not a temporary effect. It only applies to RGB lights — if the assigned entity is not a compatible RGB light, the bank falls back to its default colour automatically.

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
| `switch` / `input_boolean` | — | Toggle |
| `scene` | — | Activate |
| `script` | — | Run |

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

---

## How Automatic mode manages files

Pivot writes its own dedicated files that it fully owns. It makes only two types of change to your `scripts.yaml` and `automations.yaml`:

1. **On first setup** — appends a single `!include` line to each file. It never reads, parses, or rewrites the rest of the file content.
2. **On removal or mode change** — removes only the `!include` line it previously added. Nothing else is touched.

| File | Description |
| --- | --- |
| `pivot_{suffix}_bank_toggle.yaml` | Bank toggle script |
| `pivot_{suffix}_announcements.yaml` | Announcements automation |

Before appending the `!include` line for the first time, Pivot backs up your files:
- `scripts.yaml.pivot_backup`
- `automations.yaml.pivot_backup`

Pivot also validates its generated YAML before writing — if anything is invalid, it aborts and logs an error rather than writing a broken file.
