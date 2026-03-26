---
layout: page
title: Timer
permalink: /timer/
---

Pivot can act as a physical timer — a Pomodoro session, a cooking countdown, a focus block — controlled entirely from the device's button and reflected on the gauge LEDs. This is an optional feature: the timer entities are provisioned per device but disabled by default, and the blueprint is installed manually.

---

## How it works

Once set up, a single bank on your Pivot device becomes a timer controller:

- **Knob (idle)** — turn to set the duration; the gauge shows how much of the maximum time is selected and a voice announcement confirms the chosen time (*"25 minute timer — press to start"*)
- **Single press** — start the timer if idle, pause if running, resume if paused
- **Long press** — cancel and reset
- **Gauge LEDs** — fill to 100% when started, drain to 0% as time runs out
- **Finish** — the device switches back to the timer bank, the gauge flashes, a sound plays, and optionally a TTS message is spoken; single press to dismiss

---

## Getting started with timers

### Step 1 — Enable the timer entities

The timer helpers are provisioned with every Pivot device but disabled by default. To enable them:

1. Go to **Settings → Devices & Services → Pivot** and open your device.
2. Find the three timer entities listed under the device:
   - `number.{suffix}_timer_duration` — duration in minutes
   - `select.{suffix}_timer_state` — state mirror (idle / running / paused)
   - `text.{suffix}_timer_end` — internal countdown tracker
3. Click each entity, go to **Settings**, and toggle **Enable**.

All three must be enabled for the blueprint to work correctly.

### Step 2 — Assign the bank to `timer`

The timer blueprint only activates on a bank whose entity is set to the reserved keyword `timer`. This prevents the timer from triggering on banks that have a real entity assigned.

1. Go to **Settings → Devices & Services → Pivot → your device → Configure**
2. On the bank assignment screen, type `timer` (lowercase, no quotes) into the entity field for your chosen bank
3. Save

Once set, `timer` appears as the bank assignment. The knob is active for duration setting when the timer is idle, and passive while running or paused. The bank stays reserved for the timer until you change it.

### Step 3 — Set your duration

Turn the knob on the timer bank to select a duration. The gauge shows the proportion of the maximum (default max: 120 minutes) and TTS announces the selected time once you stop turning. You can also set the duration directly on `number.{suffix}_timer_duration` in HA — the gauge will update the next time you switch to that bank.

### Step 4 — Install the blueprint

The Pivot Timer blueprint is not installed automatically. Click the button below to import it into Home Assistant:

[![Import Blueprint](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https%3A%2F%2Fraw.githubusercontent.com%2Falistairmerritt%2Fpivot-integration%2Fmain%2Fblueprints%2Fautomation%2Fpivot%2Fpivot_timer.yaml)

Or paste this URL into **Settings → Automations → Blueprints → Import Blueprint**:

```
https://raw.githubusercontent.com/alistairmerritt/pivot-integration/main/blueprints/automation/pivot/pivot_timer.yaml
```

### Step 5 — Create an automation from the blueprint

1. Go to **Settings → Automations → Blueprints** and find **Pivot — Timer**.
2. Click **Create Automation**.
3. Fill in the inputs:

| Input | Description |
| --- | --- |
| **Device Suffix** | Your device suffix, e.g. `ha_voice_lounge` |
| **Bank Number** | Which bank controls the timer (1–4) — must match the bank you set to `timer` |
| **Media Player** | Speaker to play the finish sound |
| **TTS Entity** | Optional — text-to-speech entity for spoken finish announcement |
| **Finish Message** | Optional — what to say when the timer finishes (default: "Timer finished") |

4. Save the automation.

That's it — single-press the bank to start.

---

## Timer entities

| Entity | Purpose |
| --- | --- |
| `number.{suffix}_timer_duration` | Duration in minutes (1–120, default 25) |
| `select.{suffix}_timer_state` | State mirror — updated by blueprint (idle / running / paused) |
| `text.{suffix}_timer_end` | Internal — stores ISO end time while running, remaining seconds while paused |

All three entities are **disabled by default** and live under the Pivot device in the HA device registry. The `text.{suffix}_timer_end` entity is managed entirely by the blueprint; you don't need to interact with it directly.

---

## Tips

**Setting the duration with the knob** — Turn the knob on the timer bank while idle. The gauge fills to reflect the selected proportion of the maximum range and TTS announces the time once the knob settles. Turn right for more time, left for less.

**Using the gauge as a visual countdown** — The gauge jumps to 100% the moment you press start and drains to 0% as time runs out, giving an at-a-glance sense of how much time remains.

**Assigning the bank** — Set the bank entity to `timer` in Pivot's device configuration. The timer blueprint only runs on banks explicitly assigned this way — if you later change the bank to a real entity, the timer stops responding to that bank automatically.

**Changing duration mid-session** — Turning the knob while the timer is running or paused does nothing. To change the duration, cancel first (long press), then turn the knob to select a new time.

**Multiple timers** — You can create multiple automations from the same blueprint, one per Pivot device, each with its own suffix and bank assignment.

**Pausing and resuming** — Remaining time is stored in `text.{suffix}_timer_end` and survives HA restarts. If HA restarts while the timer is running and the stored end time has already passed, the gauge sync will detect this within 5 seconds of startup and trigger the finish sequence normally.
