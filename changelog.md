---
layout: page
title: Changelog
permalink: /changelog/
---

## Firmware

### v0.0.14
- **New:** Adds a `Timer Silent Mode` switch, exposed to Home Assistant. When on, the alarm sound is suppressed at the end of a Pivot timer — the LED ring still pulses and the "stop" wake word still activates, only the sound is silenced. Controlled automatically by the blueprint (v0.0.42+) based on the **Silent Mode** input. Off by default (`restore_mode: ALWAYS_OFF`), so existing behaviour is unchanged without a blueprint update. Requires a reflash.

### v0.0.13
- **New:** The `timer_ringing` switch is now exposed to Home Assistant. This allows a dashboard button (or any HA automation) to dismiss the timer alarm by calling `switch.turn_off` on `switch.{suffix}_timer_ringing` — the same path the physical button takes internally. Requires a reflash.

### v0.0.12
- **New:** A button press in control mode now cancels an active voice assistant session without firing a `pivot_button_press` event. Previously, pressing while voice was active sent a single_press event to HA (potentially triggering the bank toggle script) and then stopped the voice. Now the press is intercepted before the event fires — voice is cancelled cleanly and no bank action occurs.

### v0.0.11
- **New:** Timer alarm is now handled entirely in firmware. When the blueprint sets `timer_state` to `alerting`, the firmware subscribes via a new `ha_timer_state` text sensor and turns on the built-in `timer_ringing` switch — playing the alarm sound from local flash storage, pulsing the LED ring, and enabling the "stop" wake word, exactly as the stock voice-assistant timer does. When the alarm is dismissed (button press, "stop" wake word, or 15-minute auto-timeout), the firmware calls back to HA to set `timer_state` to `idle` and turn on `dim_when_idle`. This replaces the previous approach of streaming audio from a remote URL via `media_player.play_media`, which congested the WiFi/API connection and caused button press events to be silently dropped. Requires a firmware reflash.

### v0.0.10
- **Fix:** Passive banks (scene, script, switch, input_boolean) no longer show the full LED ring when **Show Control Value** is enabled. Previously, a solid full-brightness ring was displayed for passive banks, which looked identical to a 100% gauge and was misleading. Passive banks now show no gauge — the LEDs turn off — matching the fact that there is no value to display. The Bank Indicator ring still appears normally while pressing and turning to switch banks.

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

### v0.0.54
- **Fix:** Blueprint files are now bundled inside `custom_components/pivot/blueprints/` so HACS ships them correctly. Previously they lived at the repo root (`blueprints/`) which HACS does not install — `_install_blueprints` was silently copying nothing.
- **Change:** Legacy per-device blueprint files (e.g. `pivot_ha_voice_orange_announcements.yaml`) and backup files (`automations.yaml.pivot_backup`, `scripts.yaml.pivot_backup`) left by a previous Automatic mode installation are now removed automatically on startup.

### v0.0.53
- **Fix:** Fixed an `IndentationError` introduced in v0.0.52 — a missing `return` inside the `context.parent_id` guard in the timer bank branch left an empty `if` block, preventing the integration from loading.

### v0.0.52
- **Change:** Automatic (managed) mode removed. The integration no longer writes to `scripts.yaml`, `automations.yaml`, or `/config/pivot/`. Any device previously configured in Automatic mode is migrated to Blueprint mode automatically on next restart, and the files Pivot wrote are cleaned up.
- **New:** `pivot_bank_changed` event added. Fired whenever the active bank changes, with `suffix`, `bank`, and `bank_entity` fields. The announce blueprint uses this instead of a state trigger, meaning no entity IDs need to be entered as blueprint inputs.
- **Fix:** Value announcements no longer fire when an entity is changed externally (motion, dashboard, automation). The `pivot_knob_turn` event now only fires when the change originates from a physical knob turn — detected via `context.parent_id`.
- **Change:** Separate `pivot_announce_bank.yaml` and `pivot_announce_value.yaml` blueprints replaced by a single **Pivot — Announce** blueprint. All three announcement types (bank change, triple press, value) are handled in one automation with three inputs: Device Suffix, Media Player, and TTS Engine.
- **Change:** `pivot_bank_toggle.yaml` script blueprint simplified to a single **Device Suffix** input — all entity IDs are derived automatically.

