---
layout: page
title: Custom Automations
---

Pivot is designed to work out of the box, but because it exposes standard Home Assistant entities, you can also build your own custom automations on top of it.

This makes it possible to add actions that are aware of:

- which Pivot device is being used
- which bank is currently active
- which entity is assigned to that bank
- which button press occurred, such as `single_press` or `long_press`

This can be useful when you want a bank to behave differently depending on what it is controlling. For example, you might use a Pivot bank to adjust a computer’s volume via an `input_number`, while using a button press on that same bank to trigger a shell command for media play/pause.

## How it works

In Home Assistant, `event` entities such as button press helpers are best handled by:

1. triggering the automation when the event entity changes
2. checking the `event_type` attribute to determine whether it was a single, double, or long press
3. optionally checking the currently active Pivot bank
4. optionally checking which entity is currently assigned to that bank

This allows you to build custom actions without affecting other banks or normal Pivot behaviour.

## Example: Play/Pause computer media while Bank 1 is assigned to computer volume

The example below triggers a shell command when all of the following are true:

- the button event entity reports a `single_press`
- the active Pivot bank is Bank 1
- Bank 1 is assigned to a specific helper entity used for computer volume control

In this example, the helper entity is:

`input_number.example_computer_volume`

and the shell command is:

`shell_command.music_playpause`

### Example automation

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
