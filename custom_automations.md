---
layout: page
title: Custom Automations
---

In Manual mode, Pivot creates all the required entities and fires events on the HA event bus, but does not create any scripts or automations itself — you build everything. This gives you complete control over what each button press and knob turn does, without any behaviour you did not explicitly set up.

If you use Automatic mode, custom automations are an optional layer you can add on top. Because Pivot fires events on every interaction regardless of mode, you can trigger your own automations in response to things the built-in behaviour does not handle — like using a button press on a specific bank to trigger a shell command or run a scene.

Pivot events carry enough context to let automations be as specific or as broad as you need:

- which Pivot device was used
- which bank was active
- which entity is assigned to that bank
- which button press type occurred (`single_press`, `double_press`, `long_press`, etc.)
- the new value and direction of a knob turn

---

## Example: Controlling a light in Manual mode

In Manual mode you need to handle both the knob and the button yourself. The example below sets a light's brightness from the knob value and toggles it with a single press, for a device with suffix `ha_voice_lounge` and Bank 1 assigned to `light.living_room`.

### Knob — set brightness

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

### Button — toggle on/off

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

> **Available event fields** — see the [Integration reference](/pivot/integration/#events) for the full list of fields available on `pivot_knob_turn` and `pivot_button_press` events.

---

## Example: Play/Pause computer media while Bank 1 is assigned to computer volume

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
