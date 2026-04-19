---
layout: page
title: Timer
permalink: /timer/
---

Pivot includes an optional timer feature — a Pomodoro session, a cooking countdown, a focus block. Timers can be controlled by voice, by the device's knob and button, or both. The timer entities are included with each configured device but disabled by default, and the blueprints are installed manually.

---

## On this page

- [How it works](#how-it-works)
- [Getting started](#getting-started)
- [Timer entities](#timer-entities)
- [Template sensors](#template-sensors)
- [Controlling the timer alarm](#controlling-the-timer-alarm)
- [Tips](#tips)

---

## How it works

The timer is built around three blueprints. One is required; the others are optional depending on how you want to control the timer.

| Blueprint | Required? | What it does |
| --- | --- | --- |
| **Pivot - Timer Control** | Yes | Manages the countdown and fires the alarm. Also handles physical button control and LED gauge when a bank is assigned. |
| **Pivot - Timer - Voice** | No | Voice control via Home Assistant Assist — set, start, pause, resume, cancel, dismiss, and query the timer by speaking to the device. |
| **Pivot - Timer Toggle Script** | No | Dashboard helper — lets a button card start, pause, resume, or dismiss the timer the same way the physical button does. |

**Pivot - Timer Control is always required**, even if you only plan to use voice control. It manages the countdown and is what sets the alarm state when time runs out — without it, a running timer simply expires silently.

Bank assignment is optional. Without a bank, the alarm fires and TTS announcements play, but the LED gauge and physical knob and button control are not available.

**Always available (with or without a bank assigned):**
- **Alarm** — plays the built-in alarm sound and pulses the LED ring when the timer finishes; single press, "stop" wake word, or dashboard button dismisses it
- **TTS announcements** — spoken feedback on start, pause, resume, and finish
- **Voice control** — via the optional Pivot - Timer - Voice blueprint
- **Dashboard control** — via the optional Pivot - Timer Toggle Script blueprint

**Available when a bank is assigned:**
- **Knob** — turn to set the duration while idle; announces the chosen time when the knob settles
- **Single press** — start if idle, pause if running, resume if paused
- **Long press** — cancel and reset (only when the timer bank is active; long press on other banks is left free for custom actions)
- **Gauge LEDs** — fill to 100% on start and drain to 0% as time runs out; device switches to the timer bank automatically when the alarm fires

---

## Getting started

### Step 1 — Enable the timer entities

The timer helper entities are provisioned with every Pivot device but disabled by default. All three must be enabled before any timer blueprint will work.

1. Go to **Settings → Devices & Services → Pivot** and open your device
2. Find the three timer entities:
   - `number.{device_suffix}_timer_duration` — duration in minutes
   - `select.{device_suffix}_timer_state` — state mirror (idle / running / paused / alerting)
   - `text.{device_suffix}_timer_end` — internal countdown tracker
3. Click each entity, go to **Settings**, and toggle **Enable**

---

### Step 2 — Set up Timer Control

The **Pivot - Timer Control** blueprint was installed automatically when you added your device. Set it up now — it is required regardless of how you plan to control the timer.

1. Go to **Settings → Automations & Scenes → Blueprints** and find **Pivot - Timer Control**
2. Click **Create Automation**
3. Fill in the inputs:

| Input | Description |
|---|---|
| **Device Suffix** | Your Pivot device suffix, e.g. `ha_voice_lounge`. All timer entity IDs are derived from this. |
| **Bank Number** | The bank reserved for the timer (1–4). Set to `0` if no bank is assigned — the alarm and TTS still work, but the LED gauge and physical button control are not available. |
| **Pivot Device** | Your Pivot VPE device. The media player and timer ringing switch are derived automatically. |
| **Button Event Entity** | The button press event entity for your device, e.g. `event.home_assistant_voice_lounge`. Used to detect long press (cancel). Find it under Settings → Devices & Services → ESPHome → your device. |
| **Finish Message** | Optional TTS message spoken once when the timer finishes, before the alarm begins. Default: `"Timer finished"`. |
| **Silent Mode** | When enabled, the alarm sound is suppressed — the LED ring still pulses and the "stop" wake word still works. Off by default. |

4. Save the automation

If a bank is assigned, the timer is now ready for physical control: single-press to start, single-press to pause, long-press to cancel. If no bank is assigned, move on to Step 4 or Step 5 to set up voice or dashboard control.

---

### Step 3 — Optional: Assign a bank

Skip this step if you only want to control the timer by voice or dashboard.

Assigning a bank enables the LED gauge countdown, knob-based duration setting, and physical button control (single press and long press).

1. Go to **Settings → Devices & Services → Pivot → your device → Configure**
2. Step through to the **Bank Entity Assignment** screen
3. Under **Timer banks**, tick the bank you want to use
4. Save

Once assigned, turn the knob to set a duration (the gauge shows the selected proportion of the maximum), then single-press to start.

> Go back to Step 2 and update Bank Number in your Timer Control automation to match the bank you assigned here.

> You can also assign the bank directly by writing `timer` (lowercase) to `text.{device_suffix}_bank_N_entity` via Developer Tools — the Configure screen and the entity stay in sync.

---

### Step 4 — Optional: Add voice control

The **Pivot - Timer - Voice** blueprint lets you control the Pivot timer by speaking to the device. Voice and physical input share the same timer state — you can start a timer by voice and cancel it from the knob, or start it from the knob and ask how much time is left.

A bank does not need to be assigned. Timer Control (Step 2) must be running.

**Setup:**

1. Go to **Settings → Automations & Scenes → Blueprints** and find **Pivot - Timer - Voice**
2. Click **Create Automation**
3. Fill in **Device Suffix** and **Pivot Device**
4. Save

> **Important:** The voice blueprint registers local Assist intents. Open your Assist pipeline settings and ensure **Prefer local intents** is enabled — this ensures spoken timer commands are matched by the blueprint before being passed to an LLM agent.

**Supported phrases include:**

| Intent | Example phrases |
|---|---|
| Set and start | *"set a 10 minute timer"*, *"start a 25 minute timer"*, *"set a timer for 45 minutes"*, *"set me a timer"*, *"start a timer"* |
| Hours | *"set a 2 hour timer"*, *"start a 1 hour timer"* |
| Combined | *"start a 1 hour and 30 minute timer"*, *"set a timer for 2 hours and 15 minutes"* |
| Shorthand | *"set a timer for a quarter of an hour"*, *"set a timer for half an hour"*, *"set a timer for an hour and a half"* |
| Pause / Resume | *"pause the timer"*, *"resume the timer"* |
| Cancel | *"cancel the timer"* |
| Dismiss alarm | *"dismiss the alarm"*, *"dismiss alarm"* |
| Status | *"how much time is left?"*, *"how long is left on the timer?"*, *"what's left on the timer?"* |

---

### Step 5 — Optional: Add dashboard control

The **Pivot - Timer Toggle Script** blueprint lets a dashboard button card start, pause, resume, and dismiss the timer — the same way the physical button does.

A bank does not need to be assigned. Timer Control (Step 2) must be running.

**Step 1 — Install the script**

The **Pivot - Timer Toggle Script** blueprint was installed automatically when you added your device. Go to **Settings → Automations & Scenes → Scripts**, click **Add Script**, and select **Pivot - Timer Toggle Script**.

> **Important:** Do not change the script name. When saving, the script ID must remain `pivot_timer_toggle` — dashboard cards call `script.pivot_timer_toggle` directly. The script only needs to be created once and works across all your Pivot devices.

**Step 2 — Add a button card**

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

To show the current timer state on the button, point it at a template sensor (see [Template sensors](#template-sensors) below):

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

**Cancel button**

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

> This resets the timer immediately without a TTS announcement. Only takes effect when the timer is running or paused.

**Complete dashboard examples**

The cards above are intentionally minimal. If you want a fully worked example — timer state display, start/pause/cancel controls, and a progress indicator in a polished device card — the [Dashboard](/pivot/dashboard/) page has complete working examples that include timer support.

---

## Timer entities

| Entity | Purpose |
| --- | --- |
| `number.{device_suffix}_timer_duration` | Duration in minutes (1–60, default 25) |
| `select.{device_suffix}_timer_state` | State mirror — updated by blueprint and firmware (idle / running / paused / alerting) |
| `text.{device_suffix}_timer_end` | Internal — stores an ISO end timestamp while running, and a `P{seconds}` remaining-time value while paused |

All three entities are **disabled by default** and live under the Pivot device in the HA device registry. `text.{device_suffix}_timer_end` is managed entirely by the blueprint; you don't need to interact with it directly.

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

## Controlling the timer alarm

Pivot exposes two switches that let you control how timer alarms behave. These apply to both Pivot timers and voice-activated timers on the device.

### Trigger the alarm manually

The `timer_ringing` switch can be turned on to trigger the timer alarm at any time — playing the built-in alarm sound and LED animation, just as if a timer had finished. Useful for testing the alarm, triggering it from a dashboard, or using it as part of a larger automation.

### Silent timers

The `silent_timer` switch suppresses the audible alarm while keeping the visual behaviour. When enabled, timers will still finish normally — the LED ring pulses and the device behaves as expected — but no sound is played.

Useful for focus sessions, late-night use, or situations where visual feedback is enough. Because this applies at the device level, it works consistently across both Pivot timers and voice-set timers.

---

## Tips

**Setting the duration** — If a bank is assigned, turn the knob while idle to select a duration. The gauge fills to reflect the selected proportion of the maximum range and announces the time when the knob settles. You can also set `number.{device_suffix}_timer_duration` directly in HA at any time.

**Gauge sync** — The gauge jumps to 100% the moment the timer starts and syncs every 30 seconds as time runs out. On a typical 25-minute timer this is less than one LED-width per update and looks continuous in practice.

**Long pressing from another bank** — Long press cancel only fires when the timer bank is the active bank. This is intentional — long press on other banks is left free for custom actions. Switch back to the timer bank first, then long press.

**Changing duration mid-session** — Turning the knob while the timer is running or paused does nothing. To change the duration, long press to cancel, then turn the knob to select a new time.

**Multiple devices** — Create one Timer Control automation per Pivot device, each with its own suffix. The Voice and Dashboard Toggle blueprints are set up once and work across all devices.

**Pausing and resuming** — Remaining time is stored in `text.{device_suffix}_timer_end` and survives HA restarts. If HA restarts while the timer is running and the stored end time has already passed, the countdown sync will detect this within 30 seconds and trigger the finish sequence normally.

**Paused auto-cancel** — If the timer is left paused for more than 15 minutes, it resets automatically and `timer_state` returns to `idle`. This prevents the timer from being stuck paused indefinitely.
