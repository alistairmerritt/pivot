---
layout: page
title: Architecture
permalink: /architecture/
---

This page explains what the Pivot integration does inside Home Assistant, what it creates, reads and writes, and the boundaries of its access.

The full source is available at [alistairmerritt/pivot-integration](https://github.com/alistairmerritt/pivot-integration).

> **Firmware:** Pivot also uses custom firmware for the Home Assistant Voice Preview Edition. The firmware is a separate part of the project and can be reviewed independently on the [Firmware](https://alistairmerritt.github.io/pivot/firmware/) page.

---

## What the integration is

Pivot is a Home Assistant custom integration written in Python.

Its job is fairly simple:

1. **Provision entities**  
   It creates and manages the Home Assistant entities that Pivot firmware relies on, including bank values, active bank, bank assignments, colour settings and control switches.

2. **Respond to changes**  
   It listens for knob turns, bank changes, button presses and relevant external entity updates, then calls the appropriate Home Assistant services in response.

The integration runs inside Home Assistant. It does not run its own web server, connect to a separate backend or cloud service, or access anything outside your Home Assistant instance.

---

## Trust and security model

Pivot is an independent project and is not affiliated with Nabu Casa or the ESPHome team. It is built and maintained by [u/Pivotonian](https://www.reddit.com/user/Pivotonian/), a long-time Home Assistant community contributor.

The integration is designed to remain local and deliberately narrow in scope. It uses standard Home Assistant entity platforms, service calls, event listeners and device APIs. [What it does not do](#what-it-does-not-do) lists the specific boundaries.

Both the integration and firmware are open source and can be inspected before installation.

For the integration, the modules below show how the code is divided and what each part is responsible for. The firmware can be reviewed separately at [alistairmerritt/pivot-firmware](https://github.com/alistairmerritt/pivot-firmware), including the complete `home-assistant-voice.yaml`.

The [Firmware](/pivot/firmware/) page also explains what leaves the device, how microphone audio is handled, and how to compare Pivot's configuration with the official Voice PE firmware.

---

## Module overview

The integration is split into a small set of modules with defined roles.

| File | Responsibility |
|---|---|
| `__init__.py` | Setup, teardown and wiring listeners together |
| `bank_control.py` | Knob changes, bank switching, gauge sync and external state sync |
| `button.py` | Button press handling and `pivot_button_press` event firing |
| `entity_mappings.py` | Maps a 0–100 Pivot value to the correct Home Assistant service call for each supported domain |
| `announcements.py` | Formats and triggers spoken announcements |
| `mirror.py` | Watches assigned lights and mirrors their colour into the bank colour entity |
| `device_sync.py` | Pushes settings to the device after Home Assistant starts so they remain correct after a restart |
| `blueprints.py` | Sends a one-time notification on first setup with links to import the optional timer blueprints from GitHub |
| `config_flow.py` | Setup flow and options flow |
| `const.py` | Entity definitions, constants and shared configuration |
| platform files | Entity platform implementations such as `number`, `text`, `switch`, `binary_sensor`, `light` and `select` |

The separation is mainly there to keep the integration easier to inspect and reason about. It is still one system, but each module has a narrower role.

---

## What it creates

On setup, the integration registers a device and creates a set of standard Home Assistant entities.

These are normal Home Assistant entities using standard platforms.

### Per device, per bank

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

Timer entities are also created by the integration and are disabled by default:

- `number` – timer duration
- `select` – timer state
- `text` – timer end time
- `text` – restore value for Show Control Value (diagnostic)

All entity IDs follow a stable pattern:

`{platform}.{device_suffix}_{key}`

They are pinned explicitly so they remain stable across Home Assistant restarts and device renames.

### Why so many entities?

Around forty entities per device, plus four timer entities that are disabled by default, is a fair thing to question.

The reason is that these entities form the interface between the integration and the firmware.

ESPHome has no private side channel into Home Assistant. If the firmware needs to know which bank is active, what value it holds, what colour it should use or whether a setting is enabled, that information needs to exist somewhere the device can subscribe to. Pivot uses normal Home Assistant entities for that purpose.

In practice, the entities act as shared values between the two halves of Pivot.

This is deliberate. Pivot does not maintain a separate private state store for the values shared with the firmware. Bank values, assignments, colours and settings remain visible in Home Assistant, where they can be inspected in Developer Tools, used in automations or displayed on dashboards.

---

## What it reads

Pivot reads the state it needs in order to operate and keep the firmware in sync.

### Assigned entity states

When a bank is active, Pivot may read the current state of the assigned entity so the gauge and firmware state can stay in sync.

Examples include:

- brightness from a light
- volume from a media player
- percentage from a fan
- target temperature from a climate entity
- position from a cover, or whether it is open or closed if it cannot report a position
- whether a cover accepts a position, based on its supported features, which determines whether its bank is passive
- on/off state from a switch or `input_boolean`, shown on the ring
- value from a `number` or `input_number`

### Its own entities

Pivot also reads its own entities to determine things such as:

- which bank is active
- which entity each bank is assigned to
- whether control mode is enabled
- whether announcements are enabled
- whether light mirroring is enabled

### Device registry lookup

During setup, Pivot performs a read-only lookup in Home Assistant's registry so it can locate the relevant event entity for the ESPHome device.

When pushing settings, it also reads the device's current name from its ESPHome configuration entry so the sync continues to work if the device is renamed.

Only the device name is read for this purpose. Pivot does not read the stored ESPHome password or encryption key, and it does not modify device registry data.

---

## What it writes

Actions against assigned entities are made through normal Home Assistant service calls. Pivot also updates its own entities as part of keeping the integration and firmware in sync.

It does not bypass Home Assistant to control assigned devices directly.

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

These calls are made in response to Pivot input or explicit sync behaviour.

### Writes to Pivot's own entities

Pivot also updates its own entities when needed, for example:

- updating bank values so the gauge reflects an external change
- storing TTS and media player selections from the integration settings
- updating colour values used by the firmware

### Repairs notices

If an integration entry is linked to a device whose button events Pivot cannot hear, Pivot raises a notice in Home Assistant's **Repairs** section (**Settings → Repairs**).

The notice is removed again once button events can be detected or the integration entry is deleted.

### Settings push to the device

Once Home Assistant has fully started, Pivot pushes its current settings to the ESPHome device using the `pivot_sync_settings_v2` action exposed by the firmware.

Home Assistant exposes this as:

`esphome.{device_name}_pivot_sync_settings_v2`

The current ESPHome device name is looked up for each attempt, so the sync continues to work if the device is renamed.

The sync sends:

- Control Mode
- Show Control Value
- Dim LEDs When Idle
- the per-bank Mirror Light settings
- the per-bank passive flags
- the active bank
- each bank's current value
- both sets of bank colours

This is a normal Home Assistant service call. It does not bypass Home Assistant.

The extra sync is needed because the ESPHome integration normally forwards state *changes* to subscribed devices. During a Home Assistant restart, the Voice PE can reconnect before Pivot's entities have finished restoring. If an entity is simply restored to its previous value, there may be no new state change for ESPHome to forward.

The settings push makes sure the firmware receives the current values regardless of startup order.

It also runs when the device connects later and retries with backoff until a successful attempt is confirmed. If the sync cannot complete, Pivot logs a warning.

### Blueprint notification

On first setup, Pivot sends a one-time Home Assistant notification with links to import the optional timer blueprints from GitHub.

The blueprints are not installed or copied automatically. Importing them is optional and user-initiated.

---

## What it does not do

Pivot does **not**:

- make external HTTP requests
- call external APIs
- send telemetry
- access Home Assistant credentials or tokens
- write to `configuration.yaml`
- write to `automations.yaml`
- write to `scripts.yaml`
- maintain its own database
- continuously poll Home Assistant in the background

The integration is event-driven and reacts when relevant state changes occur.

---

## Scope of control

Pivot does not scan your Home Assistant instance and start controlling entities on its own.

Its control is limited to two places:

1. **the entity explicitly assigned to a bank**
2. **Pivot's own entities and settings**

That scope is intentional. Pivot only controls the Home Assistant entities you assign to it.

---

## Why this needs to be a custom integration

A fair question is: why not just do this with blueprints?

- **Entity provisioning** – blueprints cannot create entities, and Pivot needs real entities with stable IDs and device registration for bank values, assignments, colours, switches and timer state.
- **State restoration** – Pivot uses Home Assistant's restore-capable entity classes so values survive a restart in the normal way.
- **Loop prevention** – handling two-way state synchronisation without treating an external update as a new knob turn is more reliable to manage in Python than through a chain of automations.
- **Responsiveness** – knob changes arrive quickly, and native callbacks let Pivot handle them directly without relying on multiple automations and templates.
- **Device model** – Pivot behaves like a normal Home Assistant device, with grouped entities and a consistent interface between the integration and firmware.

---

## Limitations and design boundaries

Pivot is intentionally narrow in scope.

A few things to keep in mind:

- supported behaviour depends on the assigned entity domain
- some domains use simple on/off or trigger-style interactions rather than continuous control
- some behaviour is entity-dependent because different Home Assistant integrations expose different attributes and capabilities
- the timer blueprints are optional and imported separately — they are not required for Pivot to work
- firmware and integration versions should be kept in sync — see the compatibility table on the [Changelog](/pivot/changelog/) page

The goal is not to abstract every possible Home Assistant entity perfectly. It is to provide a stable, predictable control layer for the supported use cases.

---

## Source

The integration is open source and Apache 2.0 licensed at [alistairmerritt/pivot-integration](https://github.com/alistairmerritt/pivot-integration).
