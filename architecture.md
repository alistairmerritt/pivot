---
layout: page
title: Architecture
permalink: /architecture/
---

This page is for people who want to understand exactly what the Pivot integration does before installing it. It has been developed with transparency in mind, shaped by Home Assistant’s local, open-source and community-led ethos.

This page explains:

- what the integration creates
- what it reads
- what it writes
- what it does **not** do
- why it exists as a custom integration rather than just a set of blueprints

The full source is available at [alistairmerritt/pivot-integration](https://github.com/alistairmerritt/pivot-integration).

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

Pivot is designed to be local-only and intentionally narrow in scope.

In practical terms:

- no cloud dependency
- no telemetry
- no external API calls
- no external HTTP requests
- no credential handling
- no custom database layer
- no direct writes to your main YAML config files

Everything it does happens inside Home Assistant using standard entity platforms, standard service calls, and normal event listeners.

If you want to inspect exactly what it does, the source is public.

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
| `blueprints.py` | Installs bundled blueprints into Home Assistant on first run when needed |
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

- `number` — bank value (0–100)
- `text` — assigned entity ID
- `text` — live bank LED colour
- `text` — configured bank colour
- `binary_sensor` — passive flag
- `switch` — mirror light enabled
- `switch` — announce value enabled
- `light` — virtual bank colour picker

### Per device, shared

Pivot also creates shared device-level entities:

- `number` — active bank
- `switch` — control mode
- `switch` — show control value
- `switch` — dim when idle
- `switch` — system announcements
- `switch` — mute announcements
- `text` — TTS entity
- `text` — media player entity

### Timer entities

Timer entities are created as part of the integration and are disabled by default:

- `number` — timer duration
- `select` — timer state
- `text` — timer end time

All entity IDs follow a stable pattern:

`{platform}.{device_suffix}_{key}`

They are pinned explicitly so they stay stable across Home Assistant restarts and device renames.

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
- position from a cover
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

### Blueprint installation

On first run, Pivot may copy bundled blueprint YAML files into:

- `config/blueprints/automation/pivot/`
- `config/blueprints/script/pivot/`

This only happens when those bundled blueprints are missing or outdated.

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

### 1. Entity provisioning

Blueprints cannot create entities.

Pivot needs real Home Assistant entities for things like:

- bank values
- active bank
- bank assignments
- colour settings
- control switches
- timer state

These entities need stable IDs, device registration, and normal HA behaviour.

### 2. State restoration

Pivot uses Home Assistant restore-capable entity classes so values survive restart in the normal HA way.

Blueprints do not provide an equivalent entity model for this.

### 3. Loop prevention

Some Pivot behaviour requires writing a synced value back into Home Assistant without that write being mistaken for a new physical control input.

That kind of state feedback control is much easier and more reliable inside a custom integration than in blueprints or automations.

### 4. Efficiency

Knob changes can happen rapidly.

Pivot uses native Home Assistant Python callbacks to respond quickly and keep gauge sync responsive, rather than relying on a heavier chain of automations, templates, and triggers.

### 5. Device model

Pivot behaves like a real Home Assistant device with grouped entities and a consistent contract between firmware and integration.

A custom integration is the right layer for that.

* * *

## Limitations and design boundaries

Pivot is intentionally narrow in scope.

A few things to keep in mind:

- supported behaviour depends on the assigned entity domain
- some domains are simple on/off or trigger-style interactions rather than continuous control
- some behaviour is entity-dependent, because different Home Assistant integrations expose different attributes and capabilities
- bundled blueprints are optional helpers, not the core of Pivot
- firmware and integration versions should be kept in sync where recommended

The goal is not to abstract every possible Home Assistant entity perfectly. The goal is to provide a stable, predictable control layer for the supported use cases.

* * *

## Source

The integration source is public and MIT licensed at [alistairmerritt/pivot-integration](https://github.com/alistairmerritt/pivot-integration).

If you want to inspect the code before installing it, this page is intended to help you understand what to look for and where.

* * *
