---
layout: page
title: Dashboard
permalink: /dashboard/
---

This page documents a set of `custom:button-card` templates for building a Pivot control dashboard in Home Assistant.
Each component is independent, so you can use all of them, some of them, or just the bank cards on their own. The layout shown in the examples is simply a starting point.

> **Optional and experimental** — This is a community-built example dashboard, not an 'official' part of Pivot. It has not been extensively tested across all configurations and requires editing raw dashboard YAML.

> ⚠️ **Proceed with care** — Editing raw dashboard YAML carries risk. A mistake can corrupt, overwrite or completely wipe your existing dashboard. If you are not comfortable working directly in YAML, this is not recommended. **Before making any changes, copy your entire existing dashboard YAML and save it somewhere safe — even pasting it into a local text file is enough.** Please proceed at your own risk.

---

## On this page

- [Components](#components)
- [Prerequisites](#prerequisites)
- [Getting started](#getting-started)
- [Template reference](#template-reference)
- [Full example](#full-example)

---

## Components

There are four template components. The bank cards are the most important — the rest are optional additions.

### Bank cards

<img width="521" height="217" alt="bank-card" src="https://github.com/user-attachments/assets/a571cfcb-8c9c-49a9-a484-027520c20c67" />

The hero component. Each bank card shows:

- The bank number and colour dot
- The entity currently assigned to that bank
- The entity's current value (percentage, timer countdown, or on/off)
- A coloured value slider (hidden for passive/binary entities)
- A `Mirror Light Colour` toggle (visible only when a light entity is assigned)
- `Timer controls` — play/pause, reset, and preset duration buttons (15m / 30m / 45m / 60m), shown automatically when the bank entity is set to `timer`
- A `Silent Timer` toggle (shown on timer banks)

Tapping the card switches the physical device to that bank.

Below is an example of the bank card config that you would use on your dashboard. Add one card per bank per device.

```yaml
- type: custom:button-card
  template:
    - pivot_config_kitchen   # your device config template
    - pivot_banks
  variables:
    bank_number: 1           # 1, 2, 3, or 4
```

---

### Active bank indicator

<img width="236" height="68" alt="active-bank" src="https://github.com/user-attachments/assets/054a02cd-606f-4927-8b61-11c0dc24c062" />

Shows which bank is currently active on the physical device — its bank number, entity name, and colour dot. Useful as a live status line at the top of a device section.

```yaml
- type: custom:button-card
  template:
    - pivot_config_kitchen
    - pivot_active_bank
```

---

### Settings — device and behaviour toggles

<img width="567" height="440" alt="settings" src="https://github.com/user-attachments/assets/8b6ab67f-c02b-4953-a3c8-a0828096b2a0" />

Individual toggle rows for the Pivot switches. Each row shows the switch name, a short description, and a styled toggle. Pass the switch name suffix as `toggle_entity`.

Available values:

| `toggle_entity` | Switch |
| --- | --- |
| `control_mode` | Enable or disable Pivot control |
| `announcements` | Announce bank changes by voice |
| `show_control_value` | Keep value visible on LED ring when idle |
| `dim_when_idle` | Dim gauge LEDs while not interacting |

```yaml
- type: custom:button-card
  template:
    - pivot_config_kitchen
    - pivot_toggle_row
  variables:
    toggle_entity: control_mode
```

The `dim_when_idle` row automatically dims and disables itself if `show_control_value` is off, since dimming only applies when the value is shown.

You can also add a device volume slider using `custom:my-slider-v2` — useful as a central place to adjust the VPE speaker volume without going into device settings, but not absolutely necessary to include:

```yaml
- type: custom:my-slider-v2
  entity: media_player.your_vpe_media_player
  attribute: volume_level
  min: 0
  max: 100
  step: 1
```

---

### Heading card

A minimal section label with an optional subheading. This is just a styled `custom:button-card` — you can replace it with any heading approach (eg. the stock Home Assistant heading card) that suits your dashboard theme.

```yaml
- type: custom:button-card
  template: pivot_section_heading
  variables:
    heading: Kitchen Pivot
    subheading: Home Assistant Voice PE
```

---

## Prerequisites

- [HACS](https://hacs.xyz) installed in Home Assistant
- The following HACS frontend cards:
  - **custom:button-card** — required for all Pivot templates
  - **custom:my-slider-v2** — required if you want value sliders and the volume slider

`custom:layout-card` is used in the example dashboard YAML for column layout, but it is not required. The bank cards work inside any standard layout — a grid card, a vertical stack, or whatever suits your dashboard.

### Installing via HACS

1. Go to **HACS → Frontend**
2. Search for and download each card you need (see table below)
3. Once all cards are downloaded, restart Home Assistant once
4. Do a full browser refresh (Ctrl+Shift+R / Cmd+Shift+R)

| Card | Search term | Author |
| --- | --- | --- |
| custom:button-card | Button Card | RomRider |
| custom:my-slider-v2 | My Slider v2 | AnthonMS |

---

## Getting started

### 1 — Add the shared templates

Download the full `button_card_templates` YAML from GitHub:

**[button-card-templates.yaml](https://github.com/alistairmerritt/pivot/blob/main/assets/button-card-templates.yaml)**

Open your Home Assistant dashboard in **Edit** mode, click the three-dot menu, and select **Edit dashboard YAML** (or **Raw configuration editor**). **Before touching anything, select all and copy the existing YAML to a safe place.** Then paste the `button_card_templates` block at the very top of your dashboard YAML.

### 2 — Add a per-device config

The templates block includes a placeholder `pivot_config_example`. Replace it with a real entry for each Pivot device you have — or add multiple entries if you have more than one.

```yaml
button_card_templates:
  pivot_config_kitchen:
    variables:
      device_name: Kitchen Pivot          # display name
      device_suffix: ha_voice_yellow      # must match your firmware device_suffix
      media_player: media_player.home_assistant_voice_0954c7_media_player
      timer_silent_toggle: switch.kitchen_voice_pe_timer_silent_mode
```

To find entity IDs, go to **Settings → Devices & Services → ESPHome → your device**.

### 3 — Add cards to your dashboard

Reference your config template and a component template together. For example, to show all four banks for the Kitchen device:

```yaml
- type: custom:button-card
  template:
    - pivot_config_kitchen
    - pivot_banks
  variables:
    bank_number: 1
- type: custom:button-card
  template:
    - pivot_config_kitchen
    - pivot_banks
  variables:
    bank_number: 2
- type: custom:button-card
  template:
    - pivot_config_kitchen
    - pivot_banks
  variables:
    bank_number: 3
- type: custom:button-card
  template:
    - pivot_config_kitchen
    - pivot_banks
  variables:
    bank_number: 4
```

Mix in whichever other components you want alongside them.

> **Volume slider note:** The volume slider uses `custom:my-slider-v2`, which is a standalone card and cannot inherit from your `pivot_config_*` template. Set the `entity:` field directly in that card for each device.

---

## Template reference

| Template | Purpose | Key variable |
| --- | --- | --- |
| `pivot_config_*` | Per-device config. Sets `device_suffix`, `media_player`, and `timer_silent_toggle`. Pair with any other template. | `device_suffix`, `media_player`, `timer_silent_toggle` |
| `pivot_banks` | Full bank card — value display, slider, timer controls, mirror toggle. | `bank_number` (1–4) |
| `pivot_active_bank` | Active bank name, entity, and colour dot. | — |
| `pivot_section_heading` | Section label with optional subheading. | `heading`, `subheading` |
| `pivot_toggle_row` | Toggle row for a Pivot switch. | `toggle_entity` |

---

## Full example

**[dashboard-example.yaml](https://github.com/alistairmerritt/pivot/blob/main/assets/dashboard-example.yaml)**

A complete, ready-to-paste dashboard YAML that includes all the template definitions and a full five-device layout. The config variables for each device are at the top of the file — update those and the volume slider entities, paste the whole thing into your raw dashboard YAML editor, and it should produce something like this:

<!-- Replace this comment with your screenshot -->

The example uses `custom:layout-card` to achieve the column layout shown, but you can rearrange the bank cards however you like using any standard Home Assistant layout card.
