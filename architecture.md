---
layout: page
title: Architecture
permalink: /architecture/
---

This page is for people who want to understand exactly what the Pivot integration does before installing it. It has been developed with transparency in mind, shaped by Home Assistant’s local, open-source and community-led ethos.

The full source is available at [alistairmerritt/pivot-integration](https://github.com/alistairmerritt/pivot-integration).

> **Firmware:** Pivot also includes custom firmware for the Home Assistant Voice Preview Edition, which is a separate component with its own installation, update and rollback considerations. Review the [Firmware](https://alistairmerritt.github.io/pivot/firmware/) page as well before proceeding.

* * *

## What the integration is

The Pivot integration is a Home Assistant custom component written in Python.

Its job is simple:

1. **Provision entities**  
   It creates and manages the Home Assistant entities that Pivot firmware relies on, such as bank values, active bank, bank assignments, colour settings, and control switches.

2. **Respond to changes**  
   It listens for knob turns, bank changes, button presses, and relevant external entity updates, then calls the appropriate Home Assistant services in response.

Pivot does **not** run a web server, make outbound HTTP requests, connect to a cloud service, or access anything outside your local Home Assistant instance.

* * *


## Trust and security model

Pivot is an independent project and is not affiliated with Nabu Casa or the ESPHome team. It is built and maintained by [u/Pivotonian](https://www.reddit.com/user/Pivotonian/), a long-time Home Assistant community contributor.

It is designed to be local-only and intentionally narrow in scope: everything it does happens inside Home Assistant using standard entity platforms, standard service calls and normal event listeners. [What it does not do](#what-it-does-not-do) lists the specifics.

Both the integration and the firmware source are public and can be inspected before installing. For the integration, the modules listed below show exactly what each file does. For the firmware, the full YAML is at [alistairmerritt/pivot-firmware](https://github.com/alistairmerritt/pivot-firmware) — specifically `home-assistant-voice.yaml`.

The firmware half is covered on the [Firmware](/pivot/firmware/) page, under *What the firmware does and does not do*: what leaves the device, how the microphone behaves, and how to diff Pivot's configuration against the official one.

* * *

## Module overview

The integration is split into a small set of modules with defined roles.

| File | Responsibility |
|---|---|
| `__init__.py` | Setup, teardown, and wiring listeners together |
| `bank_control.py` | Knob changes, bank switching, gauge sync, and external state sync |
| `button.py` | Button press handling and `pivot_button_press` event firing |
| `entity_mappings.py` | Maps a 0–100 Pivot value to the correct HA service call for each supported domain |
| `announcements.py` | Formats and triggers spoken announcements |
| `mirror.py` | Watches assigned lights and mirrors their colour into the bank colour entity |
| `device_sync.py` | Pushes settings to the device once Home Assistant has started, so they are correct after a restart |
| `blueprints.py` | Sends a one-time notification on first setup with links to import the optional timer blueprints from GitHub |
| `config_flow.py` | Setup flow and options flow |
| `const.py` | Entity definitions, constants, and shared configuration |
| platform files | Entity platform implementations such as `number`, `text`, `switch`, `binary_sensor`, `light`, and `select` |

This separation is mainly there to keep the integration easier to inspect and reason about. It is still one system, but each module has a narrower role.

* * *

## What it creates

On setup, the integration registers a device and creates a set of standard Home Assistant entities.

These are normal HA entities using standard platforms. They are not hidden objects or special internal state.

### Per device, per bank (4 banks)

For each of the four banks, Pivot creates:

- `number` – bank value (0–100)
- `text` – assigned entity ID
- `text` – live bank LED colour
- `text` – configured bank colour
- `binary_sensor` – passive flag
- `switch` – mirror light enabled
- `switch` – announce value enabled
- `light` – virtual bank colour picker

### Per device, shared

Pivot also creates shared device-level entities:

- `number` – active bank
- `switch` – control mode
- `switch` – show control value
- `switch` – dim when idle
- `switch` – system announcements
- `switch` – mute announcements
- `text` – TTS entity
- `text` – media player entity

### Timer entities

Timer entities are created as part of the integration and are disabled by default:

- `number` – timer duration
- `select` – timer state
- `text` – timer end time
- `text` – restore value for Show Control Value (diagnostic)

All entity IDs follow a stable pattern:

`{platform}.{device_suffix}_{key}`

They are pinned explicitly so they stay stable across Home Assistant restarts and device renames.

### Why so many entities?

Around forty entities per device (plus four timer entities, disabled by default) is a fair thing to question, so here is the reason: the entities **are** the interface between the integration and the firmware.

ESPHome firmware has no private side channel into Home Assistant. For the device to read a value that lives in Home Assistant — which bank is active, what colour a bank should be, whether control mode is on — that value must exist as a real entity the firmware can subscribe to. So every bank value, assignment, colour, and setting is a normal Home Assistant entity, effectively a shared variable between the two halves of the system.

This is a deliberate design choice with a useful consequence: there is no hidden state. Every value the firmware acts on is visible and inspectable in Home Assistant — in Developer Tools, in automations, on dashboards. If Pivot does something, you can see exactly which entity told it to.

* * *

## What it reads

Pivot reads only the state it needs in order to function.

### Assigned entity states

When a bank is active, Pivot may read the current state of the assigned entity so the gauge and firmware state can stay in sync.

Examples include:

- brightness from a light
- volume from a media player
- percentage from a fan
- target temperature from a climate entity
- position from a cover, or whether it is open or closed if it cannot report a position
- whether a cover accepts a position (its supported features), which decides whether its bank is passive
- on/off state from a switch or input_boolean, shown on the ring
- value from a number or input number

### Its own entities

Pivot also reads its own entities to determine things like:

- which bank is active
- which entity each bank is assigned to
- whether control mode is enabled
- whether announcements are enabled
- whether light mirroring is enabled

### Device registry lookup

During setup, Pivot performs a read-only lookup in Home Assistant’s registry so it can locate the relevant event entity for the ESPHome device.

When it pushes settings, it also reads the device's current name from its ESPHome configuration entry – only the name, never the stored password or encryption key – so the push keeps working after the device is renamed in ESPHome.

It does not modify device registry data.

* * *

## What it writes

Everything Pivot writes goes through normal Home Assistant service calls.

It does not bypass Home Assistant, write arbitrary state directly, or perform hidden mutations elsewhere.

### Writes to assigned entities

Depending on the bank assignment, Pivot may call services such as:

- `light.turn_on` with `brightness_pct`
- `media_player.volume_set`
- `fan.set_percentage`
- `climate.set_temperature`
- `cover.set_cover_position`
- `number.set_value`
- `input_number.set_value`
- `homeassistant.toggle`
- `scene.turn_on`
- `script.turn_on`
- `media_player.media_play_pause`
- `cover.toggle`

These calls are only made in response to Pivot input or explicit sync behaviour.

### Writes to Pivot’s own entities

Pivot also writes to its own entities when needed, for example:

- updating bank value entities so the gauge reflects an external change
- writing TTS and media player selections from the integration settings
- writing colour values used by the firmware

### Repairs notices

If the entry is linked to a device whose button it cannot hear, Pivot raises a notice in Home Assistant's **Repairs** (**Settings → Repairs**), and removes it again once the button can be heard or the entry is deleted.

### Settings push to the device

Once Home Assistant has fully started, Pivot calls a dedicated action on the ESPHome device – `pivot_sync_settings_v2`, exposed to Home Assistant as `esphome.{device_name}_pivot_sync_settings_v2`, where `{device_name}` is the device's current ESPHome name, looked up at each attempt – to push Control Mode, Show Control Value, Dim LEDs When Idle, the per-bank Mirror Light and passive flags, the active bank, each bank's value, and both sets of bank colours directly into the firmware. This is a standard Home Assistant service call, the same as any other action Pivot performs – it does not bypass Home Assistant.

This exists because Home Assistant's ESPHome integration only forwards genuine state *changes* to a subscribed device – a device that connects and subscribes before Pivot's entities are restored (typical during a Home Assistant restart) would otherwise never receive their values. The push guarantees the firmware has correct settings after a restart regardless of connection or restore ordering. It also runs when the device connects later, and retries on a backoff schedule until an attempt is confirmed – a warning is logged if it never succeeds.

### Blueprint notification

On first setup, Pivot sends a one-time Home Assistant notification with links to import the optional timer blueprints from GitHub. Blueprints are not copied automatically — importing them is optional and user-initiated.

* * *

## What it does not do

Pivot does **not** do any of the following:

- make external HTTP requests
- call external APIs
- send telemetry
- access Home Assistant credentials or tokens
- write to `configuration.yaml`
- write to `automations.yaml`
- write to `scripts.yaml`
- maintain its own database
- poll constantly in the background

The integration is event-driven. It reacts when relevant state changes happen.

* * *

## Scope of control

Pivot does not scan your Home Assistant instance and start controlling things on its own.

It writes to one of two places only:

1. **the entity assigned to a bank**
2. **its own helper/config entities**

That scope is intentional. Pivot only acts on the entities you explicitly assign to it.

* * *

## Why this needs to be a custom integration

A fair question is: why not just do this with blueprints?

- **Entity provisioning** – blueprints cannot create entities, and Pivot needs real ones with stable IDs and device registration for bank values, assignments, colours, switches and timer state.
- **State restoration** – Pivot uses Home Assistant's restore-capable entity classes so values survive a restart in the normal way.
- **Loop prevention** – syncing a value back into Home Assistant without mistaking it for a new knob turn is far more reliable in Python than in automations.
- **Efficiency** – knob turns come fast, and native callbacks keep the gauge responsive where a chain of automations and templates would not.
- **Device model** – Pivot behaves like a real Home Assistant device, with grouped entities and a consistent contract between firmware and integration.

* * *

## Limitations and design boundaries

Pivot is intentionally narrow in scope.

A few things to keep in mind:

- supported behaviour depends on the assigned entity domain
- some domains are simple on/off or trigger-style interactions rather than continuous control
- some behaviour is entity-dependent, because different Home Assistant integrations expose different attributes and capabilities
- the timer blueprints are optional and imported separately — they are not required for Pivot to work
- firmware and integration versions should be kept in sync – see the compatibility table on the [Changelog](/pivot/changelog/) page

The goal is not to abstract every possible Home Assistant entity perfectly. The goal is to provide a stable, predictable control layer for the supported use cases.

* * *

## Source

The integration source is public and Apache 2.0 licensed at [alistairmerritt/pivot-integration](https://github.com/alistairmerritt/pivot-integration). This page is meant to tell you what to look for and where.

* * *
