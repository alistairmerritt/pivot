---
layout: page
title: Architecture
permalink: /architecture/
---

This page is for people who want to understand exactly what the Pivot integration does before installing it — what it creates, what it reads, what it writes, and why it needs to exist as a custom integration rather than a set of blueprints.

The full source is at [alistairmerritt/pivot-integration](https://github.com/alistairmerritt/pivot-integration).

---

## What the integration is

The Pivot integration is a Home Assistant custom component written in Python. It has two jobs:

1. **Provision entities** — create and manage the HA entities the firmware reads and writes (bank values, active bank, bank assignments, control switches, etc.)
2. **React to state changes** — listen for knob turns, bank switches, button presses, and external entity changes, and call the appropriate HA services in response

It does not run a server, make external network requests, or access anything outside your local HA instance.

---

## Module breakdown

The integration is split into focused modules. Each file has a single, clear responsibility.

| File | Lines | Responsibility |
| --- | --- | --- |
| `__init__.py` | ~145 | Setup and teardown — wires everything together |
| `bank_control.py` | ~430 | Knob turn → apply value; bank switch → sync gauge; external change → sync gauge |
| `button.py` | ~170 | Button press events → toggle entity; fires `pivot_button_press` |
| `entity_mappings.py` | ~150 | Translates a 0–100 knob value into the right HA service call per domain |
| `announcements.py` | ~70 | Formats TTS messages and calls `tts.speak` |
| `mirror.py` | ~165 | Watches assigned lights and writes hex colour to the bank colour text entity |
| `blueprints.py` | ~90 | Copies bundled blueprint YAML files into `config/blueprints/` on first run |
| `config_flow.py` | ~430 | Setup and options UI (config flow steps) |
| `const.py` | ~365 | All entity definitions and constants |
| Platform files | ~300 total | `number.py`, `switch.py`, `text.py`, `binary_sensor.py`, `light.py`, `select.py` |

---

## What it creates

On setup, the integration registers a device and provisions the following entities in Home Assistant. All of these use standard HA entity platforms — they behave exactly like any other HA entity of that type.

**Per device (4 banks):**
- 4 × `number` — bank values (0–100%)
- 4 × `text` — bank entity assignments (stores the HA entity ID each bank controls)
- 4 × `text` — bank LED colours (hex, read by firmware)
- 4 × `text` — bank configured colours (hex, read by firmware)
- 4 × `binary_sensor` — passive flags (on when a bank's entity is a scene/script/switch — tells the firmware to disable the knob)
- 4 × `switch` — mirror light (whether to mirror the assigned light's colour)
- 4 × `switch` — announce value (whether to speak the value after the knob settles)
- 4 × `light` — bank colour pickers (virtual RGB lights used as colour selectors in the UI)

**Per device (shared):**
- 1 × `number` — active bank (1–4)
- 5 × `switch` — control mode, show control value, dim when idle, system announcements, mute announcements
- 2 × `text` — TTS entity and media player (written from integration settings; read by blueprints)

**Timer entities (disabled by default):**
- 1 × `number` — timer duration
- 1 × `select` — timer state
- 1 × `text` — timer end time

All entity IDs follow the pattern `{platform}.{device_suffix}_{key}` and are pinned explicitly so they are stable across HA restarts and device renames.

---

## What it reads

- **Entity states of assigned entities** — when a bank is active, the integration reads the state of the assigned entity (e.g. `light.living_room`) to sync the LED gauge to the current value.
- **Its own entity states** — to know the active bank, control mode, bank assignments, and announce settings.
- **ESPHome device registry** — once, at setup, to locate the button press event entity for the device. This is a read-only registry lookup; no device data is modified.

---

## What it writes

Everything the integration writes goes through standard HA service calls — the same calls any automation or script would make.

**To assigned entities (your lights, fans, etc.):**
- `light.turn_on` with `brightness_pct`
- `media_player.volume_set`
- `fan.set_percentage`
- `climate.set_temperature`
- `cover.set_cover_position`
- `number.set_value` / `input_number.set_value`
- `homeassistant.toggle` (for switches, input_booleans)
- `scene.turn_on`, `script.turn_on` (on button press)
- `media_player.media_play_pause`, `cover.toggle` (on button press)

**To its own entities:**
- `number.set_value` on bank value entities — to sync the LED gauge when an external change is detected
- `text.set_value` on config text entities — to write TTS and media player settings on setup

**To the filesystem (first run only):**
- Copies bundled blueprint YAML files into `config/blueprints/automation/pivot/` and `config/blueprints/script/pivot/`. This only happens when blueprint files are absent or outdated.

---

## What it does not do

- No HTTP requests to external services or APIs
- No access to your HA long-lived access token or credentials
- No writes to `configuration.yaml`, `automations.yaml`, or `scripts.yaml`
- No database writes beyond what HA's standard entity state storage provides
- No polling — all listeners are event-driven

---

## Why it needs to be a custom integration

The same question, answered directly: why can't this just be blueprints?

**Entity provisioning.** Blueprints cannot create entities. The bank value number entities, text entities, switches, and colour lights all need to exist as real HA entities with unique IDs, device registration, and state persistence across restarts. Only a custom component can do this.

**State persistence.** The integration uses HA's `RestoreEntity` and `RestoreNumber` base classes so entity values survive a restart without writing anything extra. Blueprints have no equivalent.

**Feedback loop prevention.** When the integration writes a value back to the gauge (e.g. syncing brightness after a bank switch), it tags the state change with a HA `Context(parent_id="pivot_sync")`. The listener for knob turns checks for this tag and ignores those writes, preventing the sync from being treated as a physical knob turn. This level of context tracking is not available in automations or blueprints.

**Efficiency.** Knob turns generate rapid state changes. The integration uses native Python `async_track_state_change_event` callbacks — lightweight, immediate, and free of the overhead of template evaluation or automation trigger matching. At 1% brightness steps this matters.

---

## Source

The integration source is public and MIT licensed at [alistairmerritt/pivot-integration](https://github.com/alistairmerritt/pivot-integration). The repository readme describes the module structure; the code itself is commented where behaviour is non-obvious.
