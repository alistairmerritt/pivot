---
layout: page
title: Custom Automations
permalink: /automations/
---

Custom automations are an optional layer you can add on top of the installed blueprints. Because Pivot fires events on every interaction, you can trigger your own automations in response to things the built-in behaviour does not handle — like using a button press on a specific bank to trigger a shell command or run a scene.

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
| [Button press with configurable action](#button-press-with-configurable-action) | Run any action on button press, optionally filtered by assigned entity |
| [Colour temperature control](#colour-temperature-control-via-input_number) | Dial adjusts light warmth via an input_number helper, press toggles |
| [Light brightness and toggle](#light-brightness-and-toggle) | Dial sets brightness, press toggles on/off |

---

## Button press with configurable action

Fires a configurable action when a specific bank's button is pressed, but only when a particular entity is assigned to that bank. Useful for stacking extra behaviour on top of the built-in toggle — for example, triggering a play/pause shell command when a media volume helper is assigned.

#### Blueprint

[![Import Blueprint](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https://raw.githubusercontent.com/alistairmerritt/pivot/main/assets/blueprints/pivot-button-press-action.yaml)

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

## Colour temperature control via input_number

Uses an `input_number` helper as the bank entity so the dial adjusts the colour temperature of a light. A single press toggles the light. The dial position stays in sync if the light is changed from elsewhere.

Create a helper: **Settings → Devices & Services → Helpers → Number**, with a range of 0–100 and step 1. Then assign it to the bank on your device.

#### Blueprint

[![Import Blueprint](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https://raw.githubusercontent.com/alistairmerritt/pivot/main/assets/blueprints/pivot-colour-temperature.yaml)

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
              entity_id: "{{ light_entity }}"
            data:
              color_temp_kelvin: "{{ target_kelvin | int }}"
      - conditions:
          - condition: trigger
            id: press
        sequence:
          - action: light.toggle
            target:
              entity_id: "{{ light_entity }}"
      - conditions:
          - condition: trigger
            id: sync
          - condition: template
            value_template: "{{ current_mired > 0 and max_mired > min_mired }}"
        sequence:
          - action: input_number.set_value
            target:
              entity_id: "{{ input_number_entity }}"
            data:
              value: "{{ [[sync_percent, 0] | max, 100] | min }}"

mode: single
```

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

## Light brightness and toggle

Controls a light's brightness with the dial and toggles it with a single press.

#### Blueprint

[![Import Blueprint](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https://raw.githubusercontent.com/alistairmerritt/pivot/main/assets/blueprints/pivot-light-brightness.yaml)

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
