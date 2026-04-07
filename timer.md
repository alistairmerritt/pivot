---
layout: page
title: Timer
permalink: /timer/
---

Pivot can act as a physical timer — a Pomodoro session, a cooking countdown, a focus block — controlled from the device's knob and button, and reflected on the gauge LEDs. This is an optional feature: the timer entities are included with each device but disabled by default, and the blueprint is installed manually.

---

## On this page

- [How it works](#how-it-works)  
- [Setup](#setup)  
- [Using the timer](#using-the-timer)  
- [Dashboard control](#dashboard-control)  
- [Tips](#tips)

---

## How it works

Once set up, a single bank on your Pivot device becomes a timer controller:

- **Knob (idle)** — turn to set the duration; the gauge shows how much of the maximum time is selected and, if TTS is configured, a voice announcement confirms the chosen time (*"25 minute timer — press to start"*)
- **Single press** — start the timer if idle, pause if running, resume if paused
- **Long press** — cancel and reset (only when the timer bank is active — long press on other banks is passed through for custom actions); announces *"Timer cancelled"* if TTS is configured
- **Gauge LEDs** — fill to 100% when started, drain to 0% as time runs out
- **Finish** — the device switches back to the timer bank, plays the built-in alarm sound, pulses the LED ring, and optionally speaks a TTS message; single press, "stop" wake word, or the dashboard button dismisses the alarm.

---

## Voice timers and Pivot timers

Your voice assistant can still set timers by voice, such as “set a 25 minute timer” — that behaviour is unchanged.

However, the built-in voice timer and the Pivot timer are currently two separate systems, each with its own logic and state:

- **Starting a timer by voice** uses the VPE’s built-in timer pipeline, which runs entirely within ESPHome. It does not update any Pivot entities, so the gauge will not reflect the countdown and the blueprint will not know the timer is running.
- **Starting a timer with the knob** uses Pivot’s Home Assistant entities and blueprint. Because this timer exists within Pivot rather than the built-in voice pipeline, it cannot currently be queried by voice — for example, asking *“how long is left on the timer?”* will not return the Pivot timer’s remaining time.

At the moment, there is no bridge between the two. They operate independently and do not sync or interact with each other. The only overlap is at the end of the countdown: both use the device’s built-in `timer_ringing` mechanism, so the alarm sound and LED ring effect may appear the same when they finish.

Use whichever timer suits the moment — they can coexist without interfering with each other.

> **Enhanced timer support** — Future improvements include a Pivot-backed timer experience that can be started by voice and fully surfaced in Home Assistant. The aim is to let users say something like *“set a timer for 25 minutes”* and have it run through Pivot instead of the stock VPE timer, with visibility and control across the gauge, LED ring, and dashboard. No current timelines on this feature.

---

## Getting started with timers

### Step 1 — Enable the timer entities

The timer helpers are provisioned with every Pivot device but disabled by default. To enable them:

1. Go to **Settings → Devices & Services → Pivot** and open your device.
2. Find the three timer entities listed under the device:
   - number.{suffix}_timer_duration — duration in minutes
   - select.{suffix}_timer_state — state mirror (idle / running / paused)
   - text.{suffix}_timer_end — internal countdown tracker
3. Click each entity, go to **Settings**, and toggle **Enable**.

All three must be enabled for the blueprint to work correctly.

### Step 2 — Assign the bank to timer

The timer blueprint only activates on a bank whose entity is set to the reserved keyword `timer`. This prevents the timer from triggering on banks that have a real entity assigned.

timer is not a real Home Assistant entity, so it cannot be set through the Configure screen — the entity picker will reject it. Instead, set it directly on the text entity:

1. Go to **Devices & Services → Pivot → Your Pivot Device**
2. Locate text.{suffix}_bank_N_entity for your chosen bank (e.g. text.ha_voice_lounge_bank_2_entity)
3. Set the value to timer (lowercase, no quotes)

Once set, timer appears as the bank assignment. The knob is active for duration setting when the timer is idle, and passive while running or paused. The bank stays reserved for the timer until you change it.

### Step 3 — Set your duration

Turn the knob on the timer bank to select a duration. The gauge shows the proportion of the maximum (default max: 60 minutes) and TTS announces the selected time once you stop turning. You can also set the duration directly on number.{suffix}_timer_duration in HA — the gauge will update the next time you switch to that bank.

### Step 4 — Install the blueprint

The Pivot Timer blueprint is not installed automatically. Click the button below to import it into Home Assistant:

[![Import Blueprint](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https%3A%2F%2Fraw.githubusercontent.com%2Falistairmerritt%2Fpivot-integration%2Fmain%2Fblueprints%2Fautomation%2Fpivot%2Fpivot_timer.yaml)

Or paste this URL into **Settings → Automations → Blueprints → Import Blueprint**:

`https://raw.githubusercontent.com/alistairmerritt/pivot-integration/main/blueprints/automation/pivot/pivot_timer.yaml`

### Step 5 — Create an automation from the blueprint

1. Go to **Settings → Automations & Scenes → Blueprints** and find **Pivot — Timer**.
2. Click **Create Automation**.
3. Fill in the inputs:

| Input | Description |
|---|---|
| **Device Suffix** | Your Pivot device suffix, e.g. `ha_voice_lounge`. Timer entity IDs are derived from this. |
| **Bank Number** | Which bank controls the timer (`1–4`). This must match the bank you set to `timer`. |
| **Pivot Device** | Your Pivot VPE device. The media player and `timer_ringing` switch are derived automatically from this. |
| **Button Event Entity** | The button press event entity for your Pivot device, e.g. `event.home_assistant_voice_study`. Used to detect long press (cancel). Find it under Settings → Devices & Services → ESPHome → your device. |
| **TTS Entity** | The text-to-speech engine that *generates* spoken announcements (start, pause, resume, knob, finish). Both a TTS entity and a media player (derived from **Pivot Device**) are required for any spoken timer announcements. Leave blank to disable all speech. |
| **Finish Message** | Optional message spoken once when the timer finishes, before the alarm begins. Default: `"Timer finished"`. |
| **Silent Mode** | When enabled, the alarm sound is suppressed at finish — the LED ring still pulses and the "stop" wake word still works. Off by default. |

4. Save the automation.
   
That's it — single-press the bank to start. Single-press to pause. Long-press to reset.

---

## Timer entities

| Entity | Purpose |
| --- | --- |
| number.{suffix}_timer_duration | Duration in minutes (1–60, default 25) |
| select.{suffix}_timer_state | State mirror — updated by blueprint and firmware (idle / running / paused / alerting) |
| text.{suffix}_timer_end | Internal — stores an ISO end timestamp while running, and a `P{seconds}` remaining-time value while paused |

All three entities are **disabled by default** and live under the Pivot device in the HA device registry. The text.{suffix}_timer_end entity is managed entirely by the blueprint; you don't need to interact with it directly.

---

## Template sensors

You can expose timer state to dashboards and automations using HA template sensors. Add the following to your `configuration.yaml`, replacing `ha_voice_lounge` with your device suffix. Alternatively, create each one individually via **Settings → Helpers → Add Helper → Template**.

{% raw %}
```yaml
template:
  - sensor:

      # Remaining time — MM:SS while running, "Paused — MM:SS" while paused.
      - name: "Pivot Timer Remaining"
        icon: mdi:timer-outline
        state: >
          {% set suffix = 'ha_voice_lounge' %}
          {% set state = states('select.' ~ suffix ~ '_timer_state') %}
          {% set end_str = states('text.' ~ suffix ~ '_timer_end') %}
          {% if state == 'running' and end_str and not end_str.startswith('P') %}
            {% set secs = [(as_datetime(end_str) - now()).total_seconds(), 0] | max | int %}
            {% set h = secs // 3600 %}
            {% set m = (secs % 3600) // 60 %}
            {% set s = secs % 60 %}
            {{ '%d:%02d:%02d' | format(h, m, s) if h > 0 else '%d:%02d' | format(m, s) }}
          {% elif state == 'paused' and end_str and end_str.startswith('P') %}
            {% set secs = end_str | replace('P','') | int(0) %}
            {% set h = secs // 3600 %}
            {% set m = (secs % 3600) // 60 %}
            {% set s = secs % 60 %}
            Paused — {{ '%d:%02d:%02d' | format(h, m, s) if h > 0 else '%d:%02d' | format(m, s) }}
          {% elif state == 'alerting' %}
            Finished
          {% else %}
            —
          {% endif %}

      # Finish time — wall-clock time the timer will end, e.g. "14:35".
      - name: "Pivot Timer Ends At"
        icon: mdi:clock-end
        state: >
          {% set suffix = 'ha_voice_lounge' %}
          {% set state = states('select.' ~ suffix ~ '_timer_state') %}
          {% set end_str = states('text.' ~ suffix ~ '_timer_end') %}
          {% if state == 'running' and end_str and not end_str.startswith('P') %}
            {{ as_datetime(end_str) | as_local | strftime('%H:%M') }}
          {% else %}
            —
          {% endif %}

      # Seconds remaining — for automations and progress bars.
      - name: "Pivot Timer Seconds Remaining"
        icon: mdi:timer-sand
        unit_of_measurement: s
        state: >
          {% set suffix = 'ha_voice_lounge' %}
          {% set state = states('select.' ~ suffix ~ '_timer_state') %}
          {% set end_str = states('text.' ~ suffix ~ '_timer_end') %}
          {% if state == 'running' and end_str and not end_str.startswith('P') %}
            {{ [(as_datetime(end_str) - now()).total_seconds(), 0] | max | int }}
          {% elif state == 'paused' and end_str and end_str.startswith('P') %}
            {{ end_str | replace('P','') | int(0) }}
          {% else %}
            0
          {% endif %}
```
{% endraw %}

| Sensor | Running | Paused | Idle |
| --- | --- | --- | --- |
| **Pivot Timer Remaining** | `12:30` | `Paused — 12:30` | `—` |
| **Pivot Timer Ends At** | `14:35` | `—` | `—` |
| **Pivot Timer Seconds Remaining** | `750` | `750` | `0` |

**Pivot Timer Seconds Remaining** is the most useful for automations — trigger something when it drops below 60, or feed it into a progress bar card on a dashboard.

---

## Dashboard control

You can optionally control the timer from a Home Assistant dashboard by firing the same `pivot_button_press` event used by the physical button. The automation blueprint handles both in exactly the same way: idle starts, running pauses, paused resumes, and alerting dismisses.

Dashboard control is optional, but this script is required if you want dashboard buttons to control the timer.

### Step 1 — Install the Timer Toggle script

Import the script blueprint into Home Assistant:

[![Import Blueprint](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https%3A%2F%2Fraw.githubusercontent.com%2Falistairmerritt%2Fpivot-integration%2Fmain%2Fblueprints%2Fscript%2Fpivot_timer_toggle.yaml)

Or paste this URL into **Settings → Automations → Blueprints → Import Blueprint**:

`https://raw.githubusercontent.com/alistairmerritt/pivot-integration/main/blueprints/script/pivot_timer_toggle.yaml`

Then go to **Settings → Automations & Scenes → Scripts**, click **Add Script**, and select **Pivot — Timer Toggle Script**.

> **Important:** Do not change the script name. When saving, the script ID must remain `pivot_timer_toggle` — dashboard cards call `script.pivot_timer_toggle` directly by entity ID. The script only needs to be set up **once** — it works for all your Pivot devices, with the device suffix and bank number passed by each card at call time.

### Step 2 — Add a dashboard button card

```yaml
type: button
name: Timer
tap_action:
  action: perform-action
  perform_action: script.turn_on
  target:
    entity_id: script.pivot_timer_toggle
  data:
    variables:
      device_suffix: ha_voice_lounge
      bank: 2
show_state: false
```

Or, to show the current timer state on the button, point it at a template sensor (see the Template sensors section above):

```yaml
type: button
name: Timer
entity: sensor.pivot_timer_remaining
tap_action:
  action: perform-action
  perform_action: script.turn_on
  target:
    entity_id: script.pivot_timer_toggle
  data:
    variables:
      device_suffix: ha_voice_lounge
      bank: 2
show_state: true
```

> **Note:** The dashboard button handles all states — start, pause, resume, and alarm dismissal — identically to the physical button. Alarm dismissal requires firmware v0.0.13 or later.

### Cancel button

To add a dedicated cancel button, set `timer_state` to `idle` directly:

```yaml
type: button
name: Cancel
icon: mdi:timer-off-outline
tap_action:
  action: perform-action
  perform_action: select.select_option
  target:
    entity_id: select.ha_voice_lounge_timer_state
  data:
    option: idle
show_state: false
```

> **Note:** This resets the timer immediately. Unlike a physical long press, it does not trigger the *"Timer cancelled"* TTS announcement. Only takes effect when the timer is running or paused — pressing while idle does nothing.

---

## Tips

**Setting the duration with the knob** — Turn the knob on the timer bank while idle. The gauge fills to reflect the selected proportion of the maximum range and if configured, TTS announces the time once the knob settles. Turn clockwise for more time, anti-clockwise for less.

**Using the gauge as a visual countdown** — The gauge jumps to 100% the moment you press start and drains to 0% as time runs out, giving an at-a-glance sense of how much time remains. While the timer is running, the gauge syncs every 30 seconds — on a typical 25-minute timer this is less than one LED-width per update and looks continuous in practice.

**Assigning the bank** — Set text.{suffix}_bank_N_entity to timer directly via your device entities screen – Go to **Devices & Services → Pivot → Your Pivot Device** (the Configure screen rejects it as it's not a real entity ID). The timer blueprint only runs on banks explicitly assigned this way — if you later change the bank to a real entity, the timer stops responding to that bank automatically.

**Long pressing from another bank** — Long press cancel only fires when the timer bank is the active bank. This is intentional — long press on other banks is left free for custom actions. If you're on a different bank and want to cancel, switch back to the timer bank first, then long press.

**Changing duration mid-session** — Turning the knob while the timer is running or paused does nothing. To change the duration, switch to the timer bank, long press to cancel, then turn the knob to select a new time.

**Multiple timers** — You can create multiple automations from the same blueprint, one per Pivot device, each with its own suffix and bank assignment.

**Pausing and resuming** — Remaining time is stored in text.{suffix}_timer_end and survives HA restarts. If HA restarts while the timer is running and the stored end time has already passed, the gauge sync will detect this within 30 seconds of startup and trigger the finish sequence normally.

**Paused auto-cancel** — If the timer is left paused for more than 15 minutes, it resets automatically: the gauge goes to 0 and `timer_state` returns to `idle`. This prevents the timer from being stuck in paused state indefinitely.

