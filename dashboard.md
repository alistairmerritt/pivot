---
layout: page
title: Dashboard
permalink: /dashboard/
---

Pivot comes with its own dashboard cards. They are part of the integration (v0.0.93 and later), so there is nothing extra to install: no HACS frontend cards, no YAML templates, no resources to add. Pick a card, choose your Pivot device from a list, and you're done.

![The Pivot card: active bank, the four banks, and the device and behaviour settings]({{ "/assets/images/pivot-cards.png" | relative_url }})

The dashboard is optional. Everything it shows can also be changed from the device page, but the cards put your banks, colours, values and settings in one glanceable place.

---

## On this page

- [Adding the cards](#adding-the-cards)
- [The cards](#the-cards)
- [Common setups](#common-setups)
- [What the cards do](#what-the-cards-do)
- [Theming](#theming)
- [Moving from the button-card templates](#moving-from-the-button-card-templates)
- [Troubleshooting](#troubleshooting)
- [Legacy YAML templates](#legacy-yaml-templates)

---

## Adding the cards

1. Update Pivot to **v0.0.93 or later** in HACS (**v0.0.95 or later** for the options below) and restart Home Assistant.
2. Reload the dashboard in your browser once (in the companion app, close and reopen it).
3. Open a dashboard, choose **Edit dashboard → Add card**, and search for **Pivot**.
4. Pick a card and choose your device from the **Pivot device** list.

The **Pivot** card lays out everything for one device, exactly like the screenshot above. In a sections dashboard it takes the full width and drops to one column on a phone. Its editor groups the options under **Banks**, **Timer** and **Settings**: show only some banks, follow the dial, trim the bank cards, choose your own timer presets, or move the settings. The smaller cards let you build your own layout.

---

## The cards

Every option can be set in the card editor. Only the options you change are written to the card's YAML.

### Pivot

Everything for one Pivot: the device name, the active bank, its banks, and the device and behaviour settings.

```yaml
type: custom:pivot-device-card
device: ha_voice_kitchen
```

| Option | Default | |
| --- | --- | --- |
| `device` | – | **Required.** The device's suffix, picked from a list in the editor. |
| `show_title` | `true` | Show the device name at the top. |
| `name` | the device name | Title text. |
| `show_active_bank` | `true` | Show the *● Bank 2 – Desk Lamp* line. |
| `banks` | `all` | Which banks to show: `all`, a list such as `[1, 2]`, `active` (only the bank the dial is on, changing as it does) or `timer` (only the timer bank). |
| `hide_unassigned` | `false` | Leave out banks with nothing assigned. For `all` or a list. |
| `bank_columns` | `1` | `2` puts two banks in each row once there is room (about 610 px for the banks); narrower, they stack. |
| `show_edit` | `true` | Show *EDIT ENTITY* on each bank. Turn off for wall tablets and family dashboards. |
| `show_bank_options` | `true` | Show the *Mirror Light Colour* and *Announce Value* rows. |
| `timer_presets` | `[15, 30, 45, 60]` | The timer's preset buttons, in minutes: up to 6, each from 1 to 60. `[]` leaves just play and cancel. |
| `show_silent_timer` | `true` | Show the *Silent Timer* row on the timer bank. |
| `timer_silent_entity` | found automatically | The VPE's *Timer Silent Mode* switch, if Pivot can't find it. |
| `show_settings` | `true` | Show the Device and Behaviour settings. To put them somewhere else, turn this off and add a **Pivot settings** card. |
| `settings_rows` | all of them | Which settings to show: any of `control_mode`, `volume`, `announcements`, `mute_announcements`, `show_control_value`, `dim_when_idle`. |
| `settings_headings` | `true` | Show the *Device* and *Behaviour* headings. |
| `settings_background` | `false` | Draw the settings on a card, so they stay readable over a background image. |
| `settings_position` | `beside` | `beside` puts the settings next to the banks when there's room (672 px); `below` always puts them underneath. |

Cards set up with v0.0.93 keep working: `show_status: false` still hides both the title and the active bank line.

### Pivot bank

One bank: its entity, value, colour, toggles and (for a timer bank) the timer controls.

```yaml
type: custom:pivot-bank-card
device: ha_voice_kitchen
bank: 1
```

| Option | Default | |
| --- | --- | --- |
| `device` | – | **Required.** |
| `bank` | `1` | `1`, `2`, `3` or `4`; `active` for the bank the dial is on (it changes as the dial does); `timer` for whichever bank is the timer. |
| `show_edit`, `show_bank_options`, `timer_presets`, `show_silent_timer`, `timer_silent_entity` | | As on the Pivot card. |

### Pivot status

The device name and which bank is active, for example *● Bank 2 – Desk Lamp*. It reads *Control mode off* when Pivot control is switched off.

```yaml
type: custom:pivot-status-card
device: ha_voice_kitchen
```

| Option | Default | |
| --- | --- | --- |
| `device` | – | **Required.** |
| `show_title` | `true` | Show the device name. (`show_name` from v0.0.93 still works.) |
| `name` | the device name | Title text. |
| `show_active_bank` | `true` | Show the active bank line. |

### Pivot settings

Control Mode, the device volume and the announcement and LED settings. Use it with the Pivot card's settings turned off to put them on another page, in a popup, or anywhere else.

```yaml
type: custom:pivot-settings-card
device: ha_voice_kitchen
rows:
  - control_mode
  - volume
  - mute_announcements
```

| Option | Default | |
| --- | --- | --- |
| `device` | – | **Required.** |
| `rows` | all of them | Any of `control_mode`, `volume`, `announcements`, `mute_announcements`, `show_control_value`, `dim_when_idle`. |
| `headings` | `true` | Show the *Device* and *Behaviour* headings. |
| `background` | `false` | Draw the settings on a card, for dashboards with a background image. |

---

## Common setups

**A page just for Pivot.** The defaults: title, active bank, all four banks and the settings beside them.

**In a room view.** The room's section already has a heading, and the settings belong somewhere else:

```yaml
type: custom:pivot-device-card
device: ha_voice_bedroom
show_title: false
hide_unassigned: true
show_settings: false
```

**Only the banks you use.**

```yaml
type: custom:pivot-device-card
device: ha_voice_kitchen
banks: [1, 2]
```

**A kitchen timer.** Just the timer bank, with your own presets:

```yaml
type: custom:pivot-device-card
device: ha_voice_kitchen
show_title: false
show_active_bank: false
banks: timer
show_edit: false
show_settings: false
timer_presets: [5, 10, 15, 30]
```

**What the dial is controlling now.** One bank that changes as the dial does:

```yaml
type: custom:pivot-bank-card
device: ha_voice_living_room
bank: active
```

**A wall tablet.** Nothing that reconfigures the dial, two banks per row:

```yaml
type: custom:pivot-device-card
device: ha_voice_living_room
show_edit: false
show_bank_options: false
show_settings: false
bank_columns: 2
```

**Settings on their own.** Turn `show_settings` off on the Pivot card, then add this wherever you want them:

```yaml
type: custom:pivot-settings-card
device: ha_voice_living_room
background: true
```

---

## What the cards do

**Banks**

- **Tap a bank** to make it the active bank on the dial.
- **Drag the bar** to set the value. Dragging a bank that isn't active makes it active first, then sets the value, because the dial only applies values to its active bank.
- **Tap the name** to open the entity itself.
- **Tap the colour dot** to change the bank colour.
- **Tap EDIT ENTITY** to point the bank at a different entity.
- Banks without a value (switches, scenes, scripts and open/close-only covers such as garage doors) show their state (*On*, *Off*, *Open*, *Closed*) and no bar.
- **Mirror Light Colour** appears for lights.
- **Announce Value** appears for entities with a value to announce. It is greyed out while *Mute Announcements* is on.
- Press and hold any toggle row to open its entity.
- With **Control Mode** off, the banks dim and the bars are locked, matching the dial.

**Timer bank**

- The value shows the timer duration, the countdown while running, or *Paused – 4:12*.
- The preset buttons (**15m / 30m / 45m / 60m** unless you [choose your own](#pivot)) and the bar set the duration. Unlike the old templates, this works whether or not the timer bank is the active bank.
- **▶ / ❚❚** starts, pauses and resumes; **✕** cancels.
- When the alarm goes off, **Stop** silences it and **Repeat** silences it and starts the same timer again.
- **Silent Timer** is found automatically on the Voice PE.
- If the timer entities haven't been enabled yet, the card says so instead of showing controls that do nothing. See the [Timer page](/pivot/timer/).

The cards press the timer with the `pivot.timer_toggle` and `pivot.timer_cancel` actions, so the *Pivot – Timer Toggle Script* is not needed. **Pivot – Timer Control** must still be set up, and re-importing it is recommended (see the [changelog](/pivot/changelog/)).

**Settings**

- *Dim Gauge LEDs While Idle* is greyed out while *Display Persistent Value* is off, because dimming only applies when the value is shown.
- **Pivot Device Volume** sets the volume of the speaker chosen for this Pivot.

---

## Theming

The cards follow your Home Assistant theme: the card background, text colour and font come from it. On light themes they switch to a light palette automatically. The greys are see-through white and black rather than solid colours, so the cards sit right on translucent cards and background images. To fine-tune them, set any of these variables in your theme:

| Variable | Default (dark) |
| --- | --- |
| `--pivot-card-radius` | `20px` |
| `--pivot-value-font-family` | `'SF Pro Rounded', ui-rounded, system-ui, sans-serif` |
| `--pivot-muted-color` | `rgba(255,255,255,0.25)` (values, EDIT ENTITY, row icons) |
| `--pivot-description-color` | `rgba(255,255,255,0.5)` |
| `--pivot-active-border-color` | `rgba(255,255,255,0.27)` |
| `--pivot-toggle-track-color` / `--pivot-toggle-edge-color` | `rgba(0,0,0,0.3)` / `rgba(255,255,255,0.08)` |
| `--pivot-toggle-on-color` / `--pivot-toggle-off-color` | `#ffffff` / `rgba(255,255,255,0.26)` |
| `--pivot-timer-strip-color` | `rgba(0,0,0,0.18)` |
| `--pivot-button-color` / `--pivot-button-idle-color` | `rgba(255,255,255,0.26)` / `rgba(255,255,255,0.08)` |
| `--pivot-volume-track-color` / `--pivot-volume-fill-color` | `rgba(255,255,255,0.3)` / `#ffffff` |

---

## Moving from the button-card templates

The cards replace the YAML templates one for one:

| Template | Card |
| --- | --- |
| `pivot_banks` with `bank_number: N` | `custom:pivot-bank-card` with `bank: N` |
| `pivot_active_bank` | `custom:pivot-status-card` |
| `pivot_toggle_row`, `pivot_volume`, `pivot_section_heading` | `custom:pivot-settings-card` |
| the whole page | `custom:pivot-device-card` |
| `device_suffix:` in `pivot_config_*` | `device:` |
| `timer_silent_toggle:` | found automatically (or `timer_silent_entity:`) |

Once you've switched, you can delete the `pivot_*` entries from `button_card_templates`. If nothing else on your dashboards uses them, you can also uninstall *button-card*, *my-slider-v2* and *layout-card*, and delete `script.pivot_timer_toggle`.

---

## Troubleshooting

**"Custom element doesn't exist: pivot-device-card"**
Home Assistant hasn't been restarted since Pivot was updated, or the page is still the one loaded before the restart. Restart Home Assistant, then reload the page (in the companion app, close and reopen it).

**"No Pivot device … Choose one in the card editor"**
The card's `device` doesn't match a Pivot device. Open the card editor and pick one from the list.

**The ✕ button cancels the timer but you don't hear "Timer cancelled"**
Your **Pivot – Timer Control** blueprint predates a fix for dashboard cancels, so Pivot reset the timer itself. Re-import the blueprint from the [Timer page](/pivot/timer/) to get the spoken confirmation back.

---

## Legacy YAML templates

The `custom:button-card` templates used before v0.0.93 are still available, but they will be removed in a later release. Please move to the cards above.

- [button-card-templates.yaml](https://github.com/alistairmerritt/pivot/blob/main/assets/button-card-templates.yaml): the `button_card_templates` block. Paste it at the top of your raw dashboard YAML and add a `pivot_config_*` entry per device with its `device_suffix` and `timer_silent_toggle`.
- [dashboard-example.yaml](https://github.com/alistairmerritt/pivot/blob/main/assets/dashboard-example.yaml): a complete example dashboard built from them.

They need *button-card*, *my-slider-v2* and *layout-card* from HACS. **Before editing raw dashboard YAML, copy your existing YAML somewhere safe:** a mistake can wipe the dashboard.
