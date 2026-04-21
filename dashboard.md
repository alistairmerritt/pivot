---
layout: page
title: Dashboard
permalink: /dashboard/
---

This page documents a set of `custom:button-card` templates for building a Pivot control dashboard in Home Assistant.

Although all of these settings can already be adjusted through the device menu, it is not always the quickest or most convenient place to manage them — especially if you are regularly changing banks, colours, values or behaviour settings. As such, this dashboard is not essential to using the Pivot ecosystem, but it does provide a more central, glanceable way to view and adjust those controls from one place.

Each component is independent, so you can use all of them, some of them, or just the bank cards on their own. The layout shown in the examples is simply a starting point.

![pivot-dashboard-mockup](https://github.com/user-attachments/assets/3a08ebc1-b6bb-479e-92b0-dfcfa2c8f511)

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

There are four template components. The bank cards are the most useful — the rest are optional additions.

### Bank cards

The primary card for Pivot control.

<img width="521" height="216" alt="bank-1" src="https://github.com/user-attachments/assets/112332f6-cb71-469a-99a5-f39839ca5016" />

<img width="521" height="262" alt="bank-2" src="https://github.com/user-attachments/assets/b2e77614-18a4-476a-bb38-e96f0d02406b" />

<img width="521" height="216" alt="bank-3" src="https://github.com/user-attachments/assets/a657d202-53ee-4c50-8aa0-4a3485c52078" />

<img width="521" height="282" alt="timer" src="https://github.com/user-attachments/assets/843e9cc5-427c-4ca5-8855-9c001d1b2500" />



Each bank card shows:

- The bank number and colour dot
- The entity currently assigned to that bank
- The entity's current value (percentage, timer countdown, or on/off)
- A coloured value slider (hidden for passive/binary entities)
- A `Mirror Light Colour` toggle (visible only when a light entity is assigned)
- `Timer controls` — play/pause, reset, and preset duration buttons (15m / 30m / 45m / 60m), shown automatically when the bank entity is set to `timer`
- A `Silent Timer` toggle (shown on timer banks)
- An `Announce Value` toggle (shown for supported entity types: light, media player, fan, climate, cover, number) — enables spoken value announcements after the knob settles

Tapping the card switches the physical device to that bank.

Below is an example of the bank card config that you would use on your dashboard. For one Pivot device, you would usually add four bank cards to your dashboard. 

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

Shows which bank is currently active on the physical device — its bank number, entity name, and colour dot. Useful as a live status line at the top of a device section.

<img width="236" height="68" alt="active-bank" src="https://github.com/user-attachments/assets/054a02cd-606f-4927-8b61-11c0dc24c062" />

```yaml
- type: custom:button-card
  template:
    - pivot_config_kitchen
    - pivot_active_bank
```

---

### Settings — device and behaviour toggles

Individual toggle rows for the Pivot switches. Each row shows the switch name, a short description, and a styled toggle. Pass the switch name suffix as `toggle_entity`.

<img width="530" height="403" alt="settings" src="https://github.com/user-attachments/assets/e01ad4cd-ed05-4db5-95a3-5708d244ff22" />

Available values:

| `toggle_entity` | Switch |
| --- | --- |
| `control_mode` | Enable or disable Pivot control |
| `announcements` | Announce bank changes by voice (System Announcements) |
| `mute_announcements` | Silence all spoken announcements temporarily |
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

You can also add a device volume label and slider using `pivot_volume` — useful as a central place to adjust the VPE speaker volume without going into device settings, but not absolutely necessary to include:

```yaml
- type: custom:button-card
  template:
    - pivot_config_kitchen
    - pivot_volume
```

If you selected a media player during setup, the integration writes it to `text.{device_suffix}_media_player_entity`, and the media player entity is then derived automatically from that value.

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
      timer_silent_toggle: switch.kitchen_voice_pe_timer_silent_mode
```

To find the `timer_silent_toggle` entity ID, go to **Settings → Devices & Services → ESPHome → your device**.

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

---

## Template reference

| Template | Purpose | Key variable |
| --- | --- | --- |
| `pivot_config_*` | Per-device config. Sets `device_suffix` and `timer_silent_toggle`. Pair with any other template. | `device_suffix`, `timer_silent_toggle` |
| `pivot_banks` | Full bank card — value display, slider, timer controls, mirror toggle. | `bank_number` (1–4) |
| `pivot_active_bank` | Active bank name, entity, and colour dot. | — |
| `pivot_volume` | Device volume label and slider. Derives media player from `text.{device_suffix}_media_player_entity` automatically. | — |
| `pivot_section_heading` | Section label with optional subheading. | `heading`, `subheading` |
| `pivot_toggle_row` | Toggle row for a Pivot switch. | `toggle_entity` |

---

## Notes

**Timer bank — the bank must be active for duration changes to take effect**

The duration slider and preset buttons on a timer bank only take effect when that bank is the active bank on the device. If you adjust the duration while on a different bank, the change won't be picked up when the timer starts.

---

## Full example

**[dashboard-example.yaml](https://github.com/alistairmerritt/pivot/blob/main/assets/dashboard-example.yaml)**

A complete, ready-to-paste dashboard YAML that includes all the template definitions and a full five-device layout. The config variables for each device are at the top of the file — update those, paste the whole thing into your raw dashboard YAML editor, and it should produce something like this:

![study_pivot_half_frames](https://github.com/user-attachments/assets/7a556f57-18cb-4c95-9a22-230e97e22560)


The example uses `custom:layout-card` to achieve the layout shown, but you can rearrange the bank cards however you like using any standard Home Assistant layout card.
