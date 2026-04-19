---
layout: page
title: Timer
permalink: /timer/
---

Pivot can act as a physical timer — a Pomodoro session, a cooking countdown, a focus block — controlled from the device's knob and button, and reflected on the gauge LEDs. This is an optional feature: the timer entities are included with each configured device but disabled by default, and the blueprint is installed manually.

---

## On this page

- [How it works](#how-it-works)  
- [Voice timers and Pivot timers](#voice-timers-and-pivot-timers)
- [Getting started with timers](#getting-started-with-timers)  
- [Timer entities](#timer-entities)  
- [Template sensors](#template-sensors)
- [Dashboard control](#dashboard-control)  
- [Controlling the timer alarm](#controlling-the-timer-alarm)
- [Tips](#tips)

---

## How it works

Once set up, a single bank on your Pivot device becomes a timer controller:

- **Knob (idle)** — turn to set the duration; the gauge shows how much of the maximum time is selected and, if a TTS service is configured in the integration settings, a voice announcement confirms the chosen time (*"25 minute timer — press to start"*)
- **Single press** — start the timer if idle, pause if running, resume if paused
- **Long press** — cancel and reset (only when the timer bank is active — long press on other banks is passed through for custom actions); announces *"Timer cancelled"* if a TTS service is configured in the integration settings
- **Gauge LEDs** — fill to 100% when started, drain to 0% as time runs out
- **Finish** — the device switches back to the timer bank, plays the built-in alarm sound, pulses the LED ring, and optionally speaks a TTS message; single press, "stop" wake word, or the dashboard button dismisses the alarm.
- **Alarm control** — the timer alarm can be triggered manually or set to silent via Home Assistant switches

---

## Voice and Pivot timers

Pivot timers can be controlled by voice as well as from the device itself. Pivot includes an optional blueprint that lets spoken timer commands use the same timer system as the knob, gauge, and dashboard.

You have two options:

- **Do nothing** → spoken timer commands continue to use the stock VPE timer behaviour
- **Create the automation from the included blueprint** → spoken timer commands are routed into Pivot, giving you a unified timer experience across voice, knob, gauge, and dashboard
  
Once this blueprint is set up, voice and physical input work with the same Pivot timer state. That means you can:

- start a timer by voice, then adjust or cancel it from the knob
- start a timer from the knob, then ask how much time is left
- pause, resume, cancel, dismiss, or query the timer using either voice or the device

Because everything runs through Pivot’s timer entities and automation, the timer stays visible and controllable through:

- the LED gauge
- Home Assistant entities
- dashboard controls
- spoken status queries

> Note: The voice timer automation matches common natural phrases, but it isn’t a full language model. Most everyday phrasings like “set a 5 minute timer” will work, but you may need to keep commands simple and consistent.

---

## Getting started with timers

### Step 1 — Enable the timer entities

The timer helpers are provisioned with every Pivot device but disabled by default. To enable them:

1. Go to **Settings → Devices & Services → Pivot** and open your device.
2. Find the three timer entities listed under the device:
   - number.{device_suffix}_timer_duration — duration in minutes
   - select.{device_suffix}_timer_state — state mirror (idle / running / paused)
   - text.{device_suffix}_timer_end — internal countdown tracker
3. Click each entity, go to **Settings**, and toggle **Enable**.

All three must be enabled for the blueprint to work correctly.

### Step 2 — Assign the bank to timer

The timer blueprint only activates on a bank whose entity is set to the reserved keyword `timer`. This prevents the timer from triggering on banks that have a real entity assigned.

1. Go to **Settings → Devices & Services → Pivot → your device → Configure**
2. Step through to the **Bank Entity Assignment** screen
3. Under **Timer banks**, tick the bank you want to use as a timer
4. Save

Once set, the knob is active for duration setting when the timer is idle, and passive while running or paused. The bank stays reserved for the timer until you untick it.

> You can also set this directly by writing `timer` (lowercase) to `text.{device_suffix}_bank_N_entity` via Developer Tools or any HA service call — the Configure screen and the text entity stay in sync.

### Step 3 — Set your duration

Turn the knob on the timer bank to select a duration. The gauge shows the proportion of the maximum (default max: 60 minutes) and, if a TTS service is configured in the integration settings, announces the selected time once you stop turning. You can also set the duration directly on number.{device_suffix}_timer_duration in HA — the gauge will update the next time you switch to that bank.

### Step 4 — Create an automation from the blueprint

The **Pivot — Timer** blueprint was installed automatically when you added your device.

1. Go to **Settings → Automations & Scenes → Blueprints** and find **Pivot — Timer**.
2. Click **Create Automation**.
3. Fill in the inputs:

| Input | Description |
|---|---|
| **Device Suffix** | Your Pivot device suffix, e.g. `ha_voice_lounge`. Timer entity IDs, bank assignment, and TTS settings are all derived from this. |
| **Pivot Device** | Your Pivot VPE device. The `timer_ringing` switch is derived automatically from this. |
| **Button Event Entity** | The button press event entity for your Pivot device, e.g. `event.home_assistant_voice_study`. Used to detect long press (cancel). Find it under Settings → Devices & Services → ESPHome → your device. |
| **Finish Message** | Optional message spoken once when the timer finishes, before the alarm begins. Default: `"Timer finished"`. |
| **Silent Mode** | When enabled, the alarm sound is suppressed at finish — the LED ring still pulses and the "stop" wake word still works. Off by default. |

The bank number, TTS service, and media player are all read automatically from your Pivot integration settings — no manual input needed for any of them. The blueprint detects the timer bank by looking for whichever bank is reserved as `timer` in the configure screen. If no bank is reserved yet when you create the automation, the blueprint will be inert and activate automatically once you reserve one — no changes to the blueprint needed.

4. Save the automation.
   
That's it — single-press the bank to start. Single-press to pause. Long-press to reset.

> **Optional voice timer integration:** If you want spoken timer commands such as “set a 4 minute timer” to use Pivot instead of the stock VPE timer, you can install the optional voice timer blueprint. If you skip it, spoken timers continue to use the stock timer behaviour.

---

## Timer entities

| Entity | Purpose |
| --- | --- |
| number.{device_suffix}_timer_duration | Duration in minutes (1–60, default 25) |
| select.{device_suffix}_timer_state | State mirror — updated by blueprint and firmware (idle / running / paused / alerting) |
| text.{device_suffix}_timer_end | Internal — stores an ISO end timestamp while running, and a `P{seconds}` remaining-time value while paused |

All three entities are **disabled by default** and live under the Pivot device in the HA device registry. The text.{device_suffix}_timer_end entity is managed entirely by the blueprint; you don't need to interact with it directly.

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

Dashboard control is optional, but the below script is required if you want dashboard buttons to control the timer.

### Step 1 — Install the Timer Toggle script

The **Pivot — Timer Toggle** blueprint was installed automatically when you added your device. Go to **Settings → Automations & Scenes → Scripts**, click **Add Script**, and select **Pivot — Timer Toggle**.

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

> **Note:** The dashboard button handles all states — start, pause, resume, and alarm dismissal — identically to the physical button. 

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

### Complete dashboard examples

The cards above are intentionally minimal. If you want a fully worked example — timer state display, start/pause/cancel controls, and a progress indicator all built into a polished device card — the [Dashboard](/pivot/dashboard/) page has complete, working examples built around `custom:button-card` that include timer bank support.

---

## Controlling the timer alarm

Pivot exposes two switches that let you control how timer alarms behave. These apply to both Pivot timers and voice-activated timers on the device.

### Trigger the alarm manually

The `timer_ringing` switch can be turned on to trigger the timer alarm at any time. This plays the built-in alarm sound and LED animation, just as if a timer had finished.

This is useful for:
- testing the alarm
- triggering it from a dashboard
- using the alarm as part of a larger automation

### Silent timers

The `silent_timer` switch suppresses the audible alarm while keeping the visual behaviour. When enabled, timers will still “finish” normally — the LED ring pulses and the device behaves as expected — but no sound is played.

This is useful for:
- focus timers or work sessions
- late-night use
- situations where visual feedback is enough

Because this applies at the device level, it works consistently across both Pivot timers and voice-set timers.

---

## Tips

**Setting the duration with the knob** — Turn the knob on the timer bank while idle. The gauge fills to reflect the selected proportion of the maximum range and, if a TTS service is configured in the integration settings, announces the time once the knob settles. Turn clockwise for more time, anti-clockwise for less.

**Using the gauge as a visual countdown** — The gauge jumps to 100% the moment you press start and drains to 0% as time runs out, giving an at-a-glance sense of how much time remains. While the timer is running, the gauge syncs every 30 seconds — on a typical 25-minute timer this is less than one LED-width per update and looks continuous in practice.

**Assigning the bank** — Tick the bank under **Timer banks** in the Configure screen, or write `timer` (lowercase) directly to `text.{device_suffix}_bank_N_entity` via Developer Tools. The timer blueprint only runs on banks assigned this way — if you later change the bank to a real entity, the timer stops responding to that bank automatically.

**Long pressing from another bank** — Long press cancel only fires when the timer bank is the active bank. This is intentional — long press on other banks is left free for custom actions. If you're on a different bank and want to cancel, switch back to the timer bank first, then long press.

**Changing duration mid-session** — Turning the knob while the timer is running or paused does nothing. To change the duration, switch to the timer bank, long press to cancel, then turn the knob to select a new time.

**Multiple timers** — You can create multiple automations from the same blueprint, one per Pivot device, each with its own suffix and bank assignment.

**Pausing and resuming** — Remaining time is stored in text.{device_suffix}_timer_end and survives HA restarts. If HA restarts while the timer is running and the stored end time has already passed, the gauge sync will detect this within 30 seconds of startup and trigger the finish sequence normally.

**Paused auto-cancel** — If the timer is left paused for more than 15 minutes, it resets automatically: the gauge goes to 0 and `timer_state` returns to `idle`. This prevents the timer from being stuck in paused state indefinitely.

