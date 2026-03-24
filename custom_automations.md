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

This can be useful when you want a bank to behave differently depending on what it is controlling. For example, you might use a Pivot bank to adjust a Mac’s volume via an `input_number`, while using a button press on that same bank to trigger a shell command for media play/pause.

## How it works

In Home Assistant, `event` entities such as button press helpers are best handled by:

1. triggering the automation when the event entity changes
2. checking the `event_type` attribute to determine whether it was a single, double, or long press
3. optionally checking the currently active Pivot bank
4. optionally checking which entity is currently assigned to that bank

This allows you to build custom actions without affecting other banks or normal Pivot behaviour.

## Example: Play/Pause mac media while Bank 1 is assigned to Mac volume

The example below triggers a shell command when all of the following are true:

- the button event entity reports a `single_press`
- the active Pivot bank is Bank 1
- Bank 1 is assigned to a specific helper entity used for Mac volume control

In this example, the helper entity is:

`input_number.example_mac_volume`

and the shell command is:

`shell_command.music_playpause`

### Example automation

```yaml
alias: Pivot Example - MacBook Play Pause

triggers:
  # Trigger whenever the button press event entity updates. 
  # Note: This entity_id should be the button press one from your original Voice Preview Edition device in the ESPHome integration, not an entity created by the Pivot integration.
  - trigger: state
    entity_id: event.example_vpe_button_press
    not_from:
      - unavailable
    not_to:
      - unavailable
      - unknown

conditions:
  # Only continue if the latest button press was a single press.
  - condition: state
    entity_id: event.example_pivot_button_press
    attribute: event_type
    state: single_press

  # Only continue if the currently active bank is Bank 1.
  # Pivot bank numbering starts at 1, so:
  # Bank 1 = above 0 and below 2
  - condition: numeric_state
    entity_id: number.example_pivot_active_bank
    above: 0
    below: 2

  # Only continue if Bank 1 is currently assigned to the expected entity.
  # This checks the text helper that stores the assigned entity ID for Bank 1.
  - condition: template
    value_template: {{ states('text.example_pivot_ba nk_0_entity') == 'input_number.example_mac_volume' }}

actions:
  # Run the shell command that handles media play/pause.
  - action: shell_command.music_playpause

mode: single
```
