---
layout: page
title: Timer
permalink: /timer/
---

Pivot can act as a physical timer — a Pomodoro session, a cooking countdown, a focus block — controlled entirely from the device's button and reflected on the gauge LEDs. This is an optional feature: the timer entities are provisioned per device but disabled by default, and the blueprint is installed manually.

---

## How it works

Once set up, a single bank on your Pivot device becomes a timer controller:

- **Single press** — start the timer if idle, pause if running, resume if paused
- **Long press** — cancel and reset
- **Gauge LEDs** — drain as the timer counts down (100% = just started, 0% = finished)
- **Finish** — the gauge resets to 0, a sound plays through the media player, and optionally a TTS message is spoken

The duration is set via the `number.{suffix}_timer_duration` entity (default: 25 minutes, range: 1–120 minutes).

---

## Getting started with timers

### Step 1 — Enable the timer entities

The timer helpers are provisioned with every Pivot device but disabled by default. To enable them:

1. Go to **Settings → Devices & Services → Pivot** and open your device.
2. Find the three timer entities listed under the device:
   - `timer.{suffix}_timer` — the countdown timer
   - `number.{suffix}_timer_duration` — duration in minutes
   - `select.{suffix}_timer_state` — state mirror (idle / running / paused)
3. Click each entity, go to **Settings**, and toggle **Enable**.

All three must be enabled for the blueprint to work correctly.

### Step 2 — Set your duration

Open `number.{suffix}_timer_duration` and set your desired countdown length in minutes. The default is 25 minutes (a standard Pomodoro interval). You can change this any time — the new value takes effect the next time you start the timer.

### Step 3 — Install the blueprint

The Pivot Timer blueprint is not installed automatically. Click the button below to import it into Home Assistant:

[![Import Blueprint](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https%3A%2F%2Fraw.githubusercontent.com%2Falistairmerritt%2Fpivot-integration%2Fmain%2Fblueprints%2Fautomation%2Fpivot%2Fpivot_timer.yaml)

Or paste this URL into **Settings → Automations → Blueprints → Import Blueprint**:

```
https://raw.githubusercontent.com/alistairmerritt/pivot-integration/main/blueprints/automation/pivot/pivot_timer.yaml
```

### Step 4 — Create an automation from the blueprint

1. Go to **Settings → Automations → Blueprints** and find **Pivot — Timer**.
2. Click **Create Automation**.
3. Fill in the inputs:

| Input | Description |
| --- | --- |
| **Device Suffix** | Your device suffix, e.g. `ha_voice_lounge` |
| **Bank Number** | Which bank controls the timer (1–4) |
| **Media Player** | Speaker to play the finish sound |
| **Finish Sound URL** | URL of the audio file to play (e.g. `/local/timer_done.mp3`) |
| **TTS Entity** | Optional — text-to-speech entity for spoken finish announcement |
| **Finish Message** | Optional — what to say when the timer finishes (default: "Timer finished") |

4. Save the automation.

That's it — single-press the bank to start.

---

## Timer entities

| Entity | Purpose |
| --- | --- |
| `timer.{suffix}_timer` | Native HA timer — tracks state (idle / active / paused) and remaining time |
| `number.{suffix}_timer_duration` | Duration in minutes (1–120, default 25) |
| `select.{suffix}_timer_state` | State mirror — updated by blueprint (idle / running / paused) |

All three entities are **disabled by default** and live under the Pivot device in the HA device registry.

---

## Tips

**Using the gauge as a visual countdown** — The gauge starts full (100%) when the timer begins and drains to 0% as time runs out. When the timer finishes, it drops to 0. This gives a natural at-a-glance sense of how much time remains.

**Assigning the bank** — Pick a bank that isn't already controlling another entity, or dedicate a specific bank (e.g. Bank 4) as your timer bank. The blueprint's bank input matches the bank number shown on the device (1–4).

**Changing duration mid-session** — Adjusting `number.{suffix}_timer_duration` while a timer is running does not affect the current session. The new value applies the next time you start a fresh timer.

**Multiple timers** — You can create multiple automations from the same blueprint, one per Pivot device, each with its own suffix and bank assignment.

**Pausing and resuming** — The timer saves remaining time across pauses, including HA restarts. If HA restarts while the timer is running and the scheduled finish time has already passed, the timer resets to idle automatically rather than firing a spurious finish event.