### v0.0.51
- **Fix:** Bank entity text fields no longer revert to the configured default on every Home Assistant restart. Previously, `async_setup_entry` would overwrite whatever value the entity had restored from state storage with the value stored in the config entry — so any direct edits made outside the configuration flow would be lost on restart. The startup write loop has been removed; entities now restore correctly via Home Assistant's built-in state persistence, and the configure flow continues to write values immediately when the user reconfigures.

### v0.0.50
- **Fix:** Fixes a 500 error in the Home Assistant automation editor when opening automations created from the `pivot_announce_bank.yaml` or `pivot_timer.yaml` blueprints after the `mute_entity` input was added. The `entity` selector with `default: ""` was invalid — changed to a `text` selector so an empty value is accepted. Also tightens the mute condition to handle `null` as well as empty string (`not mute_entity or ...`).

### v0.0.49
- **New:** Per-bank **Announce Value** switches (`switch.{suffix}_bank_N_announce_value`, one per bank). When enabled, the value of the bank's assigned entity is announced via TTS after the knob settles (~600 ms debounce). Supports `climate` (speaks temperature), `cover` (speaks position), `light`, `media_player`, and `fan` (speak knob value as percent), and `number` (speaks entity state with unit). Off by default.
- **New:** Global **Mute Announcements** switch (`switch.{suffix}_mute_announcements`). When on, all spoken announcements are suppressed — no other behaviour changes. Off by default.
- **Change:** The auto-generated Announcements automation now includes value announcement logic directly — no separate blueprint import needed. The existing `pivot_{suffix}_announcements.yaml` file is regenerated on next HA restart to include bank-value triggers and the debounced value-announcement branch. Per-bank `announce_value` switches gate value announcements; the system `announcements` switch and `mute_announcements` switch gate bank-change and triple-press announcements as before.
- **Change:** Automation mode changed from `single` to `restart` so the 600 ms debounce delay resets on each knob turn and is cancelled automatically when the bank changes.
- **Change:** **Pivot Timer** (`pivot_timer.yaml`) and **Announce Bank** (`pivot_announce_bank.yaml`) blueprints updated with an optional **Mute Announcements Switch** input.
- **Change:** Announcements switch renamed from "Announcements" to "System Announcements" for clarity. Entity ID is unchanged: `switch.{suffix}_announcements`.

### v0.0.48
- **New (Script blueprint):** Adds the **Pivot Timer Toggle** script blueprint. Install once — the script is shared across all Pivot devices, with `device_suffix` and `bank` passed as runtime variables by each dashboard card. Cards call `script.pivot_timer_toggle` directly by entity ID (do not rename it). Fires `pivot_button_press` with `press_type: single_press`, matching the physical button: idle starts, running pauses, paused resumes, alerting dismisses.

### v0.0.47
- **Fix (Timer blueprint):** Duration announcement now plays reliably when the knob is turned slowly. The debounce condition was comparing `states(duration_entity)` against `trigger.event.data.duration` — if the HA entity state hadn't updated within the 150 ms debounce window, the comparison silently failed and no announcement played. Duration is now read directly from the original trigger event data, which is always available immediately.

### v0.0.46
- **Fix (Timer blueprint):** Bank entity check is now case-insensitive. `Timer`, `TIMER`, etc. are all accepted alongside the recommended lowercase `timer`.

### v0.0.45
- **Change (Timer blueprint):** Reverted the `/5s` gauge sync added in v0.0.44. With multiple devices each running an automation every 5 seconds, the idle overhead outweighs the benefit — ~86k evaluations per day across 5 devices when no timer is active. The single `/30s` trigger is restored.

### v0.0.44
- **Change (Timer blueprint):** Added a `/5s` gauge sync trigger for smoother gauge updates while running. Reverted in v0.0.45.

### v0.0.43
- **Change (Timer blueprint):** Long press cancel is now restricted to the timer bank. Previously, a long press on any bank would cancel the timer — this conflicted with long press being reserved for custom actions on other banks. The cancel now only fires when `number.{suffix}_active_bank` matches the configured timer bank number. Long press on all other banks is ignored by the timer blueprint and remains free for custom use.

