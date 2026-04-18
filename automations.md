---
layout: page
title: Custom Automations
permalink: /automations/
---

Pivot already handles common controls like brightness, volume, and toggles out of the box. But because it fires events on every interaction, it can also be used for much more custom behaviour — from colour control and scene scrubbing to sensor gauges, media shortcuts, and context-aware button actions.

The examples below show how to use Pivot as a flexible input device for Home Assistant. Each one includes an importable blueprint and a raw YAML version, so you can either use them as-is or adapt them to your own setup.

Pivot events carry enough context to make automations as specific or as broad as you need, including:

- which Pivot device was used
- which bank was active
- which entity is assigned to that bank
- which button press type occurred (`single_press`, `long_press`, etc.)
- the new value and direction of a knob turn

For the full list of event fields, see the [Events](/pivot/integration/#events) section of the Integration page.

---

| Automation | What it does |
|---|---|
| [Colour control](#colour-temperature) | Dial adjusts colour temperature or hue, press toggles on/off |
| [Button press with configurable action](#button-press-action) | Press triggers any action — e.g. play/pause a computer while dial controls volume |
| [Media player volume and power toggle](#media-player-tv) | Dial controls volume, press toggles TV on/off |
| [Scene scrubbing](#scene-scrubbing) | Dial previews scenes via LED colour, press activates |
| [Sensor gauge](#sensor-gauge) | Display any sensor on the dial, e.g. fuel level, washing machine progress, thermostat target |
| [Light brightness and toggle](#light-brightness) | Dial sets brightness, press toggles on/off — useful for light groups |
| [Climate control](#climate-control) | Dial sets temperature, press toggles on/off — LED ring colour follows temperature |

---

## Colour control — dial adjusts colour temperature or hue, press toggles light
{: #colour-temperature}

Assigning a light directly to a bank gives you brightness control. This automation lets you use the dial for colour instead — either colour temperature (cool to warm) or hue (sweeping through the colour wheel at full saturation). Choose the mode when setting up the blueprint.

Assign an `input_number` helper (range 0–100) to the bank. The automation translates the dial position into the appropriate colour value for your light, and keeps the helper in sync if the light is changed from elsewhere.

In hue mode, an optional **Sync LED ring colour** setting will update the LED ring to match the light's current hue — both when the dial is turned and when the light changes externally.

Create a helper: **Settings → Devices & Services → Helpers → Number**, with a range of 0–100 and step 1. Then assign it to the bank on your device.

#### Blueprint

[![Import Blueprint](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https://raw.githubusercontent.com/alistairmerritt/pivot/main/assets/blueprints/pivot-colour-temperature.yaml)

<details markdown="1">
<summary>Blueprint YAML</summary>

{% raw %}
```yaml
blueprint:
  name: Pivot - Colour Control
  description: >
    Control a light's colour temperature or hue with the Pivot dial via an input_number helper.
    Single press toggles the light. Dial position stays in sync with the light.
    In hue mode, optionally syncs the LED ring colour to match the light.
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
    control_mode:
      name: Control mode
      description: >
        Colour temperature adjusts warmth (cool to warm).
        Hue sweeps through the colour wheel at full saturation.
      default: colour_temperature
      selector:
        select:
          options:
            - label: Colour temperature
              value: colour_temperature
            - label: Hue
              value: hue
    sync_ring_colour:
      name: Sync LED ring colour (hue mode only)
      description: >
        When enabled, the LED ring colour updates to match the light's current hue.
        Has no effect in colour temperature mode.
      default: false
      selector:
        boolean:

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
      control_mode: !input control_mode
      sync_ring_colour: !input sync_ring_colour
      suffix_var: !input suffix
      bank_var: !input bank
      percent: "{{ states(input_number_entity) | float(0) }}"
      min_mired: "{{ state_attr(light_entity, 'min_mireds') | float(153) }}"
      max_mired: "{{ state_attr(light_entity, 'max_mireds') | float(500) }}"
      target_kelvin: >-
        {{ (1000000 / ((min_mired | int) + ((percent / 100) * ((max_mired | int)
        - (min_mired | int))))) | round(0) | int }}
      current_mired: "{{ state_attr(light_entity, 'color_temp') | float(0) }}"
      sync_percent_ct: >-
        {{ (((current_mired - min_mired) / (max_mired - min_mired)) * 100) | round(0) }}
      target_hue: "{{ (percent / 100 * 360) | int }}"
      current_hue: "{{ (state_attr(light_entity, 'hs_color') | default([0, 0]))[0] | float(0) }}"
      sync_percent_hue: "{{ (current_hue / 360 * 100) | round(0) }}"
      target_hex: >-
        {% set h = (percent / 100 * 360) | float %}
        {% set ns = namespace(r=0.0, g=0.0, b=0.0) %}
        {% set x = 1.0 - ((h / 60.0) % 2.0 - 1.0) | abs %}
        {% if h < 60 %}{% set ns.r = 1.0 %}{% set ns.g = x %}
        {% elif h < 120 %}{% set ns.r = x %}{% set ns.g = 1.0 %}
        {% elif h < 180 %}{% set ns.g = 1.0 %}{% set ns.b = x %}
        {% elif h < 240 %}{% set ns.g = x %}{% set ns.b = 1.0 %}
        {% elif h < 300 %}{% set ns.r = x %}{% set ns.b = 1.0 %}
        {% else %}{% set ns.r = 1.0 %}{% set ns.b = x %}{% endif %}
        {{ '#%02x%02x%02x' | format((ns.r * 255) | round | int, (ns.g * 255) | round | int, (ns.b * 255) | round | int) }}
      current_hex: >-
        {% set h = (state_attr(light_entity, 'hs_color') | default([0, 0]))[0] | float %}
        {% set ns = namespace(r=0.0, g=0.0, b=0.0) %}
        {% set x = 1.0 - ((h / 60.0) % 2.0 - 1.0) | abs %}
        {% if h < 60 %}{% set ns.r = 1.0 %}{% set ns.g = x %}
        {% elif h < 120 %}{% set ns.r = x %}{% set ns.g = 1.0 %}
        {% elif h < 180 %}{% set ns.g = 1.0 %}{% set ns.b = x %}
        {% elif h < 240 %}{% set ns.g = x %}{% set ns.b = 1.0 %}
        {% elif h < 300 %}{% set ns.r = x %}{% set ns.b = 1.0 %}
        {% else %}{% set ns.r = 1.0 %}{% set ns.b = x %}{% endif %}
        {{ '#%02x%02x%02x' | format((ns.r * 255) | round | int, (ns.g * 255) | round | int, (ns.b * 255) | round | int) }}
  - choose:
      - conditions:
          - condition: trigger
            id: dial
          - condition: template
            value_template: "{{ control_mode == 'colour_temperature' }}"
          - condition: template
            value_template: "{{ (trigger.to_state.state | float | round(0) | int) != (sync_percent_ct | round(0) | int) }}"
        sequence:
          - action: light.turn_on
            target:
              entity_id: !input light_entity
            data:
              color_temp_kelvin: "{{ target_kelvin | int }}"
      - conditions:
          - condition: trigger
            id: dial
          - condition: template
            value_template: "{{ control_mode == 'hue' }}"
          - condition: template
            value_template: "{{ (trigger.to_state.state | float | round(0) | int) != (sync_percent_hue | round(0) | int) }}"
        sequence:
          - action: light.turn_on
            target:
              entity_id: !input light_entity
            data:
              hs_color:
                - "{{ target_hue }}"
                - 100
          - if:
              - condition: template
                value_template: "{{ sync_ring_colour }}"
            then:
              - action: text.set_value
                target:
                  entity_id: "text.{{ suffix_var }}_bank_{{ bank_var }}_color"
                data:
                  value: "{{ target_hex }}"
              - action: text.set_value
                target:
                  entity_id: "text.{{ suffix_var }}_bank_{{ bank_var }}_configured_color"
                data:
                  value: "{{ target_hex }}"
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
            value_template: "{{ control_mode == 'colour_temperature' and current_mired > 0 and max_mired > min_mired }}"
        sequence:
          - action: input_number.set_value
            target:
              entity_id: !input input_number_entity
            data:
              value: "{{ [[sync_percent_ct, 0] | max, 100] | min }}"
      - conditions:
          - condition: trigger
            id: sync
          - condition: template
            value_template: "{{ control_mode == 'hue' and state_attr(light_entity, 'hs_color') is not none }}"
        sequence:
          - action: input_number.set_value
            target:
              entity_id: !input input_number_entity
            data:
              value: "{{ [[sync_percent_hue, 0] | max, 100] | min }}"
          - if:
              - condition: template
                value_template: "{{ sync_ring_colour }}"
            then:
              - action: text.set_value
                target:
                  entity_id: "text.{{ suffix_var }}_bank_{{ bank_var }}_color"
                data:
                  value: "{{ current_hex }}"
              - action: text.set_value
                target:
                  entity_id: "text.{{ suffix_var }}_bank_{{ bank_var }}_configured_color"
                data:
                  value: "{{ current_hex }}"

mode: single
```
{% endraw %}

</details>

#### Raw automation examples

<details markdown="1">
<summary>Colour temperature</summary>

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
      sync_percent_ct: >-
        {{ (((current_mired - min_mired) / (max_mired - min_mired)) * 100) | round(0) }}
  - choose:
      - conditions:
          - condition: trigger
            id: dial
          - condition: template
            value_template: "{{ (trigger.to_state.state | float | round(0) | int) != (sync_percent_ct | round(0) | int) }}"
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
              value: "{{ [[sync_percent_ct, 0] | max, 100] | min }}"

mode: single
```
{% endraw %}

</details>

<details markdown="1">
<summary>Hue</summary>

{% raw %}
```yaml
alias: Pivot - Desk Light Hue

triggers:
  - trigger: state
    entity_id: input_number.desk_light_hue
    id: dial
  - trigger: event
    event_type: pivot_button_press
    event_data:
      suffix: ha_voice_lounge
      bank: 3
      press_type: single_press
    id: press
  - trigger: state
    entity_id: light.desk_light
    id: sync

actions:
  - variables:
      percent: "{{ states('input_number.desk_light_hue') | float(0) }}"
      target_hue: "{{ (percent / 100 * 360) | int }}"
      current_hue: "{{ (state_attr('light.desk_light', 'hs_color') | default([0, 0]))[0] | float(0) }}"
      sync_percent_hue: "{{ (current_hue / 360 * 100) | round(0) }}"
      target_hex: >-
        {% set h = (percent / 100 * 360) | float %}
        {% set ns = namespace(r=0.0, g=0.0, b=0.0) %}
        {% set x = 1.0 - ((h / 60.0) % 2.0 - 1.0) | abs %}
        {% if h < 60 %}{% set ns.r = 1.0 %}{% set ns.g = x %}
        {% elif h < 120 %}{% set ns.r = x %}{% set ns.g = 1.0 %}
        {% elif h < 180 %}{% set ns.g = 1.0 %}{% set ns.b = x %}
        {% elif h < 240 %}{% set ns.g = x %}{% set ns.b = 1.0 %}
        {% elif h < 300 %}{% set ns.r = x %}{% set ns.b = 1.0 %}
        {% else %}{% set ns.r = 1.0 %}{% set ns.b = x %}{% endif %}
        {{ '#%02x%02x%02x' | format((ns.r * 255) | round | int, (ns.g * 255) | round | int, (ns.b * 255) | round | int) }}
      current_hex: >-
        {% set h = current_hue %}
        {% set ns = namespace(r=0.0, g=0.0, b=0.0) %}
        {% set x = 1.0 - ((h / 60.0) % 2.0 - 1.0) | abs %}
        {% if h < 60 %}{% set ns.r = 1.0 %}{% set ns.g = x %}
        {% elif h < 120 %}{% set ns.r = x %}{% set ns.g = 1.0 %}
        {% elif h < 180 %}{% set ns.g = 1.0 %}{% set ns.b = x %}
        {% elif h < 240 %}{% set ns.g = x %}{% set ns.b = 1.0 %}
        {% elif h < 300 %}{% set ns.r = x %}{% set ns.b = 1.0 %}
        {% else %}{% set ns.r = 1.0 %}{% set ns.b = x %}{% endif %}
        {{ '#%02x%02x%02x' | format((ns.r * 255) | round | int, (ns.g * 255) | round | int, (ns.b * 255) | round | int) }}
  - choose:
      - conditions:
          - condition: trigger
            id: dial
          - condition: template
            value_template: "{{ (trigger.to_state.state | float | round(0) | int) != (sync_percent_hue | round(0) | int) }}"
        sequence:
          - action: light.turn_on
            target:
              entity_id: light.desk_light
            data:
              hs_color:
                - "{{ target_hue }}"
                - 100
          - action: text.set_value
            target:
              entity_id: text.ha_voice_lounge_bank_3_color
            data:
              value: "{{ target_hex }}"
          - action: text.set_value
            target:
              entity_id: text.ha_voice_lounge_bank_3_configured_color
            data:
              value: "{{ target_hex }}"
      - conditions:
          - condition: trigger
            id: press
        sequence:
          - action: light.toggle
            target:
              entity_id: light.desk_light
      - conditions:
          - condition: trigger
            id: sync
          - condition: template
            value_template: "{{ state_attr('light.desk_light', 'hs_color') is not none }}"
        sequence:
          - action: input_number.set_value
            target:
              entity_id: input_number.desk_light_hue
            data:
              value: "{{ [[sync_percent_hue, 0] | max, 100] | min }}"
          - action: text.set_value
            target:
              entity_id: text.ha_voice_lounge_bank_3_color
            data:
              value: "{{ current_hex }}"
          - action: text.set_value
            target:
              entity_id: text.ha_voice_lounge_bank_3_configured_color
            data:
              value: "{{ current_hex }}"

mode: single
```
{% endraw %}

</details>

---

## Button press with configurable action — e.g. dial controls volume, press to play/pause media
{: #button-press-action}

The built-in bank toggle handles turning entities on and off, but sometimes a button press should do something extra — like triggering play/pause on your computer when the bank is controlling system volume, or running a scene when a specific entity is active.

This automation fires any action you choose on button press, with an optional check that a particular entity is currently assigned to the bank. That check is what makes it context-aware: reassign the bank to something else and the automation automatically stops firing.

#### Blueprint

[![Import Blueprint](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https://raw.githubusercontent.com/alistairmerritt/pivot/main/assets/blueprints/pivot-button-press-action.yaml)

<details markdown="1">
<summary>Blueprint YAML</summary>

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

</details>

#### Raw automation example

<details markdown="1">
<summary>Raw automation example</summary>

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

</details>

---

## Media player volume and power toggle — dial controls volume, press toggles TV on/off
{: #media-player-tv}

When a media player is assigned to a bank, Pivot normally controls volume with the dial and play/pause with a single press. For a TV, however, power control is often more useful than play/pause. This automation adds that behaviour, letting you assign your TV as normal while using a single press to toggle power as well.

The standard play/pause action still fires, but in most cases this causes no issue. If the TV is turning off, it is irrelevant. If it is turning on, the command is usually ignored.

If you want separate play/pause control for streaming apps, it is best to assign the relevant streaming device to a different bank.

#### Blueprint

[![Import Blueprint](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https://raw.githubusercontent.com/alistairmerritt/pivot/main/assets/blueprints/pivot-media-player-tv.yaml)

<details markdown="1">
<summary>Blueprint YAML</summary>

{% raw %}
```yaml
blueprint:
  name: Pivot - Media Player Volume & Power Toggle
  description: >
    Control a TV's volume with the Pivot dial and toggle power on single press.
    Assign your TV entity to the bank as normal. Single press will toggle power alongside
    the native play/pause — in practice the play/pause is harmless for a TV entity.
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

</details>

#### Raw automation example

<details markdown="1">
<summary>Raw automation example</summary>

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

</details>

---

## Scene scrubbing — dial previews scenes, press activates
{: #scene-scrubbing}

A useful pattern for rooms where you have a handful of lighting presets. Divide the dial's 0–100 range into four equal bands, each mapped to a different scene. As you turn the dial the LED ring changes colour to preview which scene you're about to select — turn to the one you want, then press to activate it.

For example: 0–25% → Relax (warm amber), 25–50% → Evening (blue), 50–75% → Bright (green), 75–100% → Focus (purple).

Create a helper: **Settings → Devices & Services → Helpers → Number**, with a range of 0–100 and step 1. Assign it to the bank on your device.

#### Blueprint

[![Import Blueprint](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https://raw.githubusercontent.com/alistairmerritt/pivot/main/assets/blueprints/pivot-scene-scrubbing.yaml)

<details markdown="1">
<summary>Blueprint YAML</summary>

{% raw %}
```yaml
blueprint:
  name: Pivot - Scene Scrubbing
  description: >
    Scrub through up to four scenes with the Pivot dial. The LED ring colour
    updates as you turn to preview which scene is selected. Press to activate it.
    Assign an input_number helper (range 0–100) to the bank.
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
    scene_1:
      name: Scene 1 (0–25%)
      selector:
        entity:
          domain: scene
    scene_2:
      name: Scene 2 (25–50%)
      selector:
        entity:
          domain: scene
    scene_3:
      name: Scene 3 (50–75%)
      selector:
        entity:
          domain: scene
    scene_4:
      name: Scene 4 (75–100%)
      selector:
        entity:
          domain: scene
    color_1:
      name: Colour — Scene 1
      default: [255, 147, 41]
      selector:
        color_rgb: {}
    color_2:
      name: Colour — Scene 2
      default: [10, 132, 255]
      selector:
        color_rgb: {}
    color_3:
      name: Colour — Scene 3
      default: [48, 209, 88]
      selector:
        color_rgb: {}
    color_4:
      name: Colour — Scene 4
      default: [191, 90, 242]
      selector:
        color_rgb: {}

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

actions:
  - variables:
      input_number_entity: !input input_number_entity
      suffix_var: !input suffix
      bank_var: !input bank
      scene_1: !input scene_1
      scene_2: !input scene_2
      scene_3: !input scene_3
      scene_4: !input scene_4
      color_1: !input color_1
      color_2: !input color_2
      color_3: !input color_3
      color_4: !input color_4
      val: "{{ states(input_number_entity) | float(0) }}"
      selected_scene: >-
        {% if val < 25 %}{{ scene_1 }}
        {% elif val < 50 %}{{ scene_2 }}
        {% elif val < 75 %}{{ scene_3 }}
        {% else %}{{ scene_4 }}{% endif %}
      ring_color: >-
        {% if val < 25 %}{% set c = color_1 %}
        {% elif val < 50 %}{% set c = color_2 %}
        {% elif val < 75 %}{% set c = color_3 %}
        {% else %}{% set c = color_4 %}{% endif %}
        {{ '#%02x%02x%02x' | format(c[0] | int, c[1] | int, c[2] | int) }}
  - choose:
      - conditions:
          - condition: trigger
            id: dial
        sequence:
          - action: text.set_value
            target:
              entity_id: "text.{{ suffix_var }}_bank_{{ bank_var }}_color"
            data:
              value: "{{ ring_color }}"
          - action: text.set_value
            target:
              entity_id: "text.{{ suffix_var }}_bank_{{ bank_var }}_configured_color"
            data:
              value: "{{ ring_color }}"
      - conditions:
          - condition: trigger
            id: press
        sequence:
          - action: scene.turn_on
            target:
              entity_id: "{{ selected_scene }}"

mode: single
```
{% endraw %}

</details>

#### Raw automation example

<details markdown="1">
<summary>Raw automation example</summary>

{% raw %}
```yaml
alias: Pivot - Living Room Scenes

triggers:
  - trigger: state
    entity_id: input_number.living_room_scene
    id: dial
  - trigger: event
    event_type: pivot_button_press
    event_data:
      suffix: ha_voice_lounge
      bank: 1
      press_type: single_press
    id: press

actions:
  - variables:
      val: "{{ states('input_number.living_room_scene') | float(0) }}"
      selected_scene: >-
        {% if val < 25 %}scene.relax
        {% elif val < 50 %}scene.evening
        {% elif val < 75 %}scene.bright
        {% else %}scene.focus{% endif %}
      ring_color: >-
        {% if val < 25 %}#ff9329
        {% elif val < 50 %}#0a84ff
        {% elif val < 75 %}#30d158
        {% else %}#bf5af2{% endif %}
  - choose:
      - conditions:
          - condition: trigger
            id: dial
        sequence:
          - action: text.set_value
            target:
              entity_id: text.ha_voice_lounge_bank_1_color
            data:
              value: "{{ ring_color }}"
          - action: text.set_value
            target:
              entity_id: text.ha_voice_lounge_bank_1_configured_color
            data:
              value: "{{ ring_color }}"
      - conditions:
          - condition: trigger
            id: press
        sequence:
          - action: scene.turn_on
            target:
              entity_id: "{{ selected_scene }}"

mode: single
```
{% endraw %}

</details>

> **How it works:** The dial updates the `input_number` helper, which triggers this automation. Instead of activating the scene immediately, it writes the preview colour to the LED ring so you can see which scene you're hovering over. When you press, the button event fires and the selected scene activates. The scene is computed at press time from the current dial position, so whatever band you stopped in is what gets activated.

---

## Sensor gauge — display any numeric value on the dial
{: #sensor-gauge}

A display-only automation that maps any numeric value onto a Pivot bank. Works with sensor, input_number, and number entities — useful for things like a fuel level, water tank, battery, thermostat target, or any numeric value where you want a physical gauge without any control.

This is always one-way. The automation only ever writes to the `input_number` helper assigned to the bank — it never writes back to the source entity. For sensors this is obvious, but it applies equally to `input_number` and `number` entities: even if their value changes elsewhere in HA, Pivot just follows it. The dial is fully locked — if it is turned, the automation immediately snaps it back by listening directly to the `pivot_knob_turn` event.

The LED ring colour can optionally update based on the current percentage, using six configurable colour bands. Defaults run red → orange → amber → yellow → yellow-green → green.

Assign an `input_number` helper (range 0–100) to the bank. The automation scales the source value between a configurable minimum and maximum.

Create a helper: **Settings → Devices & Services → Helpers → Number**, with a range of 0–100 and step 1. Assign it to the bank on your device.

#### Blueprint

[![Import Blueprint](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https://raw.githubusercontent.com/alistairmerritt/pivot/main/assets/blueprints/pivot-sensor-gauge.yaml)

<details markdown="1">
<summary>Blueprint YAML</summary>

{% raw %}
```yaml
blueprint:
  name: Pivot - Sensor Gauge
  description: >
    Map any numeric sensor, input_number, or number entity to a Pivot bank for
    display only. Always one-way — the source entity is never written to. The dial
    position reflects the value scaled between a configurable min and max. If the
    dial is accidentally turned, it automatically reverts to the source value.
    Optionally colours the LED ring based on the current percentage.
  domain: automation
  input:
    sensor_entity:
      name: Source entity
      description: The sensor, input_number, or number entity to monitor (e.g. a fuel level, battery, tank sensor, or any numeric helper)
      selector:
        entity:
          domain:
            - sensor
            - input_number
            - number
    input_number_entity:
      name: Input number helper
      description: The input_number entity assigned to this bank (range 0–100)
      selector:
        entity:
          domain: input_number
    sensor_min:
      name: Sensor minimum
      description: The sensor value that corresponds to 0% on the dial
      default: 0
      selector:
        number:
          min: -9999
          max: 9999
          step: 0.1
          mode: box
    sensor_max:
      name: Sensor maximum
      description: The sensor value that corresponds to 100% on the dial
      default: 100
      selector:
        number:
          min: -9999
          max: 9999
          step: 0.1
          mode: box
    suffix:
      name: Device suffix
      description: The suffix of your Pivot device (e.g. ha_voice_lounge)
      selector:
        text: {}
    bank:
      name: Bank number
      description: The bank number this input_number is assigned to (1–4)
      selector:
        number:
          min: 1
          max: 4
          mode: box
    sync_ring_colour:
      name: Sync LED ring colour
      description: >
        When enabled, the LED ring colour updates based on the current percentage
        using the colour bands below.
      default: true
      selector:
        boolean:
    color_0_16:
      name: Colour — 0 to 16%
      default: [255, 69, 58]
      selector:
        color_rgb: {}
    color_16_33:
      name: Colour — 16 to 33%
      default: [255, 122, 0]
      selector:
        color_rgb: {}
    color_33_50:
      name: Colour — 33 to 50%
      default: [255, 159, 10]
      selector:
        color_rgb: {}
    color_50_66:
      name: Colour — 50 to 66%
      default: [255, 214, 10]
      selector:
        color_rgb: {}
    color_66_83:
      name: Colour — 66 to 83%
      default: [169, 220, 56]
      selector:
        color_rgb: {}
    color_83_100:
      name: Colour — 83 to 100%
      default: [48, 209, 88]
      selector:
        color_rgb: {}

triggers:
  - trigger: state
    entity_id: !input sensor_entity
    id: sensor
  - trigger: state
    entity_id: !input input_number_entity
    id: number_changed
  - trigger: event
    event_type: pivot_knob_turn
    event_data:
      suffix: !input suffix
      bank: !input bank
    id: knob

actions:
  - variables:
      input_number_entity: !input input_number_entity
      sensor_entity: !input sensor_entity
      sensor_min: !input sensor_min
      sensor_max: !input sensor_max
      suffix_var: !input suffix
      bank_var: !input bank
      sync_ring_colour: !input sync_ring_colour
      color_0_16: !input color_0_16
      color_16_33: !input color_16_33
      color_33_50: !input color_33_50
      color_50_66: !input color_50_66
      color_66_83: !input color_66_83
      color_83_100: !input color_83_100
      sensor_value: "{{ states(sensor_entity) | float(0) }}"
      computed_percent: >-
        {{ [[(sensor_value - sensor_min) / (sensor_max - sensor_min) * 100, 0]
        | max, 100] | min | round(0) | int }}
      current_number: "{{ states(input_number_entity) | float(-1) }}"
      ring_color: >-
        {% if computed_percent < 16 %}{% set c = color_0_16 %}
        {% elif computed_percent < 33 %}{% set c = color_16_33 %}
        {% elif computed_percent < 50 %}{% set c = color_33_50 %}
        {% elif computed_percent < 66 %}{% set c = color_50_66 %}
        {% elif computed_percent < 83 %}{% set c = color_66_83 %}
        {% else %}{% set c = color_83_100 %}{% endif %}
        {{ '#%02x%02x%02x' | format(c[0] | int, c[1] | int, c[2] | int) }}
  - choose:
      - conditions:
          - condition: trigger
            id: knob
        sequence:
          - action: input_number.set_value
            target:
              entity_id: !input input_number_entity
            data:
              value: "{{ computed_percent }}"
    default:
      - if:
          - condition: template
            value_template: "{{ computed_percent != current_number and states(sensor_entity) not in ('unknown', 'unavailable') }}"
        then:
          - action: input_number.set_value
            target:
              entity_id: !input input_number_entity
            data:
              value: "{{ computed_percent }}"
          - if:
              - condition: template
                value_template: "{{ sync_ring_colour }}"
            then:
              - action: text.set_value
                target:
                  entity_id: "text.{{ suffix_var }}_bank_{{ bank_var }}_color"
                data:
                  value: "{{ ring_color }}"
              - action: text.set_value
                target:
                  entity_id: "text.{{ suffix_var }}_bank_{{ bank_var }}_configured_color"
                data:
                  value: "{{ ring_color }}"

mode: single
```
{% endraw %}

</details>

#### Raw automation example

<details markdown="1">
<summary>Raw automation example</summary>

{% raw %}
```yaml
alias: Pivot - Fuel Level

triggers:
  - trigger: state
    entity_id: sensor.car_fuel_level
    id: sensor
  - trigger: state
    entity_id: input_number.fuel_gauge
    id: number_changed

actions:
  - variables:
      sensor_value: "{{ states('sensor.car_fuel_level') | float(0) }}"
      computed_percent: "{{ [[(sensor_value - 0) / (60 - 0) * 100, 0] | max, 100] | min | round(0) | int }}"
      current_number: "{{ states('input_number.fuel_gauge') | float(-1) }}"
      ring_color: >-
        {% if computed_percent < 16 %}{% set c = [255, 69, 58] %}
        {% elif computed_percent < 33 %}{% set c = [255, 122, 0] %}
        {% elif computed_percent < 50 %}{% set c = [255, 159, 10] %}
        {% elif computed_percent < 66 %}{% set c = [255, 214, 10] %}
        {% elif computed_percent < 83 %}{% set c = [169, 220, 56] %}
        {% else %}{% set c = [48, 209, 88] %}{% endif %}
        {{ '#%02x%02x%02x' | format(c[0], c[1], c[2]) }}
  - if:
      - condition: template
        value_template: "{{ computed_percent != current_number and states('sensor.car_fuel_level') not in ('unknown', 'unavailable') }}"
    then:
      - action: input_number.set_value
        target:
          entity_id: input_number.fuel_gauge
        data:
          value: "{{ computed_percent }}"
      - action: text.set_value
        target:
          entity_id: text.ha_voice_lounge_bank_2_color
        data:
          value: "{{ ring_color }}"
      - action: text.set_value
        target:
          entity_id: text.ha_voice_lounge_bank_2_configured_color
        data:
          value: "{{ ring_color }}"

mode: single
```
{% endraw %}

</details>

---

## Light brightness and toggle — dial sets brightness, press toggles on/off
{: #light-brightness}

The simplest custom automation pattern — the dial controls a light's brightness directly and a single press toggles it. Useful if you want more precise brightness control than Pivot's built-in toggle provides, or if you're using a bank for a light that isn't directly assignable (e.g. a light group).

Unlike the colour temperature example, this one uses the raw `pivot_knob_turn` event directly rather than going via an `input_number` helper — the dial value is passed straight through as a brightness percentage.

#### Blueprint

[![Import Blueprint](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https://raw.githubusercontent.com/alistairmerritt/pivot/main/assets/blueprints/pivot-light-brightness.yaml)

<details markdown="1">
<summary>Blueprint YAML</summary>

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

</details>

#### Raw automation example

<details markdown="1">
<summary>Raw automation examples</summary>

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

</details>

## Climate control — dial sets temperature, press toggles on/off
{: #climate-control}

Assign a climate entity directly to a Pivot bank. The integration handles everything natively — turn the knob to change the set temperature, press once in Control Mode to toggle the climate on and off. This blueprint manages only the LED ring colour, which smoothly interpolates across eight configurable colour stops distributed evenly between your minimum and maximum temperature.

No helper entities required. Works with both Celsius and Fahrenheit — just set `temp_min` and `temp_max` in whichever unit your Home Assistant uses (e.g. 16 and 30 for °C, or 61 and 86 for °F). The colour interpolation is percentage-based and unit-agnostic.

#### Blueprint

[![Import Blueprint](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https://raw.githubusercontent.com/alistairmerritt/pivot/main/assets/blueprints/pivot-climate-control.yaml)

<details markdown="1">
<summary>Blueprint YAML</summary>

{% raw %}
```yaml
blueprint:
  name: Pivot - Climate Control
  description: >
    Colours the LED ring of a Pivot bank based on the set temperature of a
    climate entity. Assign the climate entity directly to the bank — the Pivot
    integration handles temperature control and on/off toggle natively. This
    blueprint only manages the ring colour, smoothly interpolating across eight
    colour stops that are evenly distributed between your configured minimum and
    maximum temperatures.

    Works with both Celsius and Fahrenheit. Set temp_min and temp_max in whichever
    unit your Home Assistant uses (e.g. 16 and 30 for °C, or 61 and 86 for °F).
    The colour interpolation is percentage-based and fully unit-agnostic.
  domain: automation
  input:
    climate_entity:
      name: Climate entity
      description: The climate entity assigned to this bank
      selector:
        entity:
          domain: climate
    suffix:
      name: Device suffix
      description: The suffix of your Pivot device (e.g. ha_voice_lounge)
      selector:
        text: {}
    bank:
      name: Bank number
      description: The bank number this climate entity is assigned to (1–4)
      selector:
        number:
          min: 1
          max: 4
          mode: box
    temp_min:
      name: Temperature minimum
      description: >
        The temperature mapped to 0% on the dial. Use your HA unit — °C or °F
        (e.g. 16 for Celsius, 61 for Fahrenheit).
      default: 16
      selector:
        number:
          min: -40
          max: 120
          step: 0.5
          mode: box
    temp_max:
      name: Temperature maximum
      description: >
        The temperature mapped to 100% on the dial. Use your HA unit — °C or °F
        (e.g. 30 for Celsius, 86 for Fahrenheit).
      default: 30
      selector:
        number:
          min: -40
          max: 120
          step: 0.5
          mode: box
    color_1:
      name: Colour 1 — coldest (0%)
      description: Colour shown at or below the minimum temperature
      default: [101, 210, 255]
      selector:
        color_rgb: {}
    color_2:
      name: Colour 2 — (~14%)
      default: [49, 209, 88]
      selector:
        color_rgb: {}
    color_3:
      name: Colour 3 — (~29%)
      default: [186, 206, 29]
      selector:
        color_rgb: {}
    color_4:
      name: Colour 4 — (~43%)
      default: [255, 185, 0]
      selector:
        color_rgb: {}
    color_5:
      name: Colour 5 — (~57%)
      default: [255, 148, 0]
      selector:
        color_rgb: {}
    color_6:
      name: Colour 6 — (~71%)
      default: [255, 76, 1]
      selector:
        color_rgb: {}
    color_7:
      name: Colour 7 — (~86%)
      default: [234, 37, 11]
      selector:
        color_rgb: {}
    color_8:
      name: Colour 8 — hottest (100%)
      description: Colour shown at or above the maximum temperature
      default: [191, 30, 30]
      selector:
        color_rgb: {}

triggers:
  - trigger: state
    entity_id: !input climate_entity
    attribute: temperature
    id: climate
  - trigger: event
    event_type: pivot_knob_turn
    event_data:
      suffix: !input suffix
      bank: !input bank
    id: knob

actions:
  - variables:
      climate_entity: !input climate_entity
      suffix_var: !input suffix
      bank_var: !input bank
      temp_min: !input temp_min
      temp_max: !input temp_max
      color_1: !input color_1
      color_2: !input color_2
      color_3: !input color_3
      color_4: !input color_4
      color_5: !input color_5
      color_6: !input color_6
      color_7: !input color_7
      color_8: !input color_8
      set_temp: >-
        {% if trigger.id == 'knob' %}
          {{ temp_min + (trigger.event.data.value | float(0) / 100) * (temp_max - temp_min) }}
        {% else %}
          {{ state_attr(climate_entity, 'temperature') | float(temp_min) }}
        {% endif %}
      ring_color: >-
        {% set pct = [[(set_temp | float - temp_min) / (temp_max - temp_min) * 100, 0] | max, 100] | min %}
        {% set sw = 100 / 7 %}
        {% set seg = [[((pct / sw) | int), 0] | max, 6] | min %}
        {% set frac = (pct - seg * sw) / sw %}
        {% set ns = namespace(c0=[0, 0, 0], c1=[0, 0, 0]) %}
        {% if seg == 0 %}{% set ns.c0 = color_1 %}{% set ns.c1 = color_2 %}
        {% elif seg == 1 %}{% set ns.c0 = color_2 %}{% set ns.c1 = color_3 %}
        {% elif seg == 2 %}{% set ns.c0 = color_3 %}{% set ns.c1 = color_4 %}
        {% elif seg == 3 %}{% set ns.c0 = color_4 %}{% set ns.c1 = color_5 %}
        {% elif seg == 4 %}{% set ns.c0 = color_5 %}{% set ns.c1 = color_6 %}
        {% elif seg == 5 %}{% set ns.c0 = color_6 %}{% set ns.c1 = color_7 %}
        {% else %}{% set ns.c0 = color_7 %}{% set ns.c1 = color_8 %}{% endif %}
        {% set r = (ns.c0[0] + frac * (ns.c1[0] - ns.c0[0])) | int %}
        {% set g = (ns.c0[1] + frac * (ns.c1[1] - ns.c0[1])) | int %}
        {% set b = (ns.c0[2] + frac * (ns.c1[2] - ns.c0[2])) | int %}
        {{ '#%02x%02x%02x' | format(r, g, b) }}
  - action: text.set_value
    target:
      entity_id: "text.{{ suffix_var }}_bank_{{ bank_var }}_color"
    data:
      value: "{{ ring_color }}"
  - action: text.set_value
    target:
      entity_id: "text.{{ suffix_var }}_bank_{{ bank_var }}_configured_color"
    data:
      value: "{{ ring_color }}"

mode: queued
max: 5
```
{% endraw %}

</details>
