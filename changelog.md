---
layout: page
title: Changelog
permalink: /changelog/
---

## Firmware

### v0.0.9
- **New:** "Dim LEDs When Idle" feature. When enabled (via the new **Dim LEDs When Idle** switch in the integration), the control mode gauge dims to 50% brightness after 2 seconds of no interaction (no knob turns, button presses, or bank switches). LEDs snap back to full brightness instantly on next interaction. Fades smoothly to dim over 1.5 s. Only active when **Show Control Value** is also enabled. Requires integration v0.0.23.

### v0.0.8
- **Fix:** Switches such as Show Control Value and Control Mode now apply correctly after a restart without needing to be toggled. After connecting to Home Assistant, the firmware now re-reads all display-affecting state from HA 2 seconds later (once HA has finished pushing entity values) and re-renders the LEDs. Previously, if the HA entity state matched the firmware's cached value, the `on_state` callback was skipped and the initial render could be based on stale defaults.

### v0.0.7
- **Fix:** Bank Indicator (press+turn to switch banks) now always shows the colour set in the bank's colour picker, regardless of mirror state. Previously, when mirror light was on or the assigned light had no colour value, the indicator still showed the default bank colour (Blue/Orange/Green/Purple) instead of any custom colour you had set. Requires both a firmware reflash **and** an integration update to v0.0.20.
- **Change:** Removed the `ha_initialized` startup guard and the `update_configured_colour_N` workaround scripts. The configured colour is now maintained via a dedicated `bank_N_configured_color` text entity (written by the integration's colour picker, never overwritten by the mirror listener) — eliminating the race conditions that made the old approach unreliable.

### v0.0.6
- **Fix:** Double-pressing to toggle control mode now plays the sound first, then speaks the announcement — previously the TTS could cut off the sound or play simultaneously. The LEDs update immediately on press; the HA switch change (which triggers the TTS) is delayed until the sound has finished (~1.2 s)
- **Change:** Triple-pressing in normal/voice mode no longer plays the triple-press sound effect — it now just speaks the announcement ("Control mode off") the same way it does in control mode

### v0.0.5
- **New:** When pressing and turning the knob to switch banks, the Bank Indicator quadrant now shows the bank's configured colour (Blue/Orange/Green/Purple, or your custom colour from the picker) instead of the mirrored light colour. This restores the visual bank identity when all your lights are the same colour (e.g. white). Requires a firmware reflash via ESPHome Device Builder → Install → Wirelessly.
- **Fix:** `bank_mirror_r_0` global was missing from the firmware — bank 0's red mirror component was never declared. Fixed alongside the new feature.
- **Fix:** Configured-colour backup was being overwritten with the light's colour due to a race condition — when mirror was turned on, the integration wrote the light's colour to the colour entity before the mirror switch state reached the firmware. The backup update is now delayed by 500 ms so the mirror state is known before the decision is made.

---

## Integration

### v0.0.25
- **Fix (Timer blueprint):** Timer finish alert now reliably dismisses on first button press. Restructured the alert loop so `dismissed` is only ever set at the sequence top level — variables inside nested `if/then` blocks are not visible to `repeat…until` in HA, which was causing the loop to always run all 10 iterations. The pre-check now uses a `condition` action to abort the iteration when already dismissed, avoiding the nested scope entirely.
- **Fix (Timer blueprint):** Long press now reliably cancels the timer. The 2-second staleness check was discarding long press events that had queued behind gauge sync instances — long press is now exempt from this check.
- **Change (Timer blueprint):** Finish alert flash timing increased to 500 ms on / 300 ms off (was 150 ms) — more visible and easier to tap to dismiss.
- **New (Timer blueprint):** TTS announcements on start (*"25 minute timer started"*), pause (*"Timer paused. 12 minutes remaining"*), and resume (*"Timer resumed"*). Requires a TTS entity to be configured in the blueprint.
- **Fix:** Clamp timer duration to a minimum of 1 minute when set via the knob — prevents an out-of-range error when the knob is at 0%.

### v0.0.24
- **New:** Timer banks now respond to the knob when the timer is idle. Turning the knob sets the timer duration (mapped across the entity's configured min–max range) and the gauge shows the selected duration as a proportion of the maximum. When the knob stops moving, the selected time is announced via TTS (e.g. *"25 minute timer — press to start"*). The knob is passive while the timer is running or paused. Switching to a timer bank while idle now syncs the current duration to the gauge. Requires the Timer blueprint to also be updated to v0.0.24.
- **Change (Timer blueprint):** Timer switches to `mode: queued` to handle simultaneous gauge updates and duration announcements without blocking. Gauge fills to 100% immediately on start. Timer bank auto-switches back to its bank on finish before the alert loop (so the flash is visible from any bank).

### v0.0.23
- **New:** Added **Dim LEDs When Idle** switch. When on, the control mode gauge dims to 50% brightness after 2 seconds of inactivity and snaps back to full brightness the moment you interact with the device. Has no effect unless **Show Control Value** is also enabled. Requires firmware v0.0.9.

### v0.0.22
- **New:** `input_number` and `number` entities are now supported. Assign one to a bank and the knob scales 0–100% across the entity's configured min/max range. The gauge syncs back automatically when the value is changed externally. Single press on a number bank does nothing (there is no meaningful toggle action).

### v0.0.21
- **Fix:** `bank_N_configured_color` entities introduced in v0.0.20 were incorrectly marked as disabled by default (`entity_registry_enabled_default: False`). Disabled entities have no state in HA and cannot be subscribed to by ESPHome, so `bank_mirror_r/g/b_N` was never updated and the Bank Indicator always showed the default identity colour. Changed to `entity_category: diagnostic` — entities are now always enabled but tucked away in the UI. **If you installed v0.0.20:** you may need to manually enable the four `Bank N Configured Colour` entities under Settings → Devices & Services → Pivot → your device → Entities, or remove them from the entity registry and restart HA.

### v0.0.20
- **New:** Added `bank_N_configured_color` text entities (one per bank, hidden). These store the user's chosen colour from the colour picker and are never overwritten by the mirror listener. Required by firmware v0.0.7 to reliably show the correct bank identity colour in the Bank Indicator.

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
