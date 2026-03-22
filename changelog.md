---
layout: page
title: Changelog
permalink: /changelog/
---

## Firmware

### v0.0.5
- **New:** While the knob is actively being turned, the LED ring now shows the bank's configured colour (Blue/Orange/Green/Purple, or your custom colour from the picker) instead of the mirrored light colour. Once you stop turning and the knob is idle for ~1 second, the ring returns to mirroring the actual light colour. This restores the visual bank identity when all your lights are the same colour (e.g. white). Requires a firmware reflash via ESPHome Device Builder → Install → Wirelessly.
- **Fix:** `bank_mirror_r_0` global was missing from the firmware — bank 0's red mirror component was never declared. Fixed alongside the new feature.
- **Fix:** Configured-colour backup was being overwritten with the light's colour due to a race condition — when mirror was turned on, the integration wrote the light's colour to the colour entity before the mirror switch state reached the firmware, causing the backup to save the wrong colour. The backup update is now delayed by 500 ms so the mirror state is known before the decision is made.

---

## Integration

### v0.0.19
- **Fix:** Announcements automation reverted to `mode: single` — `mode: restart` introduced in v0.0.16 caused bank change announcements to be silently cancelled when any secondary state change fired while TTS was in-flight

### v0.0.18
- **Fix:** Timer finish alert now reliably dismisses on first button press — added `script.{suffix}_bank_toggle` as a second dismiss trigger alongside the `pivot_button_press` event. The script fires ~2-3 s after any press via the firmware's HA API call, catching presses that happened during the flash sequence when no event listener was active

### v0.0.17
- **Fix:** LED gauge now shows correctly on timer banks — `timer` banks are no longer marked passive, which was preventing the firmware from displaying the value gauge

### v0.0.16

> **After updating:** two manual steps are required:
> 1. If you use the timer, go to **Settings → Devices & Services → Pivot → your device → Configure** and set the bank entity for your timer bank to `timer` (lowercase). The timer blueprint will not respond until this is set.
> 2. Open your Pivot Timer automation and re-save it so it picks up the updated blueprint logic.

- **Fix:** Switches (e.g. Show Control Value, Control Mode) now correctly restore their state after a Home Assistant restart without needing to be toggled
- **Fix:** Timer blueprint now requires the bank entity to be set to `timer` — the timer will no longer accidentally trigger when a real entity is assigned to the same bank
- **Change:** Timer finish flash speed increased — LED flashes are now 150 ms each (was 400 ms), making the alert snappier and reducing the window in which a dismiss press can be missed
- **Fix:** Dismiss press is now checked at the start of each alert loop iteration (200 ms pre-check), catching any press that fired during the previous flash sequence
- **Change:** Announcements automation now uses `mode: restart` instead of `mode: single` — a new announcement immediately interrupts any in-progress announcement rather than being dropped
- **Fix:** Bank toggle script no longer attempts to toggle `timer` as if it were a real entity — pressing the button on a timer bank now correctly fires only the `pivot_button_press` event

### v0.0.15
- **New:** Timer finish now repeats — gauge flashes 3 times and the finish sound plays in a loop (up to 10 times) until dismissed with a single press on any bank, matching VPE timer behaviour
- **Change:** Finish sound is now hardcoded to the VPE built-in `timer_finished.flac` — removed the user-facing "Finish Sound URL" input
- **New:** Optional TTS message now plays once before the alert loop begins (if configured)

### v0.0.14
- **Fix:** Single press no longer causes `idle → running → paused` on reflashed firmware. On firmware v0.0.4+, the `button_press_event` entity fires `single_press` immediately, then 2–3 seconds later the HA API call to run the bank-toggle script completes and fires a second `pivot_button_press`. The integration now deduplicates these — the bank-toggle script path is suppressed if the firmware path already fired within the last 3 seconds. Pre-reflash devices are unaffected.

### v0.0.13
- **Fix:** `switch` and `input_boolean` entities now correctly mark their bank as passive — the knob is disabled for these domains just as it is for scenes and scripts

### v0.0.12
- **Fix:** Changed bank-toggle script and timer blueprint to `mode: single` — only one instance can run at a time, so duplicate events from rapid or back-to-back presses are dropped rather than queued and processed in sequence

### v0.0.11
- **Fix:** Pressing once after a cancel/reset no longer immediately puts the timer into a paused state. Two defensive changes: stale button press events (older than 2 seconds) are now discarded before any state changes run; automation queue max reduced from 5 to 2 to prevent event pile-up from previous presses triggering back-to-back transitions

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
