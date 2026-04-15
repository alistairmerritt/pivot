---
layout: page
title: Custom Automations
permalink: /automations/
---

Because Pivot fires events on every interaction, you can trigger your own automations in response to any dial turn or button press. Each example below includes an importable blueprint and a raw YAML version.

Pivot events carry enough context to let automations be as specific or as broad as you need:

- which Pivot device was used
- which bank was active
- which entity is assigned to that bank
- which button press type occurred (`single_press`, `long_press`, etc.)
- the new value and direction of a knob turn

For the full list of event fields, see the [Events](/pivot/integration/#events) section of the Integration page.

---

| Automation | What it does |
|---|---|
| [Colour temperature control](#colour-temperature) | Dial adjusts light warmth via an input_number helper, press toggles |
| [Button press with configurable action](#button-press-action) | Run any action on button press, optionally filtered by assigned entity |
| [Light brightness and toggle](#light-brightness) | Dial sets brightness, press toggles on/off |
| [Media player volume and power toggle](#media-player-tv) | Dial sets volume, press toggles TV on/off |

---

## Colour temperature control — dial adjusts warmth, press toggles light
{: #colour-temperature}

Assigning a light directly to a bank gives you brightness control — the 0–100 dial value maps to brightness percentage. To control colour temperature instead, assign an `input_number` helper to the bank and use this automation to translate that 0–100 value into the correct kelvin for your specific light.


The automation also keeps the helper in sync if the light's temperature is changed from elsewhere (e.g. via the HA UI or another automation), so the dial position always reflects the current state.

Create a helper: **Settings → Devices & Services → Helpers → Number**, with a range of 0–100 and step 1. Then assign it to the bank on your device.

#### Blueprint

[![Import Blueprint](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https://raw.githubusercontent.com/alistairmerritt/pivot/main/assets/blueprints/pivot-colour-temperature.yaml)

{% raw %}
```yaml
blueprint:
  name: Pivot - Colour Temperature Control
  description: >
    Control a light's colour temperature with the Pivot dial via an input_number helper.
    Single press toggles the light. Dial position stays in sync with the light.
  domain: automation
  input:
    suffix:
      name: Device suffix
      description: The suffix of your Pivot device (e.g. ha_voice_lounge)
      selector:
        text: {}
    bank:
      name: Bank number
      description: The bank number to listen on (1–4)
      selector:
        number:
          min: 1
          max: 4
          mode: box
    input_number_entity:
      name: Input number helper
      description: The input_number entity assigned to this bank (range 0–100)
      selector:
        entity:
          domain: input_number
    light_entity:
      name: Light
      description: The light to control
      selector:
        entity:
          domain: light

triggers:
  - trigger: state
    entity_id: !input input_number_entity
    id: dial
  - trigger: event
    event_type: pivot_button_press
    event_data:
      suffix: !input suffix
      bank: !input bank
      press_type: single_press
    id: press
  - trigger: state
    entity_id: !input light_entity
    id: sync

actions:
  - variables:
      light_entity: !input light_entity
      input_number_entity: !input input_number_entity
      percent: "{{ states(input_number_entity) | float(0) }}"
      min_mired: "{{ state_attr(light_entity, 'min_mireds') | float(153) }}"
      max_mired: "{{ state_attr(light_entity, 'max_mireds') | float(500) }}"
      target_kelvin: >-
        {{ (1000000 / ((min_mired | int) + ((percent / 100) * ((max_mired | int)
        - (min_mired | int))))) | round(0) | int }}
      current_mired: "{{ state_attr(light_entity, 'color_temp') | float(0) }}"
      sync_percent: >-
        {{ (((current_mired - min_mired) / (max_mired - min_mired)) * 100) | round(0) }}
  - choose:
      - conditions:
          - condition: trigger
            id: dial
        sequence:
          - action: light.turn_on
            target:
              entity_id: !input light_entity
            data:
              color_temp_kelvin: "{{ target_kelvin | int }}"
      - conditions:
          - condition: trigger
            id: press
        sequence:
          - action: light.toggle
            target:
              entity_id: !input light_entity
      - conditions:
          - condition: trigger
            id: sync
          - condition: template
            value_template: "{{ current_mired > 0 and max_mired > min_mired }}"
        sequence:
          - action: input_number.set_value
            target:
              entity_id: !input input_number_entity
            data:
              value: "{{ [[sync_percent, 0] | max, 100] | min }}"

mode: single
```
{% endraw %}

#### Raw automation example

{% raw %}
```yaml
alias: Pivot - Study Lamp Warmth

triggers:
  - trigger: state
    entity_id: input_number.study_lamp_warmth
    id: dial
  - trigger: event
    event_type: pivot_button_press
    event_data:
      suffix: ha_voice_lounge
      bank: 2
      press_type: single_press
    id: press
  - trigger: state
    entity_id: light.study_lamp
    id: sync

actions:
  - variables:
      percent: "{{ states('input_number.study_lamp_warmth') | float(0) }}"
      min_mired: "{{ state_attr('light.study_lamp', 'min_mireds') | float(153) }}"
      max_mired: "{{ state_attr('light.study_lamp', 'max_mireds') | float(500) }}"
      target_kelvin: >-
        {{ (1000000 / ((min_mired | int) + ((percent / 100) * ((max_mired | int)
        - (min_mired | int))))) | round(0) | int }}
      current_mired: "{{ state_attr('light.study_lamp', 'color_temp') | float(0) }}"
      sync_percent: >-
        {{ (((current_mired - min_mired) / (max_mired - min_mired)) * 100) | round(0) }}
  - choose:
      - conditions:
          - condition: trigger
            id: dial
        sequence:
          - action: light.turn_on
            target:
              entity_id: light.study_lamp
            data:
              color_temp_kelvin: "{{ target_kelvin | int }}"
      - conditions:
          - condition: trigger
            id: press
        sequence:
          - action: light.toggle
            target:
              entity_id: light.study_lamp
      - conditions:
          - condition: trigger
            id: sync
          - condition: template
            value_template: "{{ current_mired > 0 and max_mired > min_mired }}"
        sequence:
          - action: input_number.set_value
            target:
              entity_id: input_number.study_lamp_warmth
            data:
              value: "{{ [[sync_percent, 0] | max, 100] | min }}"

mode: single
```
{% endraw %}

---

## Button press with configurable action — e.g. dial controls volume, press to play/pause media
{: #button-press-action}

The built-in bank toggle handles turning entities on and off, but sometimes a button press should do something extra — like triggering play/pause on your computer when the bank is controlling system volume, or running a scene when a specific entity is active.

This automation fires any action you choose on button press, with an optional check that a particular entity is currently assigned to the bank. That check is what makes it context-aware: reassign the bank to something else and the automation automatically stops firing.

#### Blueprint

[![Import Blueprint](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https://raw.githubusercontent.com/alistairmerritt/pivot/main/assets/blueprints/pivot-button-press-action.yaml)

{% raw %}
```yaml
blueprint:
  name: Pivot - Button Press Action
  description: >
    Run a configurable action on button press for a specific Pivot device and bank,
    with an optional check that a particular entity is assigned to that bank.
  domain: automation
  input:
    suffix:
      name: Device suffix
      description: The suffix of your Pivot device (e.g. ha_voice_lounge)
      selector:
        text: {}
    bank:
      name: Bank number
      description: The bank number to listen on (1–4)
      selector:
        number:
          min: 1
          max: 4
          mode: box
    press_type:
      name: Press type
      description: Which press type triggers the action
      default: single_press
      selector:
        select:
          options:
            - single_press
            - double_press
            - triple_press
            - long_press
    assigned_entity:
      name: Assigned entity (optional)
      description: >
        Only fire the action if this entity is currently assigned to the bank.
        Leave blank to fire regardless of what is assigned.
      default: ""
      selector:
        text: {}
    action:
      name: Action
      description: Action to run when the button is pressed
      selector:
        action: {}

triggers:
  - trigger: event
    event_type: pivot_button_press
    event_data:
      suffix: !input suffix
      bank: !input bank
      press_type: !input press_type

conditions:
  - condition: template
    value_template: >
      {% set entity = 'text.' ~ suffix ~ '_bank_' ~ bank ~ '_entity' %}
      {% set assigned = assigned_entity %}
      {{ assigned == '' or states(entity) == assigned }}

actions:
  - sequence: !input action

mode: single
```
{% endraw %}

#### Raw automation example

{% raw %}
```yaml
alias: Pivot - Computer Play/Pause

triggers:
  - trigger: event
    event_type: pivot_button_press
    event_data:
      suffix: ha_voice_lounge
      bank: 1
      press_type: single_press

conditions:
  - condition: template
    value_template: >
      {{ states('text.ha_voice_lounge_bank_1_entity') == 'input_number.computer_volume' }}

actions:
  - action: shell_command.music_playpause

mode: single
```
{% endraw %}

---

## Light brightness and toggle — dial sets brightness, press toggles on/off
{: #light-brightness}

The simplest custom automation pattern — the dial controls a light's brightness directly and a single press toggles it. Useful if you want more precise brightness control than Pivot's built-in toggle provides, or if you're using a bank for a light that isn't directly assignable (e.g. a light group).

Unlike the colour temperature example, this one uses the raw `pivot_knob_turn` event directly rather than going via an `input_number` helper — the dial value is passed straight through as a brightness percentage.

#### Blueprint

[![Import Blueprint](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https://raw.githubusercontent.com/alistairmerritt/pivot/main/assets/blueprints/pivot-light-brightness.yaml)

{% raw %}
```yaml
blueprint:
  name: Pivot - Light Brightness & Toggle
  description: >
    Control a light's brightness with the Pivot dial. Single press toggles the light on/off.
  domain: automation
  input:
    suffix:
      name: Device suffix
      description: The suffix of your Pivot device (e.g. ha_voice_lounge)
      selector:
        text: {}
    bank:
      name: Bank number
      description: The bank number to listen on (1–4)
      selector:
        number:
          min: 1
          max: 4
          mode: box
    light_entity:
      name: Light
      description: The light to control
      selector:
        entity:
          domain: light

triggers:
  - trigger: event
    event_type: pivot_knob_turn
    event_data:
      suffix: !input suffix
      bank: !input bank
    id: knob
  - trigger: event
    event_type: pivot_button_press
    event_data:
      suffix: !input suffix
      bank: !input bank
      press_type: single_press
    id: press

actions:
  - choose:
      - conditions:
          - condition: trigger
            id: knob
        sequence:
          - action: light.turn_on
            target:
              entity_id: !input light_entity
            data:
              brightness_pct: "{{ trigger.event.data.value }}"
      - conditions:
          - condition: trigger
            id: press
        sequence:
          - action: light.toggle
            target:
              entity_id: !input light_entity

mode: single
```
{% endraw %}

#### Raw automation example

{% raw %}
```yaml
alias: Pivot - Living Room Brightness

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

{% raw %}
```yaml
alias: Pivot - Living Room Toggle

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

---

## Media player volume and power toggle — dial controls volume, press toggles TV on/off
{: #media-player-tv}

When a media player is assigned to a bank, Pivot controls volume with the dial and fires play/pause on a single press. For a TV, play/pause on the TV's own media player entity is rarely what you want — if you're assigning a TV to a bank, you're almost certainly after power control. This automation adds that: assign your TV to the bank as normal, and single press will toggle power in addition to the native play/pause. In practice the play/pause is harmless — if the TV is turning off it doesn't matter, and if it's turning on it's typically ignored.

If you do use the TV's built-in streaming apps and want play/pause separately, assign that streaming device to a different bank instead.

#### Blueprint

[![Import Blueprint](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https://raw.githubusercontent.com/alistairmerritt/pivot/main/assets/blueprints/pivot-media-player-tv.yaml)

{% raw %}
```yaml
blueprint:
  name: Pivot - Media Player Volume & Power Toggle
  description: >
    Control a media player's volume with the Pivot dial and toggle power on single press.
    Designed for TVs where you want the dial to set volume and a press to turn the TV on or off.
    Leave the bank entity unassigned so the button press has no built-in behaviour to conflict with.
  domain: automation
  input:
    suffix:
      name: Device suffix
      description: The suffix of your Pivot device (e.g. ha_voice_lounge)
      selector:
        text: {}
    bank:
      name: Bank number
      description: The bank number to listen on (1–4)
      selector:
        number:
          min: 1
          max: 4
          mode: box
    media_player_entity:
      name: Media player
      description: The TV or media player to control
      selector:
        entity:
          domain: media_player

triggers:
  - trigger: event
    event_type: pivot_knob_turn
    event_data:
      suffix: !input suffix
      bank: !input bank
    id: knob
  - trigger: event
    event_type: pivot_button_press
    event_data:
      suffix: !input suffix
      bank: !input bank
      press_type: single_press
    id: press

actions:
  - choose:
      - conditions:
          - condition: trigger
            id: knob
        sequence:
          - action: media_player.volume_set
            target:
              entity_id: !input media_player_entity
            data:
              volume_level: "{{ trigger.event.data.value / 100 }}"
      - conditions:
          - condition: trigger
            id: press
        sequence:
          - action: media_player.toggle
            target:
              entity_id: !input media_player_entity

mode: single
```
{% endraw %}

#### Raw automation example

{% raw %}
```yaml
alias: Pivot - Living Room TV

triggers:
  - trigger: event
    event_type: pivot_knob_turn
    event_data:
      suffix: ha_voice_lounge
      bank: 3
    id: knob
  - trigger: event
    event_type: pivot_button_press
    event_data:
      suffix: ha_voice_lounge
      bank: 3
      press_type: single_press
    id: press

actions:
  - choose:
      - conditions:
          - condition: trigger
            id: knob
        sequence:
          - action: media_player.volume_set
            target:
              entity_id: media_player.living_room_tv
            data:
              volume_level: "{{ trigger.event.data.value / 100 }}"
      - conditions:
          - condition: trigger
            id: press
        sequence:
          - action: media_player.toggle
            target:
              entity_id: media_player.living_room_tv

mode: single
```
{% endraw %}
