---
layout: page
title: Changelog
permalink: /changelog/
---

## Integration

### v0.0.10
- **Fix:** Gauge now holds the correct remaining percentage when the timer is paused, rather than dropping to 0. The pause sequence now writes the current progress to the bank value entity at the moment of pausing.

### v0.0.9
- **Fix:** Timer blueprint now triggers correctly on single press — `pivot_button_press` with `press_type: single_press` was never fired because the bank_toggle script exited early when the bank had no entity assigned. The event is now fired before the entity guard, so automations (including the timer blueprint) always receive single-press events in control mode

### v0.0.8
- **Fix:** Timer entities now appear correctly under the device — HA's `timer` domain does not support the entity-platform mechanism used by custom integrations, so `timer.{suffix}_timer` was silently never created. Replaced with `text.{suffix}_timer_end`, which stores the countdown end time and is managed entirely by the blueprint
- **Change:** Timer entity set is now `number.{suffix}_timer_duration`, `select.{suffix}_timer_state`, and `text.{suffix}_timer_end` (all disabled by default)
- **Change:** Pivot Timer blueprint rewritten to be fully self-contained — no longer uses `timer.*` services; finish is detected in the 5-second gauge-sync loop
- **Change:** Gauge drains from 100% → 0% as the timer counts down (was 0% → 100%)

### v0.0.7
- **New:** Timer helper feature — three new per-device entities provisioned per device, disabled by default
- **New:** Pivot Timer blueprint — turns any bank into a Pomodoro-style countdown timer with single-press start/pause/resume, long-press cancel, live gauge sync, finish sound, and optional TTS announcement
- **New:** Timer docs page with setup guide and blueprint import link

### v0.0.6
- **Fix:** Resolved oscillation loop when adjusting brightness via the knob — intermediate brightness values reported by the light during a transition are now ignored for 2 seconds after a Pivot-initiated change

### v0.0.5
- **Fix:** Resolved a feedback loop in live entity sync where lights reporting intermediate brightness values during a transition could cause the LED gauge to oscillate uncontrollably

### v0.0.4
- **Fix:** Turning off Mirror light colour now restores the colour set in the bank's colour picker, rather than always reverting to the factory default colour
- **New:** Bank value now stays in sync when the assigned entity is changed externally — e.g. dimming a light via a voice command, another dashboard, or a physical switch will now be reflected in the LED gauge on the device

### v0.0.3
- Added **Mirror light colour** per-bank switch — when enabled for a bank assigned to an RGB light, the LED ring mirrors the light's current colour instead of the fixed bank colour
- Turning mirror off restores the default bank colour (Blue/Orange/Green/Purple)
- Removed secondary control value entity

### v0.0.2
- Setup mode screen: descriptions and note added
- Bank assignment screen: removed colour names from bank labels, added fine print note
- Triple press announcement now says entity name only (removed "Control mode on" prefix)
- Bank change announcements now suppressed when bank is changed externally (e.g. via automation)
- Added icon files to repo root for HACS

### v0.0.1
- Initial release

---

## Firmware

### v0.0.4
- **Fix:** `single_press` added to `button_press_event` entity — the firmware now fires this event on every valid single press, so `pivot_button_press` with `press_type: single_press` is correctly received by HA automations (including the timer blueprint) without needing to be in control mode. Requires a firmware reflash.

### v0.0.3
- Removed secondary scroll (hold+turn in Normal mode) — restored to stock behaviour of changing the LED ring colour (hue cycling)
- Removed unused mirror firmware globals

### v0.0.1
- Initial release
