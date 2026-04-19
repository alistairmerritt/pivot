---
layout: page
title: Custom Automations
permalink: /automations/
---

Pivot already handles common controls like brightness, volume and toggles out of the box, but because every press and turn also fires an event in Home Assistant, Pivot can become much more than a dial for default controls.

These examples show how Pivot can be extended into a more flexible input device — from colour control and scene scrubbing to sensor gauges, media shortcuts and context-aware button actions. Some examples simply add feedback, like changing the LED ring colour, while others completely repurpose the dial or button to do something new.

Each example includes:
- a quick explanation of the default Pivot behaviour
- what the automation changes
- an importable blueprint
- a raw YAML version you can adapt to your own setup

All blueprints include a built-in guard: if the entity configured in the blueprint is not currently assigned to the bank, the automation does nothing. That means you can reassign banks freely without needing to disable automations first.

For the full list of event fields, see the [Events](/pivot/integration/#events) section of the Integration page.

---

Some examples gently extend Pivot’s native behaviour. Others completely change what a bank does. Start with the ones that match how you already use Pivot, then branch into the more custom setups once you’re comfortable.

| Automation | What it does |
|---|---|
| [Colour control](#colour-temperature) | Dial adjusts colour temperature or hue of a light, press toggles on/off |
| [Climate control](#climate-control) | LED reflects the thermostat set temperature (e.g. a spectrum from blue to red) — turn to adjust, press to turn on or off  |
| [Button press with configurable action](#button-press-action) | Button press and knob turn adjust different entities — e.g. play/pause a computer while dial controls volume |
| [Media player volume and power toggle](#media-player-tv) | Dial controls volume, press toggles TV on/off |
| [Scene scrubbing](#scene-scrubbing) | Press cycles through up to four entities (scenes, lights, or anything) — LED colour shows the active slot |
| [Sensor gauge](#sensor-gauge) | Display any sensor on the dial, e.g. fuel level, washing machine progress, thermostat target |
| [Light brightness and toggle](#light-brightness) | Basic automation example - dial sets brightness, press toggles on/off — useful for light groups |

---

## Colour control — dial adjusts colour temperature or hue, press toggles light
{: #colour-temperature}

**Default behaviour**  
When a light is assigned directly to a bank, the knob adjusts brightness and a single press toggles the light on or off.

**Automation behaviour**  
This automation keeps the button behaviour the same, but repurposes the dial for colour control instead — either colour temperature (cool to warm) or hue (across the colour wheel).

**Why use it**  
Great for lamps and accent lighting where colour matters more than brightness, or where brightness is already set elsewhere.

Assign an `input_number` helper (range 0–100) to the bank. The automation translates the dial position into the appropriate colour value for your light, and can also keep the helper in sync if the light changes from elsewhere.

An optional **Sync LED ring colour** setting lets the ring reflect the light’s current colour. In colour temperature mode, it fades from cool blue to warm amber. In hue mode, it follows the light’s hue directly.

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
    Optionally syncs the LED ring colour to match the light in both colour temperature and hue modes.
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
      name: Sync LED ring colour
      description: >
        When enabled, the LED ring colour updates to match the light's current colour.
        In colour temperature mode, the ring fades from cool blue to warm amber.
        In hue mode, the ring matches the light's current hue.
      default: false
      selector:
        boolean:

variables:
  suffix_var: !input suffix
  bank_var: !input bank

triggers:
  - trigger: homeassistant
    event: start
    id: sync
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
      current_hue: "{{ (state_attr(light_entity, 'hs_color') or [0, 0])[0] | float(0) }}"
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
        {% set h = (state_attr(light_entity, 'hs_color') or [0, 0])[0] | float %}
        {% set ns = namespace(r=0.0, g=0.0, b=0.0) %}
        {% set x = 1.0 - ((h / 60.0) % 2.0 - 1.0) | abs %}
        {% if h < 60 %}{% set ns.r = 1.0 %}{% set ns.g = x %}
        {% elif h < 120 %}{% set ns.r = x %}{% set ns.g = 1.0 %}
        {% elif h < 180 %}{% set ns.g = 1.0 %}{% set ns.b = x %}
        {% elif h < 240 %}{% set ns.g = x %}{% set ns.b = 1.0 %}
        {% elif h < 300 %}{% set ns.r = x %}{% set ns.b = 1.0 %}
        {% else %}{% set ns.r = 1.0 %}{% set ns.b = x %}{% endif %}
        {{ '#%02x%02x%02x' | format((ns.r * 255) | round | int, (ns.g * 255) | round | int, (ns.b * 255) | round | int) }}
      ct_ring_hex: >-
        {% set rgb = state_attr(light_entity, 'rgb_color') or [255, 255, 255] %}
        {{ '#%02x%02x%02x' | format(rgb[0] | int, rgb[1] | int, rgb[2] | int) }}
  - condition: template
    value_template: "{{ states('text.' ~ suffix_var ~ '_bank_' ~ bank_var ~ '_entity') == input_number_entity }}"
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
          - if:
              - condition: template
                value_template: "{{ sync_ring_colour }}"
            then:
              - action: text.set_value
                target:
                  entity_id: "text.{{ suffix_var }}_bank_{{ bank_var }}_color"
                data:
                  value: "{{ ct_ring_hex }}"
              - action: text.set_value
                target:
                  entity_id: "text.{{ suffix_var }}_bank_{{ bank_var }}_configured_color"
                data:
                  value: "{{ ct_ring_hex }}"
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
          - if:
              - condition: template
                value_template: "{{ sync_ring_colour }}"
            then:
              - action: text.set_value
                target:
                  entity_id: "text.{{ suffix_var }}_bank_{{ bank_var }}_color"
                data:
                  value: "{{ ct_ring_hex }}"
              - action: text.set_value
                target:
                  entity_id: "text.{{ suffix_var }}_bank_{{ bank_var }}_configured_color"
                data:
                  value: "{{ ct_ring_hex }}"
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

## Climate control — dial sets temperature, press toggles on/off
{: #climate-control}

**Default behaviour**  
When a climate entity is assigned to a bank, Pivot already handles temperature adjustment with the dial and on/off control with a press.

**Automation behaviour**  
This automation does not change the control behaviour. Instead, it adds richer LED feedback by mapping the thermostat set temperature to a colour range on the ring.

**Why use it**  
It makes climate control feel more physical at a glance — cooler settings can look blue, warmer settings can fade toward orange or red.

No helper entities are required. Assign a climate entity directly to a bank as normal, then use this blueprint to colour the LED ring based on the current set temperature.

The ring interpolates smoothly across eight configurable colour stops distributed evenly between your minimum and maximum temperature.

Works with both Celsius and Fahrenheit — just set `temp_min` and `temp_max` in whichever unit Home Assistant uses.

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

    The climate entity is read automatically from the bank assignment — no need
    to specify it here.

    Works with both Celsius and Fahrenheit. Set temp_min and temp_max in whichever
    unit your Home Assistant uses (e.g. 16 and 30 for °C, or 61 and 86 for °F).
    The colour interpolation is percentage-based and fully unit-agnostic.
  domain: automation
  input:
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
      default: [77, 92, 224]
      selector:
        color_rgb: {}
    color_2:
      name: Colour 2 — (~14%)
      default: [75, 176, 208]
      selector:
        color_rgb: {}
    color_3:
      name: Colour 3 — (~29%)
      default: [77, 196, 168]
      selector:
        color_rgb: {}
    color_4:
      name: Colour 4 — (~43%)
      default: [61, 232, 112]
      selector:
        color_rgb: {}
    color_5:
      name: Colour 5 — (~57%)
      default: [244, 224, 28]
      selector:
        color_rgb: {}
    color_6:
      name: Colour 6 — (~71%)
      default: [240, 128, 48]
      selector:
        color_rgb: {}
    color_7:
      name: Colour 7 — (~86%)
      default: [216, 60, 32]
      selector:
        color_rgb: {}
    color_8:
      name: Colour 8 — hottest (100%)
      description: Colour shown at or above the maximum temperature
      default: [190, 46, 30]
      selector:
        color_rgb: {}

variables:
  suffix_var: !input suffix
  bank_var: !input bank

triggers:
  - trigger: template
    value_template: >
      {{ state_attr(states('text.' ~ suffix_var ~ '_bank_' ~ bank_var ~ '_entity'), 'temperature') | string }}
    id: climate
  - trigger: event
    event_type: pivot_knob_turn
    event_data:
      suffix: !input suffix
      bank: !input bank
    id: knob

actions:
  - variables:
      climate_entity: "{{ states('text.' ~ suffix_var ~ '_bank_' ~ bank_var ~ '_entity') }}"
  - condition: template
    value_template: "{{ climate_entity.startswith('climate.') }}"
  - variables:
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

## Button press with configurable action — e.g. dial controls volume, press to play/pause media
{: #button-press-action}

**Default behaviour**  
A bank normally uses its built-in button action for the currently assigned entity.

**Automation behaviour**  
This automation lets the button press do something completely custom instead, while the dial continues controlling the assigned bank value as normal.

**Why use it**  
Useful when the thing you want on press is related to the bank, but not the entity’s native toggle action — for example, pausing your computer while the dial controls system volume.

You can choose the press type and action, and optionally require that a specific entity is currently assigned to the bank. That makes the automation context-aware: reassign the bank and the custom action stops firing automatically.

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

**Default behaviour**  
When a media player is assigned to a bank, the knob adjusts volume and a single press sends play/pause.

**Automation behaviour**  
This automation keeps volume on the dial, but changes the press action to toggle power instead — better suited to TV-style use.

**Why use it**  
For televisions, power control is usually more useful than play/pause from the bank itself.

Assign your TV entity to the bank as normal. The automation reads the assigned media player automatically and adds power toggle behaviour on single press.

The standard play/pause action may still fire alongside the power toggle, but for most TV entities this is harmless. If you want separate play/pause control for streaming apps, it is usually better to assign the streaming device to a different bank.

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
    Assign your TV entity to the bank as normal — the media player entity is read
    automatically from the bank assignment. Single press will toggle power alongside
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

variables:
  suffix_var: !input suffix
  bank_var: !input bank

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
  - variables:
      media_player_entity: "{{ states('text.' ~ suffix_var ~ '_bank_' ~ bank_var ~ '_entity') }}"
  - condition: template
    value_template: "{{ media_player_entity.startswith('media_player.') }}"
  - choose:
      - conditions:
          - condition: trigger
            id: knob
        sequence:
          - action: media_player.volume_set
            target:
              entity_id: "{{ media_player_entity }}"
            data:
              volume_level: "{{ trigger.event.data.value / 100 }}"
      - conditions:
          - condition: trigger
            id: press
        sequence:
          - action: media_player.toggle
            target:
              entity_id: "{{ media_player_entity }}"

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

## Scene scrubbing — press to cycle through up to four entities
{: #scene-scrubbing}

**Default behaviour**  
If a light is assigned to a bank, the knob adjusts brightness and the button toggles the light.

**Automation behaviour**  
This automation turns the bank into a slot cycler. Each press jumps to the next entity slot and activates it — scenes, lights, switches, or anything HA can turn on. The LED ring updates immediately to show which slot is active. After the last slot it loops back to the first. Turning the dial previews slots without activating them.

**Why use it**  
A really tactile way to browse a small set of room moods or presets without opening a dashboard. No dial-turning required — just press to step through.

The dial’s 0–100 range is split into four equal bands, each mapped to a different entity.

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
    Cycle through up to four entities with the Pivot button. Press to jump to
    the next slot and activate it — stepping through your configured entities in
    order and looping back to the first after the last. The LED ring colour
    updates to reflect the active slot. Turning the dial also previews slots
    without activating. Assign an input_number helper (range 0–100) to the bank.
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
      name: Entity 1 (0–25%)
      selector:
        entity: {}
    scene_2:
      name: Entity 2 (25–50%)
      selector:
        entity: {}
    scene_3:
      name: Entity 3 (50–75%)
      selector:
        entity: {}
    scene_4:
      name: Entity 4 (75–100%)
      selector:
        entity: {}
    color_1:
      name: Colour — Entity 1
      default: [40, 137, 255]
      selector:
        color_rgb: {}
    color_2:
      name: Colour — Entity 2
      default: [255, 125, 25]
      selector:
        color_rgb: {}
    color_3:
      name: Colour — Entity 3
      default: [151, 255, 61]
      selector:
        color_rgb: {}
    color_4:
      name: Colour — Entity 4
      default: [200, 0, 255]
      selector:
        color_rgb: {}

variables:
  suffix_var: !input suffix
  bank_var: !input bank

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
  - condition: template
    value_template: "{{ states('text.' ~ suffix_var ~ '_bank_' ~ bank_var ~ '_entity') == input_number_entity }}"
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
          - action: input_number.set_value
            target:
              entity_id: input_number.living_room_scene
            data:
              value: >-
                {% if val < 25 %}25{% elif val < 50 %}50{% elif val < 75 %}75{% else %}0{% endif %}
          - action: homeassistant.turn_on
            target:
              entity_id: >-
                {% if val < 25 %}scene.evening
                {% elif val < 50 %}scene.bright
                {% elif val < 75 %}scene.focus
                {% else %}scene.relax{% endif %}

mode: single
```
{% endraw %}

</details>

> **How it works:** The dial previews which entity is active by updating the LED ring colour as you turn. Pressing the button jumps to the next slot — advancing the `input_number` to the start of the next band, updating the ring colour, and activating the corresponding entity. After the last slot it loops back to the first. You never need to turn the dial; the button alone cycles through all four entities.

---

## Sensor gauge — display any numeric value on the dial
{: #sensor-gauge}

**Default behaviour**  
A bank normally controls the assigned entity by writing values back to it when the dial is turned.

**Automation behaviour**  
This automation turns the bank into a read-only gauge. Pivot follows a numeric value and displays it physically, but the dial itself does not control the source.

**Why use it**  
A nice way to surface live values like fuel level, battery charge, washing machine progress or a thermostat target on a physical dial.

This is always one-way. The automation only writes to the `input_number` helper assigned to the bank — never back to the source entity. If the dial is turned, it snaps back to the real value.

The LED ring can also optionally reflect the current percentage using six configurable colour bands.

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

variables:
  suffix_var: !input suffix
  bank_var: !input bank

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
  - condition: template
    value_template: "{{ states('text.' ~ suffix_var ~ '_bank_' ~ bank_var ~ '_entity') == input_number_entity }}"
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

**Default behaviour**  
Assigning a light directly to a bank already gives you brightness on the dial and toggle on press.

**Automation behaviour**  
This reproduces that behaviour using an automation instead of relying on the built-in bank handling.

**Why use it**  
Useful as a simple example, or when you want the same light-style control pattern for something Pivot would not normally handle directly, such as a light group.

Unlike the colour control example, this one uses the raw `pivot_knob_turn` event directly rather than an `input_number` helper. The dial value is passed straight through as a brightness percentage.

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

variables:
  suffix_var: !input suffix
  bank_var: !input bank

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
  - variables:
      light_entity: !input light_entity
  - condition: template
    value_template: "{{ states('text.' ~ suffix_var ~ '_bank_' ~ bank_var ~ '_entity') == light_entity }}"
  - choose:
      - conditions:
          - condition: trigger
            id: knob
        sequence:
          - action: light.turn_on
            target:
              entity_id: "{{ light_entity }}"
            data:
              brightness_pct: "{{ trigger.event.data.value }}"
      - conditions:
          - condition: trigger
            id: press
        sequence:
          - action: light.toggle
            target:
              entity_id: "{{ light_entity }}"

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
