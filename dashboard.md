---
layout: page
title: Dashboard
permalink: /dashboard/
---

The Pivot integration includes a set of dashboard cards that let you view and control your Pivot devices directly from Home Assistant. You can see which bank is active, adjust values, change what each bank controls, and manage the device settings without having to find individual entities.

The cards are included with the Pivot integration – there's no additional components to install. 

> **The dashboard cards are entirely optional.** Pivot works without them, and the same settings are available through your device's entities in Home Assistant. The cards simply bring everything together in one place.

![pivot-dashboard-mockup](https://github.com/user-attachments/assets/3a08ebc1-b6bb-479e-92b0-dfcfa2c8f511)

![The Pivot dashboard showing the active bank, four banks and device settings](https://alistairmerritt.github.io/pivot/assets/images/pivot-cards.png)

---

## Adding the cards

Once you've installed the Pivot integration through HACS, you can add the cards to any Home Assistant dashboard.

1. Restart Home Assistant after installing Pivot.
2. Reload your dashboard in the browser. If you're using the Home Assistant companion app, close and reopen it.
3. Open a dashboard and select **Edit dashboard → Add card**.
4. Search for **Pivot** and choose the card you want to add.
5. Select your device from the **Pivot device** list.

Each card has its own visual editor, so you can change its layout and settings without writing YAML.

If you prefer to configure cards manually, the examples further down this page show how.

---

## The cards

There are four cards to choose from. You can use the main Pivot card to show everything for a device, or use the smaller cards to build your own layout.

### Pivot

The main card brings everything together for one Pivot device. It shows the device name, which bank is currently active, the four banks and their controls, and the device and behaviour settings.

It's a good starting point if you want a dedicated dashboard for Pivot.

```yaml
type: custom:pivot-device-card
device: ha_voice_kitchen
```

By default, the card shows all four banks, with the settings alongside them where there's enough room. On narrower screens, including phones, the layout automatically adjusts to a single column.

You can customise which banks appear, hide controls you don't need, change the timer presets or move the settings below the banks. You can also hide the settings entirely and display them using a separate card.

### Pivot bank

This card displays an individual bank, including the assigned entity, its current value, colour and available controls.

It's useful if you only want to show a particular bank as part of an existing room dashboard.

<img width="521" height="216" alt="bank-1" src="https://github.com/user-attachments/assets/112332f6-cb71-469a-99a5-f39839ca5016" />

<img width="521" height="262" alt="bank-2" src="https://github.com/user-attachments/assets/b2e77614-18a4-476a-bb38-e96f0d02406b" />

<img width="521" height="216" alt="bank-3" src="https://github.com/user-attachments/assets/a657d202-53ee-4c50-8aa0-4a3485c52078" />

```yaml
type: custom:pivot-bank-card
device: ha_voice_kitchen
bank: 1
```

You can choose a specific bank from 1 to 4, or use one of two other options:

- **Active:** Displays whichever bank is currently selected on the dial, updating automatically as you switch banks.
- **Timer:** Displays whichever bank is assigned to the timer.

The bank card includes the same controls as a bank on the main Pivot card.

### Pivot status

A smaller card that displays the device name and the bank currently selected on the dial.

For example, if Bank 2 is controlling your desk lamp, the card will show *Bank 2 – Desk Lamp*. If Pivot's Control Mode is switched off, it will show *Control mode off* instead.

<img width="236" height="68" alt="active-bank" src="https://github.com/user-attachments/assets/054a02cd-606f-4927-8b61-11c0dc24c062" />

```yaml
type: custom:pivot-status-card
device: ha_voice_kitchen
```

This is useful if you want to see what the dial is controlling without displaying all four banks.

### Pivot settings

This card displays Pivot's device and behaviour settings separately from the banks.

It includes Control Mode, device volume, announcements and LED behaviour. You can choose which settings appear, so it doesn't need to show everything.

<img width="530" height="403" alt="settings" src="https://github.com/user-attachments/assets/e01ad4cd-ed05-4db5-95a3-5708d244ff22" />

```yaml
type: custom:pivot-settings-card
device: ha_voice_kitchen
rows:
  - control_mode
  - volume
  - mute_announcements
```

For example, you might keep the banks on a room dashboard but move the settings to another view or a popup. Just disable **Show Settings** on the main Pivot card, then add a Pivot settings card wherever you want them.

---

## Using the cards

The cards work directly with your Pivot device, so changes you make on the dashboard are reflected on the dial.

### Banks

Each bank shows the entity it's controlling, its current value or state, and any controls that apply to that entity.

- **Tap a bank** to make it the active bank on the dial.
- **Drag the value bar** to adjust the entity. If the bank isn't currently active, Pivot will switch to it before changing the value.
- **Tap the entity name** to open it in Home Assistant.
- **Tap the colour dot** to change the bank's LED colour.
- **Tap EDIT ENTITY** to assign a different entity to the bank.

The controls change depending on the type of entity assigned.

For example, a light with brightness control will show a value bar, while a switch, scene, script or an open/close-only cover (such as a garage door) will simply show its current state without a bar.

There are also two additional options that appear when applicable:

- **Mirror Light Colour:** Available for lights, allowing the LEDs to match the light's colour.
- **Announce Value:** Available for entities with a value that can be announced. This is disabled while Mute Announcements is switched on.

You can press and hold any toggle row to open its corresponding entity in Home Assistant.

When Control Mode is switched off, the banks are dimmed and the value bars are disabled, matching the behaviour of the dial itself.

### Timer

If you've assigned a bank to the timer, its card includes the timer controls.

<img width="521" height="282" alt="timer" src="https://github.com/user-attachments/assets/843e9cc5-427c-4ca5-8855-9c001d1b2500" />

The value displays the selected duration before the timer starts, changes to a countdown while it's running, and shows the remaining time when paused.

You can control the timer directly from the dashboard:

- **Set a duration** using the value bar or one of the preset buttons. The default presets are 15, 30, 45 and 60 minutes, but you can choose your own.
- **Start, pause or resume** the timer using the play/pause button.
- **Cancel** the timer using the ✕ button.
- **Stop** the alarm once the timer has finished, or select **Repeat** to silence the alarm and start the same timer again.

The duration can be changed from the dashboard even when the timer bank isn't currently selected on the dial.

The card also includes a **Silent Timer** switch, which is automatically detected on the Voice Preview Edition.

> **Timer setup is required.** You'll need to have the Pivot – Timer Control blueprint configured and the relevant timer entities enabled. The dashboard uses the `pivot.timer_toggle` and `pivot.timer_cancel` actions directly, so a separate timer toggle script isn't needed.
>
> If the timer entities aren't enabled, the card will let you know rather than displaying controls that won't work. See the [Timer](https://alistairmerritt.github.io/pivot/timer/) guide for setup instructions.

### Settings

The settings cards provide the same controls available through Pivot's entities in Home Assistant.

There are a couple of behaviours worth mentioning:

- **Dim Gauge LEDs While Idle** is only available when Display Persistent Value is enabled, as there's nothing to dim if the value isn't being displayed.
- **Pivot Device Volume** controls the volume of the speaker assigned to that Pivot device.

Any settings you change from the dashboard are applied to the device, just as they would be if you changed them from Home Assistant directly.

---

## Customising the cards

All four cards include a visual editor where you can choose what appears and how it's laid out.

The main Pivot card groups its options under **Banks**, **Timer** and **Settings**. You can use these to simplify the card, show only particular banks, change the timer presets or rearrange the settings.

If you're configuring a card using YAML, the following options are available. The examples use `ha_voice_kitchen` as the device, but you'll need to replace this with your own device suffix.

Only options that differ from their defaults need to be included in the YAML.

### Pivot card options

| Option | Default | Description |
|---|---|---|
| `device` | Required | The device suffix. Select your device from the list in the card editor. |
| `show_title` | `true` | Show the device name at the top of the card. |
| `name` | Device name | Use a custom title instead of the device name. |
| `show_active_bank` | `true` | Show which bank is currently active. |
| `banks` | `all` | Choose which banks appear: `all`, a list such as `[1, 2]`, `active` or `timer`. |
| `hide_unassigned` | `false` | Hide banks that don't have an entity assigned. Applies when showing all banks or a selected list. |
| `bank_columns` | `1` | Set to `2` to display two banks per row when there's enough space (approximately 610px). Otherwise, they stack vertically. |
| `show_edit` | `true` | Show the EDIT ENTITY button on each bank. |
| `show_bank_options` | `true` | Show the Mirror Light Colour and Announce Value settings. |
| `timer_presets` | `[15, 30, 45, 60]` | Choose up to six timer presets, in minutes, between 1 and 60. Use `[]` to hide the presets. |
| `show_silent_timer` | `true` | Show the Silent Timer switch on the timer bank. |
| `timer_silent_entity` | Automatic | Manually specify the VPE's Timer Silent Mode switch if Pivot can't find it. |
| `show_settings` | `true` | Show the device and behaviour settings. |
| `settings_rows` | All | Choose which settings are displayed. |
| `settings_headings` | `true` | Show the Device and Behaviour headings. |
| `settings_background` | `false` | Place the settings on a card background, useful if your dashboard has a background image. |
| `settings_position` | `beside` | Use `beside` to place settings alongside the banks when space allows (approximately 672px), or `below` to always place them underneath. |

For `settings_rows`, you can choose any combination of:

- `control_mode`
- `volume`
- `announcements`
- `mute_announcements`
- `show_control_value`
- `dim_when_idle`

### Pivot bank card options

| Option | Default | Description |
|---|---|---|
| `device` | Required | The device suffix. |
| `bank` | `1` | Bank `1`, `2`, `3` or `4`, or `active` to follow the dial, or `timer` to show the timer bank. |
| `show_edit` | `true` | Show the EDIT ENTITY button. |
| `show_bank_options` | `true` | Show the bank's additional options. |
| `timer_presets` | `[15, 30, 45, 60]` | Choose the timer preset buttons. |
| `show_silent_timer` | `true` | Show the Silent Timer switch. |
| `timer_silent_entity` | Automatic | Specify the Silent Timer entity manually if needed. |

### Pivot status card options

| Option | Default | Description |
|---|---|---|
| `device` | Required | The device suffix. |
| `show_title` | `true` | Show the device name. |
| `name` | Device name | Use a custom title. |
| `show_active_bank` | `true` | Show which bank is currently active. |

### Pivot settings card options

| Option | Default | Description |
|---|---|---|
| `device` | Required | The device suffix. |
| `rows` | All | Choose which settings to display. |
| `headings` | `true` | Show the Device and Behaviour headings. |
| `background` | `false` | Place the settings on a card background. |

The `rows` option accepts the same settings as `settings_rows` on the main Pivot card.

---

## Theming

The cards follow your existing Home Assistant theme, including its background, text colour and fonts. They'll also automatically adjust between light and dark themes.

Most of the greys used throughout the cards are semi-transparent rather than solid colours, so they should work with translucent dashboards and background images too.

If you want more control over their appearance, Pivot includes a number of CSS variables you can override in your Home Assistant theme.

| Variable | Default (dark theme) |
|---|---|
| `--pivot-card-radius` | `20px` |
| `--pivot-value-font-family` | `'SF Pro Rounded', ui-rounded, system-ui, sans-serif` |
| `--pivot-muted-color` | `rgba(255,255,255,0.25)` |
| `--pivot-description-color` | `rgba(255,255,255,0.5)` |
| `--pivot-active-border-color` | `rgba(255,255,255,0.27)` |
| `--pivot-toggle-track-color` | `rgba(0,0,0,0.3)` |
| `--pivot-toggle-edge-color` | `rgba(255,255,255,0.08)` |
| `--pivot-toggle-on-color` | `#ffffff` |
| `--pivot-toggle-off-color` | `rgba(255,255,255,0.26)` |
| `--pivot-timer-strip-color` | `rgba(0,0,0,0.18)` |
| `--pivot-button-color` | `rgba(255,255,255,0.26)` |
| `--pivot-button-idle-color` | `rgba(255,255,255,0.08)` |
| `--pivot-volume-track-color` | `rgba(255,255,255,0.3)` |
| `--pivot-volume-fill-color` | `#ffffff` |

---

## Troubleshooting

### The Pivot cards don't appear in the card picker

Make sure the Pivot integration has been installed through HACS and Home Assistant has been restarted.

Try reloading the dashboard in your browser. If you're using the companion app, close it completely and reopen it.

If you see an error such as `Custom element doesn't exist: pivot-device-card`, Home Assistant hasn't loaded the card's frontend resources. Restarting Home Assistant and refreshing the dashboard should resolve this.

### The card says "No Pivot device"

The card can't find a Pivot device matching the one configured.

Open the card editor and choose your device from the **Pivot device** list. If you're using YAML, check that the `device` value matches your configured device suffix.

### The timer controls aren't showing

The timer entities need to be enabled and the Pivot – Timer Control blueprint configured before the dashboard can control them.

Follow the [Timer](https://alistairmerritt.github.io/pivot/timer/) guide to complete the setup.

