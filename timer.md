---
layout: page
title: Timer
permalink: /timer/
---

Pivot includes an optional timer feature: a Pomodoro session, a cooking countdown, or a focus block. Timers can be controlled by voice, by the device's knob and button, or both.

## On this page

- [How it works](#how-it-works)
- [Getting started](#getting-started)
- [Timer entities](#timer-entities)
- [Template sensors](#template-sensors)
- [Controlling the timer alarm](#controlling-the-timer-alarm)
- [Tips](#tips)
- [Why not use the built-in HA timer?](#why-not-use-the-built-in-ha-timer)

---

## How it works

The timer is built around three blueprints. One is required; the others are optional depending on how you want to control the timer.

| Blueprint | Required? | What it does | Import |
| --- | --- | --- | --- |
| **Pivot - Timer Control** | Yes | Manages the countdown and fires the alarm. Also handles physical button control and the LED gauge when a bank is assigned. | [Import Blueprint](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https%3A%2F%2Fraw.githubusercontent.com%2Falistairmerritt%2Fpivot-integration%2Fmain%2Fblueprints%2Fautomation%2Fpivot_timer.yaml) |
| **Pivot - Timer - Voice** | Optional | Voice control via Home Assistant Assist. Set, start, pause, resume, cancel, dismiss, and query the timer by speaking to the device. | [Import Blueprint](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https%3A%2F%2Fraw.githubusercontent.com%2Falistairmerritt%2Fpivot-integration%2Fmain%2Fblueprints%2Fautomation%2Fpivot_timer_voice.yaml) |
| **Pivot - Timer Toggle Script** | Optional | Dashboard helper. Lets a button card start, pause, resume, or dismiss the timer the same way the physical button does. | [Import Blueprint](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https%3A%2F%2Fraw.githubusercontent.com%2Falistairmerritt%2Fpivot-integration%2Fmain%2Fblueprints%2Fscript%2Fpivot_timer_toggle.yaml) |

**Pivot - Timer Control is always required**, even if you only plan to use voice control. It manages the countdown and is what sets the alarm state when time runs out. Without it, a running timer simply expires silently.

Bank assignment is optional. Without a bank, the alarm still fires and TTS announcements still play, but the LED gauge and physical knob and button control are not available.

**Always available (with or without a bank assigned):**
- **Alarm** — plays the built-in alarm sound and pulses the LED ring when the timer finishes; single press, "stop" wake word, or the dashboard button dismisses it
- **TTS announcements** — spoken feedback on start, pause, resume, and finish
- **Voice control** — via the optional **Pivot - Timer - Voice** blueprint
- **Dashboard control** — via the optional **Pivot - Timer Toggle Script** blueprint

**Available when a bank is assigned:**
- **Knob** — turn to set the duration while idle; announces the chosen time when the knob settles. Knob durations are in whole minutes (minimum 1 minute) — for sub-minute timers, use voice control
- **Single press** — start if idle, pause if running, resume if paused
- **Long press** — cancel and reset (only when the timer bank is active; long press on other banks is left free for custom actions)
- **Gauge LEDs** — fill to 100% on start and drain to 0% as time runs out; the device switches to the timer bank automatically when the alarm fires

---

## Getting started

### Step 1. Enable the timer entities

The timer helper entities are provisioned with every Pivot device but disabled by default. All three must be enabled before any timer blueprint will work.

1. Go to **Settings → Devices & Services → Pivot** and open your device
2. Find the three timer entities:
   - `number.{device_suffix}_timer_duration` — duration in minutes
   - `select.{device_suffix}_timer_state` — state mirror (`idle` / `running` / `paused` / `alerting`)
   - `text.{device_suffix}_timer_end` — internal countdown tracker
3. Click each entity, go to **Settings**, and toggle **Enable**

---

### Step 2. Set up Timer Control – [Import Blueprint (Open in a new tab)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https%3A%2F%2Fraw.githubusercontent.com%2Falistairmerritt%2Fpivot-integration%2Fmain%2Fblueprints%2Fautomation%2Fpivot_timer.yaml)

The **Pivot - Timer Control** blueprint is required regardless of how you plan to control the timer. Import it first using the link above if you haven't already.

1. Go to **Settings → Automations & Scenes → Blueprints** and find **Pivot - Timer Control**
2. Click **Create Automation**
3. Fill in the inputs:

| Input | Description |
|---|---|
| **Device Suffix** | Your Pivot device suffix, e.g. `ha_voice_lounge`. All timer entity IDs are derived from this. |
| **Bank Number** | The bank reserved for the timer (`1–4`). Set to `0` if no bank is assigned — the alarm and TTS still work, but the LED gauge and physical button control are not available. |
| **Button Event Entity** | The button press event entity for your device, e.g. `event.home_assistant_voice_lounge`. Used to detect long press (cancel). Find it under **Settings → Devices & Services → ESPHome → your device**. |
| **Timer Ringing Switch** | The `timer_ringing` switch for your device. Find it under **Settings → Devices & Services → ESPHome → your device**. |
| **Finish Message** | Optional. TTS message spoken once when the timer finishes, before the alarm begins. Default: `"Timer finished"`. |

4. Save the automation

If a bank is assigned, the timer is now ready for physical control: single press to start, single press to pause, and long press to cancel. If no bank is assigned, move on to Step 4 or Step 5 to set up voice or dashboard control.

---

### Step 3. Optional: assign a bank

Skip this step if you only want to control the timer by voice or dashboard.

Assigning a bank enables the LED gauge countdown, knob-based duration setting, and physical button control (single press and long press).

1. Go to **Settings → Devices & Services → Pivot → your device → Configure**
2. Step through to the **Bank Entity Assignment** screen
3. Under **Timer banks**, tick the bank you want to use
4. Save

Once assigned, turn the knob to set a duration (the gauge shows the selected proportion of the maximum), then single press to start.

> Go back to Step 2 and update **Bank Number** in your **Pivot - Timer Control** automation to match the bank you assigned here.

> You can also assign the bank directly by writing `timer` (lowercase) to `text.{device_suffix}_bank_N_entity` via Developer Tools — the Configure screen and the entity stay in sync.

---

### Step 4. Optional: add voice control – [Import Blueprint (Open in a new tab)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https%3A%2F%2Fraw.githubusercontent.com%2Falistairmerritt%2Fpivot-integration%2Fmain%2Fblueprints%2Fautomation%2Fpivot_timer_voice.yaml)

By default, spoken timer commands on the VPE use the stock Home Assistant timer behaviour — they are handled by the Assist pipeline and are completely independent of Pivot. **Installing this blueprint is the only thing that changes that.** If you skip this step, voice timers continue to work exactly as they did before.

You have two options:

- **Do nothing** — spoken timer commands continue to use the stock VPE timer. The Pivot timer (knob, gauge, dashboard) and the stock voice timer remain separate.
- **Install Pivot - Timer - Voice** — spoken timer commands are routed into the Pivot timer instead, giving you a single unified timer across voice, knob, gauge, and dashboard.

If you install the blueprint and later want to go back to stock behaviour, simply disable or delete the automation — the stock timer is restored immediately with no other changes needed.

When installed, voice and physical input share the same timer state — you can start a timer by voice and cancel it from the knob, or start it from the knob and ask how much time is left.

A bank does not need to be assigned. **Pivot - Timer Control** (Step 2) must already be running.

**Setup:**

1. Import **Pivot - Timer - Voice** using the link above if you haven't already
2. Go to **Settings → Automations & Scenes → Blueprints** and find **Pivot - Timer - Voice**
3. Click **Create Automation**
4. Fill in **Device Suffix** and **Timer Ringing Switch**
5. Save

> **Important:** The voice blueprint registers local Assist intents. Open your Assist pipeline settings and ensure **Prefer local intents** is enabled — this ensures spoken timer commands are matched by the blueprint before being passed to an LLM agent.

**Supported phrases:**

| Intent | Phrases |
|---|---|
| Start (minutes) | *"set a {n} minute timer"*, *"start a {n} minute timer"*, *"set timer for {n} minutes"* |
| Start (hours) | *"set a {n} hour timer"*, *"start a {n} hour timer"*, *"set timer for {n} hours"* |
| Start (seconds) | *"set a {n} second timer"*, *"start a {n} second timer"*, *"set a timer for {n} seconds"*, *"start a timer for {n} seconds"* |
| Start (combined) | *"start a {h} hour and {m} minute timer"*, *"set a {h} hour and {m} minute timer"*, *"set timer for {h} hours and {m} minutes"* |
| Start (shorthand) | *"set a quarter hour timer"*, *"start a quarter hour timer"*, *"set a timer for a quarter of an hour"*, *"start a timer for a quarter of an hour"*, *"set a half hour timer"*, *"start a half hour timer"*, *"set a timer for half an hour"*, *"start a timer for half an hour"*, *"set a timer for an hour and a half"*, *"start a timer for an hour and a half"*, *"set an hour and a half timer"*, *"set a timer for two and a half hours"*, *"start a timer for two and a half hours"* |
| Start (current duration) | *"start timer"*, *"set timer"*, *"start a timer"*, *"set a timer"*, *"set me a timer"* |
| Pause | *"pause timer"*, *"pause the timer"* |
| Resume | *"resume timer"*, *"resume the timer"*, *"continue timer"*, *"continue the timer"*, *"unpause timer"*, *"unpause the timer"* |
| Cancel | *"cancel timer"*, *"cancel the timer"*, *"stop timer"*, *"stop the timer"*, *"reset timer"*, *"reset the timer"* |
| Dismiss alarm | *"dismiss timer"*, *"dismiss the timer"*, *"dismiss alarm"*, *"stop the alarm"* |
| Status | *"how long is left on the timer"*, *"how much time is left"*, *"how much time is left on the timer"*, *"how much time remains"*, *"how much longer is left"*, *"what's the timer at"*, *"what's left on the timer"*, *"timer status"* |

Number values (`{n}`, `{h}`, `{m}`) accept both digits (*"5"*) and words (*"five"*).

> **Seconds timers:** Second-based durations are supported by voice only. The knob and dashboard controls are constrained to whole minutes (minimum 1 minute).

---

### Step 5. Optional: add dashboard control

The [**Pivot - Timer Toggle Script**](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https%3A%2F%2Fraw.githubusercontent.com%2Falistairmerritt%2Fpivot-integration%2Fmain%2Fblueprints%2Fscript%2Fpivot_timer_toggle.yaml) blueprint lets a dashboard button card start, pause, resume, and dismiss the timer the same way the physical button does.

A bank does not need to be assigned. **Pivot - Timer Control** (Step 2) must already be running.

#### Step 1. Install the script

Import **Pivot - Timer Toggle Script** using the link above if you haven't already. Then go to **Settings → Automations & Scenes → Scripts**, click **Add Script**, and select **Pivot - Timer Toggle Script**.

> **Important:** Do not change the script name. When saving, the script ID must remain `pivot_timer_toggle` — dashboard cards call `script.pivot_timer_toggle` directly. The script only needs to be created once and works across all your Pivot devices.

#### Step 2. Add a tile card

This example uses the `sensor.pivot_timer_remaining` template sensor to display the current timer state — see [Template sensors](#template-sensors) below for how to create it.

```yaml
type: tile
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
```

#### Cancel button

To add a dedicated cancel button, set `timer_state` to `idle` directly:

```yaml
type: tile
name: Cancel Timer
icon: mdi:timer-off-outline
entity: select.ha_voice_lounge_timer_state
tap_action:
  action: perform-action
  perform_action: select.select_option
  target:
    entity_id: select.ha_voice_lounge_timer_state
  data:
    option: idle
```

> This resets the timer immediately without a TTS announcement. It only takes effect when the timer is running or paused.

#### Complete dashboard examples

The cards above are intentionally minimal. If you want a fully worked example — timer state display, start/pause/cancel controls, and a progress indicator in a polished device card — the [Dashboard](/pivot/dashboard/) page has complete working examples that include timer support.

---

## Timer entities

| Entity | Purpose |
| --- | --- |
| `number.{device_suffix}_timer_duration` | Duration in minutes (`1–60`, default `25`) |
| `select.{device_suffix}_timer_state` | State mirror — updated by blueprint and firmware (`idle` / `running` / `paused` / `alerting`) |
| `text.{device_suffix}_timer_end` | Internal — stores an ISO end timestamp while running, and a `P{seconds}` remaining-time value while paused |

All three entities are **disabled by default** and live under the Pivot device in the HA device registry. `text.{device_suffix}_timer_end` is managed entirely by the blueprint; you do not need to interact with it directly.

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
          {% set device_suffix = 'ha_voice_lounge' %}
          {% set state = states('select.' ~ device_suffix ~ '_timer_state') %}
          {% set end_str = states('text.' ~ device_suffix ~ '_timer_end') %}
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
          {% set device_suffix = 'ha_voice_lounge' %}
          {% set state = states('select.' ~ device_suffix ~ '_timer_state') %}
          {% set end_str = states('text.' ~ device_suffix ~ '_timer_end') %}
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
          {% set device_suffix = 'ha_voice_lounge' %}
          {% set state = states('select.' ~ device_suffix ~ '_timer_state') %}
          {% set end_str = states('text.' ~ device_suffix ~ '_timer_end') %}
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

The `timer_ringing` switch can be turned on to trigger the timer alarm at any time — playing the built-in alarm sound and LED animation, just as if a timer had finished. This is useful for testing the alarm, triggering it from a dashboard, or using it as part of a larger automation.

### Silent timers

The `silent_timer` switch suppresses the audible alarm while keeping the visual behaviour. When enabled, timers still finish normally — the LED ring pulses and the timer enters its normal finished state — but no sound is played.

This is useful for focus sessions, late-night use, or situations where visual feedback is enough. Because this applies at the device level, it works consistently across both Pivot timers and voice-set timers.

---

## Tips

**Setting the duration** — If a bank is assigned, turn the knob while idle to select a duration. The gauge fills to reflect the selected proportion of the maximum range and announces the time when the knob settles. Knob and dashboard durations are in whole minutes — minimum 1 minute. For sub-minute timers, use voice control. You can also set `number.{device_suffix}_timer_duration` directly in HA at any time.

**Gauge sync** — The gauge jumps to 100% the moment the timer starts and syncs every 30 seconds as time runs out. On a typical 25-minute timer this is less than one LED-width per update and looks continuous in practice.

**Long pressing from another bank** — Long press cancel only fires when the timer bank is the active bank. This is intentional — long press on other banks is left free for custom actions. Switch back to the timer bank first, then long press.

**Changing duration mid-session** — Turning the knob while the timer is running or paused does nothing. To change the duration, long press to cancel, then turn the knob to select a new time.

**Multiple devices** — Create one **Pivot - Timer Control** automation and one **Pivot - Timer - Voice** automation per Pivot device, each with its own suffix and timer ringing switch. The **Pivot - Timer Toggle Script** is the exception — it is set up once and works across all devices.

**Pausing and resuming** — Remaining time is stored in `text.{device_suffix}_timer_end` and survives HA restarts. If HA restarts while the timer is running and the stored end time has already passed, the countdown sync will detect this within 30 seconds and trigger the finish sequence normally.

**Paused auto-cancel** — If the timer is left paused for more than 15 minutes, it resets automatically and `timer_state` returns to `idle`. This prevents the timer from being stuck paused indefinitely.

---

## Why not use the built-in HA timer?

Home Assistant has a built-in `timer` domain, and it is a reasonable question why Pivot does not use it.

**ESPHome cannot expose a `timer` entity.** The ESPHome integration supports a defined set of entity platforms — `sensor`, `switch`, `number`, `select`, `text`, `button`, and a few others. The `timer` domain is not one of them. Any HA timer would have to be a standalone helper created separately, with no connection to the Pivot device in the device registry.

**The `alerting` state does not exist in the HA timer.** HA's built-in timer has three states: `idle`, `active`, and `paused`. When it finishes it fires a `timer.finished` event and returns to `idle`. Pivot needs a persistent `alerting` state so the firmware knows to hold the LED ring animation and alarm sound until the user actively dismisses it. Even if an HA timer were used for the countdown, `select.{device_suffix}_timer_state` would still be needed to track the alerting phase.

**The stock VPE timer already uses the `timer` domain.** The Voice Preview Edition handles spoken timer commands through Assist, which targets HA timer entities. Introducing Pivot-owned `timer.` entities would create ambiguity about what Assist is controlling, which is exactly the problem the Pivot - Timer - Voice blueprint is designed to resolve cleanly.