### v0.0.42
- **New (Timer blueprint):** Adds a **Silent Mode** boolean input (off by default). When enabled, the blueprint sets the `timer_silent_mode` switch on the device before triggering the alarm — suppressing the alarm sound while leaving the LED ring pulse and "stop" wake word intact. Requires firmware v0.0.14 or later; gracefully ignored on older firmware.

### v0.0.41
- **New (Timer blueprint):** A dedicated cancel button can now be added to the dashboard. The blueprint accepts `pivot_button_press` with `press_type: long_press` as a secondary cancel trigger alongside the physical long press — enabling a dashboard script to cancel in the same way the physical button does, including the *"Timer cancelled"* TTS announcement. See the [Timer page](/timer#cancel-button) for setup.

### v0.0.40
- **Fix (Timer blueprint):** Long press cancel now works reliably. Adds a **Button Event Entity** input (pre-filtered to event entities on the selected Pivot device) and uses a direct state trigger on it instead of relying on `pivot_button_press` — which requires the Pivot integration to have a correctly configured device ID that is not always set. The condition checks `event_type == long_press` so single and double presses are ignored.

### v0.0.39
- **Change (Timer blueprint):** Replaced separate **Media Player** and **Timer Ringing Switch** entity inputs with a single **Pivot Device** picker. The media player, timer ringing switch, and button event entity are now derived automatically from the selected device using `device_entities()` — no manual entity picking required, and it works regardless of how the ESPHome device is named in HA.

### v0.0.38
- **Fix (Timer blueprint):** Long press cancel now reliably triggers the automation. The previous blueprint used a single unfiltered `pivot_button_press` trigger and relied on conditions to match the right device and press type — causing long press traces to be silently displaced from the 5-entry history by gauge sync and other events before they could be inspected. Triggers now use `!input` to filter at the source: `single_press` matches suffix + bank + press type, `long_press` matches suffix + press type only. The staleness check is also simplified, and redundant suffix/bank/press_type checks are removed from `choose` conditions throughout. Note: the any-bank behaviour was later revised in v0.0.43 to restrict cancel to the timer bank only.

### v0.0.37
- **Fix (Timer blueprint):** Dashboard alarm dismissal now works correctly. The previous approach constructed `switch.{suffix}_timer_ringing` to identify the alarm switch, but the ESPHome device name and the Pivot suffix are independent and don't have to match — the entity was never found. A new optional **Timer Ringing Switch** input lets you pick the correct entity directly. Leave it blank to use button/wake-word dismiss only.

### v0.0.36
- **New (Timer blueprint):** A dashboard button can now start, pause, resume, and dismiss the timer alarm by firing a `pivot_button_press` event — the same event the physical button fires. See the [Timer page](/timer) for setup instructions. Requires firmware v0.0.13.

### v0.0.34
- **Change:** Pivot-owned YAML files are now written to a `/config/pivot/` subfolder instead of the `/config/` root, keeping your config directory clean.
- **Fix:** Automation filenames had a double `pivot_` prefix (`pivot_pivot_{suffix}_announcements.yaml`). Files are now named `pivot_{suffix}_announcements.yaml` as intended.
- **Fix:** The duplicate include-line check was comparing exact strings — a format change between versions could bypass it and append a second entry, creating a duplicate YAML key that prevented HA from loading scripts or automations. The check now uses a filename-based marker, consistent with how removal already worked.
- **Fix:** The write order for `scripts.yaml`/`automations.yaml` was not atomic — the old file could be deleted before the include line was updated, leaving a broken reference if anything interrupted the sequence. Each write now creates the new file first, then rewrites the parent YAML in one pass, then deletes any legacy files.
- **Change:** The include-line write is now self-healing. On every load, Pivot strips all existing entries for its keys and writes the correct single line — stale, duplicate, or misformatted entries from any prior version are corrected automatically without user action. Migration from flat-path to subfolder layout is handled the same way.

### v0.0.33
- **Fix (Timer blueprint):** Long press cancel now works from any bank on the device. The previous condition required the active bank to equal the configured timer bank at the exact moment of the long press — if you were on a different bank, the condition silently failed and nothing happened. Long press is a deliberate one-second hold, so the bank check is unnecessary; the suffix check still ensures the press comes from the correct device.
- **Fix (Timer blueprint):** Long press is now correctly restricted to `running` and `paused` states. Pressing while `idle` or `alerting` does nothing (alerting is dismissed by single press in firmware).
- **New (Timer blueprint):** Paused timers auto-cancel after 15 minutes. If the timer is left in `paused` state for more than 15 minutes, the gauge resets to 0 and `timer_state` returns to `idle` automatically on the next 5-second gauge sync.

### v0.0.32
- **Fix (Timer blueprint):** Timer alarm now dismisses reliably on the first press. The previous approach of streaming audio from a remote URL via `media_player.play_media` congested the ESPHome API connection, causing `pivot_button_press` events to be silently dropped — presses were never received by HA at all. The alarm is now handed off entirely to device firmware (v0.0.11+): the blueprint sets `timer_state` to `alerting`, the firmware starts the built-in alarm (local sound, LED ring, "stop" wake word), and dismiss is handled natively in firmware, which calls back to HA to reset the state. No blueprint-side dismiss logic, no remote audio download.
- **Change (Timer blueprint):** Switched back to `mode: queued` — the parallel dismiss branch is no longer needed since dismiss is handled in firmware. Queue depth increased to 10.
- **Change (Timer blueprint):** Long press during `alerting` state is now ignored — the guard `not is_state(timer_state_entity, 'alerting')` prevents a long press from corrupting the timer state while the firmware alarm is active.

### v0.0.31
- **Fix (Timer blueprint):** Finish alert dismiss is now more reliable. The dismiss branch no longer requires the button press to match the configured bank number — any press on the device dismisses the alarm. Previously, if `active_bank` in HA diverged from the blueprint's configured bank for any reason, the bank check silently failed and the dismiss branch was skipped. The press would fall through to the start/pause/resume handler, which did nothing (state was `alerting`, not `idle`/`running`/`paused`), leaving the alarm running indefinitely.

### v0.0.30
- **Fix (Timer blueprint):** Finish alert now dismisses immediately on any press. The blueprint previously used `mode: queued`, which meant the alert loop held the automation queue — a button press could not run the dismiss handler until the loop reached a `wait_for_trigger` step. A press during `media_player.play_media` or any other service call was silently dropped, forcing the user to time the press precisely. The blueprint now uses `mode: parallel`: the button press fires a new instance that runs the dismiss branch immediately, regardless of what the alert loop is doing. The loop tracks `timer_state` rather than a local variable, so it exits at the next condition check once dismiss sets the state to `idle`.
- **Fix (Timer blueprint):** The "1 minute timer — press to start" announcement no longer plays after dismissing the finish alert. The flash loop was setting the gauge to 0 while timer state was `idle`, which the integration interpreted as a knob turn mapping to 0 minutes (clamped to 1). Timer state is now set to `alerting` during the alert sequence — the integration returns early for any state other than `idle`, so gauge changes during the alert are ignored.
- **Fix (Timer blueprint):** A press during the optional TTS finish announcement now also dismisses immediately, rather than being lost until the announcement delay expired.
- **Fix (Timer blueprint):** TTS Entity input is now an entity picker (dropdown) instead of a plain text field.
- **New:** Added `alerting` as a valid `timer_state` select option. The integration already ignored gauge changes when state was not `idle`; `alerting` makes this explicit and signals the dismiss branch to intercept presses before any other handler runs.

### v0.0.29
- **Fix:** Passive banks (scene, script, switch, input_boolean) now show no gauge — when switching to a passive bank the LED gauge immediately goes to 0. Previously the gauge held whatever value it had last, giving the false impression that the knob controlled something. Also zeros the gauge if the active bank's entity is reassigned to a passive entity while already on that bank, and on HA startup so the firmware cache is correct after a restart.

### v0.0.26
- **Fix (Timer blueprint):** Pressing to dismiss the finish alert no longer immediately re-starts the timer. A queued automation instance from the dismiss press was reaching the start sequence after the alert exited. A 3-second hold after the alert loop makes the press stale before the queued instance runs.
- **Change:** Timer duration maximum reduced from 120 to 60 minutes — the knob now spans 1–60 minutes. Update `number.{suffix}_timer_duration` on your device if you have an existing value above 60.

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
