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
- [What the cards do](#what-the-cards-do)
- [Theming](#theming)
- [Moving from the button-card templates](#moving-from-the-button-card-templates)
- [Troubleshooting](#troubleshooting)
- [Legacy YAML templates](#legacy-yaml-templates)

---

## Adding the cards

1. Update Pivot to **v0.0.93 or later** in HACS and restart Home Assistant.
2. Reload the dashboard in your browser once (in the companion app, close and reopen it).
3. Open a dashboard, choose **Edit dashboard → Add card**, and search for **Pivot**.
4. Pick a card and choose your device from the **Pivot device** list.

The **Pivot** card lays out everything for one device, exactly like the screenshot above. In a sections dashboard it takes the full width and drops to one column on a phone. The smaller cards let you build your own layout.

---

## The cards

### Pivot

The whole dashboard for one Pivot: the device name, the active bank, the four banks, and the device and behaviour settings.

```yaml
type: custom:pivot-device-card
device: ha_voice_kitchen
```

| Option | Default | |
| --- | --- | --- |
| `device` | – | **Required.** The device's suffix, picked from a list in the editor. |
| `name` | the device name | Title shown at the top. |
| `show_status` | `true` | Show the title and the active bank line. |
| `show_settings` | `true` | Show the Device and Behaviour settings. |
| `timer_silent_entity` | found automatically | The VPE's *Timer Silent Mode* switch, if Pivot can't find it. |

### Pivot bank

One bank: its entity, value, colour, toggles and (for a timer bank) the timer controls. Use four of them for one device.

```yaml
type: custom:pivot-bank-card
device: ha_voice_kitchen
bank: 1
```

| Option | Default | |
| --- | --- | --- |
| `device` | – | **Required.** |
| `bank` | `1` | `1`, `2`, `3` or `4`. |
| `timer_silent_entity` | found automatically | As above. |

### Pivot status

The device name and which bank is active, for example *● Bank 2 – Desk Lamp*. It reads *Control mode off* when Pivot control is switched off.

```yaml
type: custom:pivot-status-card
device: ha_voice_kitchen
```

| Option | Default | |
| --- | --- | --- |
| `device` | – | **Required.** |
| `name` | the device name | Title text. |
| `show_name` | `true` | Set to `false` to show only the active bank line. |

### Pivot settings

Control Mode, the device volume and the announcement and LED settings.

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
- **15m / 30m / 45m / 60m** and the bar set the duration. Unlike the old templates, this works whether or not the timer bank is the active bank.
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

The cards follow your Home Assistant theme: the card background, text colour and font come from it. On light themes they switch to a light palette automatically. To fine-tune them, set any of these variables in your theme:

| Variable | Default (dark) |
| --- | --- |
| `--pivot-card-radius` | `20px` |
| `--pivot-value-font-family` | `'SF Pro Rounded', ui-rounded, system-ui, sans-serif` |
| `--pivot-muted-color` | `#545454` (values, EDIT ENTITY, row icons) |
| `--pivot-description-color` | `rgba(255,255,255,0.5)` |
| `--pivot-active-border-color` | `rgba(180,180,180,0.4)` |
| `--pivot-toggle-track-color` | `#171717` |
| `--pivot-toggle-on-color` / `--pivot-toggle-off-color` | `#ffffff` / `#545454` |
| `--pivot-timer-strip-color` | `#171717` |
| `--pivot-button-color` / `--pivot-button-idle-color` | `#545454` / `#2a2a2a` |
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
