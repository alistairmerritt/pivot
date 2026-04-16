---
layout: page
title: Changelog
permalink: /changelog/
---

## Compatibility

| Firmware | Integration | ESPHome Device Builder | Home Assistant |
| --- | --- | --- | --- |
| v0.0.21 | v0.0.76 | 2026.4.0+ | 2024.4.0+ |
| v0.0.20 | v0.0.75 | 2026.4.0+ | 2024.4.0+ |
| v0.0.15–v0.0.19 | v0.0.59–v0.0.74 | 2026.2.0+ | 2024.4.0+ |

**Always run the latest firmware and integration together.** If you update the integration, check the firmware changelog for any matching firmware release.

> **Breaking change in v0.0.75:** Bank entity IDs changed from 0-based (`bank_0_*`) to 1-based (`bank_1_*`). If upgrading from v0.0.74 or earlier, existing entity registry entries for the renamed entities will be orphaned — remove them after upgrading. Any custom automations or dashboards using the old entity IDs must also be updated.

---

## Integration

### v0.0.76
- **Fix:** Configure screen bank entity assignments were shifted by one and Bank 4 was unreachable. The options flow was using the 0-based form field key directly as the entity ID key, producing `bank_0_entity` through `bank_3_entity` entity lookups that no longer exist after the v0.0.75 rename. Form key and entity ID key are now decoupled with the correct `+1` offset applied only to the entity ID lookup.

### v0.0.75
- **Breaking change:** Bank entity IDs are now 1-based to match user-facing bank numbers. `bank_0_*` entities are now `bank_1_*`, `bank_1_*` are now `bank_2_*`, and so on through `bank_4_*`. Existing entity registry entries for renamed entities will be orphaned and must be removed. Any custom automations or dashboards referencing the old entity IDs will need to be updated. The firmware, integration, dashboard YAML, and blueprints are all updated together in this release.

### v0.0.74
- **Fix:** All Pivot devices failed to load on startup with `AttributeError: 'EntityRegistry' object has no attribute 'async_entries_for_device'`. The button event entity lookup was using an instance method form of the API that was added in a later HA version. Reverted to the stable module-level `er.async_entries_for_device(registry, device_id)` call, which works across all supported versions. The crash occurred after bank control listeners were registered but before they were stored, so knob control continued to work while button toggle and announcements were completely non-functional.

### v0.0.73
- **Fix:** Non-dimmable lights (lights with no `brightness` attribute) now sync to 100% when on instead of 0%. Switching to a bank assigned to a non-dimmable light no longer shows an empty gauge.
- **Fix:** `number` and `input_number` entity values are now snapped to the entity's declared step size before being applied. Previously, the exact knob value was written regardless of the entity's step, which could produce out-of-step values on strict entities.
- **Fix:** Climate temperature sync no longer falls back silently on conversion error. If `min_temp`, `max_temp`, or the current temperature cannot be read, the sync is skipped cleanly rather than writing an incorrect value.
- **Fix:** Switch entities reliably publish their restored state immediately on startup. Previously, if the last state was unavailable, the entity could briefly appear unavailable before the next state write.
- **Fix:** The startup push timer on bank colour light entities is now correctly cancelled if the entity is removed before it fires. Previously the cancel handle was not registered with `async_on_remove`, so it could attempt to fire on an entity that no longer existed.
- **Fix:** ESPHome device hostname is now URL-encoded when building the device configuration link. Devices with hyphens, dots, or other special characters in their name no longer produce a malformed link in the device page.
- **Fix:** Config flow now fails clearly if the ESPHome device name cannot be read from the config entry, instead of silently falling back to a wrong suffix and misconfiguring the device.
- **Change:** Config flow suffix field now pre-fills with the suffix detected from the selected ESPHome device, so it does not need to be re-typed.
- **Internal:** Translations file (`translations/en.json`) added — config flow steps, field labels, error messages, and abort reasons now appear correctly in the HA UI.
- **Internal:** `__init__.py` split into focused modules (`bank_control`, `button`, `mirror`, `blueprints`, `announcements`, `entity_mappings`). No change in behaviour.
- **Internal:** `PASSIVE_DOMAINS` consolidated into a single definition in `const.py` — previously duplicated in four places.

### v0.0.72
- **Fix:** Climate temperature mapping now reads `min_temp`, `max_temp`, and `target_temp_step` directly from the thermostat entity's attributes instead of assuming a hardcoded 16–30 °C range. Works correctly with Fahrenheit thermostats, non-standard ranges, and any step size — no configuration required.
- **Change:** All legacy migration code that read and wrote `scripts.yaml` and `automations.yaml` has been removed. This eliminates a category of fragile file-mutation behaviour that was a holdover from the removed Automatic (managed) mode.

### v0.0.71
- **Fix:** Turning off **System Announcements** no longer silences value announcements (knob turn). System Announcements and Value Announcements are independent — System Announcements controls bank change and control mode spoken announcements only; per-bank Announce Value switches control value announcements. These were incorrectly coupled.
- **Fix:** `input_number` entities now correctly trigger value announcements and speak their value with unit after the knob settles. They were inadvertently excluded from the announcement domain list despite being a fully supported active domain.
- **Fix:** Switching to a timer bank while the timer is idle no longer causes a spurious write back to the timer duration. A missing sync context meant the bank value update was treated as a physical knob turn, producing a rounding-affected duration write and a false `pivot_timer_duration_set` event.
- **Fix:** Minimum required Home Assistant version corrected to 2024.4.0 in HACS metadata (was incorrectly listed as 2024.1.0, which could lead to a confusing install failure on older versions).

### v0.0.70
- **Fix:** Switching to a timer bank now announces "Timer" correctly. The bank change announcement was skipped for timer banks because the bank entity value `timer` contains no dot — which was the guard used to detect a real entity. Timer banks are now handled separately and announce "Timer" explicitly.

### v0.0.69
- **Fix:** Value announcements (knob turn) now fire correctly. `hass.async_call_later` is not a valid method on the `hass` object — the debounce call threw an exception on every knob turn, silently preventing announcements. Replaced with `async_call_later(hass, delay, callback)` from `homeassistant.helpers.event`.

### v0.0.68
- **Change:** Announcements are now handled natively by the integration. The **Pivot — Announce** blueprint has been removed. To enable announcements, go to **Settings → Devices & Services → Pivot → your device → Configure**, pick a TTS service and speaker, and tick **Enable spoken announcements**. The integration announces bank names on bank change and triple press, and speaks entity values after the knob settles (gated per-bank by the Announce Value switches). The same TTS and speaker settings are shared with the Timer blueprint.

### v0.0.66
- **Change:** Manual setup mode removed. Blueprints are always installed. The configure dialog now shows only TTS and speaker settings, with a note that advanced users can build custom automations using Pivot's events.

### v0.0.65
- **Fix:** Value announcements no longer fire when an entity is changed externally (dashboard, motion, assist, etc.) after a button press or bank sync. `_sync_value_from_entity` was calling `number.set_value` without a context, so the resulting state change had `parent_id=None` and was treated as a physical knob turn, firing `pivot_knob_turn` and triggering the announce blueprint.

### v0.0.64
- **Fix:** **Pivot — Timer** blueprint no longer triggers start/pause on button presses from non-timer banks. The single press condition now checks that the pressed bank matches the timer bank number.

### v0.0.63
- **Debug:** Added detailed debug logging to the button press listener — logs which entity is being watched, skipped reconnect transitions, and the resolved press type, control mode, and bank entity on every press.

### v0.0.62
- **Fix:** Corrected the fallback device_id lookup for older config entries. The previous fallback incorrectly matched the ESPHome device name against device registry identifiers (which are MAC addresses). Now mirrors the same `host`/`name` lookup the config flow uses.

### v0.0.61
- **Fix:** Button event entity lookup now uses `device_class: button` instead of a string match on the entity ID, making it reliable regardless of how the ESPHome device is named in HA. Falls back to any `event`-domain entity on the device if device class is absent.
- **Fix:** Added fallback device_id lookup for config entries set up before `device_id` was stored — resolves via the ESPHome device name. Failures are now logged at WARNING level instead of being silently dropped.

### v0.0.60
- **Fix:** Button toggle no longer fires spuriously when the ESPHome device reconnects to Home Assistant. Previously, the `button_press_event` entity restoring its last state on reconnect could cause an unintended entity toggle.

### v0.0.59
- **Change:** Button toggle is now handled natively by the integration — the **Pivot — Bank Toggle** script blueprint is no longer needed. Requires a firmware reflash (v0.0.15+). Existing users: delete the `{suffix}_bank_toggle` script from Home Assistant after reflashing.

### v0.0.58
- **Change:** **Pivot — Timer** blueprint no longer requires a Bank Number input. The blueprint automatically detects which bank is reserved as a timer by reading the bank entity text entities at runtime. If no timer bank is reserved when the automation is created, the blueprint is inert and activates automatically once one is assigned — no changes to the blueprint needed.

### v0.0.57
- **New:** TTS service and media player are now configured once in the integration settings and shared automatically across all blueprints. Two new diagnostic text entities (`text.{device_suffix}_tts_entity`, `text.{device_suffix}_media_player_entity`) are written from the config entry on every setup and reload.
- **Change:** **Pivot — Timer** blueprint no longer has a TTS Entity input — TTS is read automatically from the integration settings.
- **Change:** "Speak bank and mode changes" toggle removed from the configure dialog. The System Announcements switch (`switch.{device_suffix}_announcements`) now uses normal restore behaviour — defaults on at first creation, then user-controlled from the device page.

### v0.0.54
- **Fix:** Blueprint files are now bundled inside `custom_components/pivot/blueprints/` so HACS ships them correctly. Previously they lived at the repo root (`blueprints/`) which HACS does not install — `_install_blueprints` was silently copying nothing.
- **Change:** Legacy per-device blueprint files (e.g. `pivot_ha_voice_orange_announcements.yaml`) and backup files (`automations.yaml.pivot_backup`, `scripts.yaml.pivot_backup`) left by a previous Automatic mode installation are now removed automatically on startup.

### v0.0.53
- **Fix:** Fixed an `IndentationError` introduced in v0.0.52 — a missing `return` inside the `context.parent_id` guard in the timer bank branch left an empty `if` block, preventing the integration from loading.

### v0.0.52
- **Change:** Automatic (managed) mode removed. The integration no longer writes to `scripts.yaml`, `automations.yaml`, or `/config/pivot/`. Any device previously configured in Automatic mode is migrated to Blueprint mode automatically on next restart, and the files Pivot wrote are cleaned up.
- **New:** `pivot_bank_changed` event added. Fired whenever the active bank changes, with `suffix`, `bank`, and `bank_entity` fields.
- **Fix:** Value announcements no longer fire when an entity is changed externally (motion, dashboard, automation). The `pivot_knob_turn` event now only fires when the change originates from a physical knob turn — detected via `context.parent_id`.

### v0.0.51
- **Fix:** Bank entity text fields no longer revert to the configured default on every Home Assistant restart. Previously, `async_setup_entry` would overwrite whatever value the entity had restored from state storage with the value stored in the config entry — so any direct edits made outside the configuration flow would be lost on restart. The startup write loop has been removed; entities now restore correctly via Home Assistant's built-in state persistence, and the configure flow continues to write values immediately when the user reconfigures.

### v0.0.50
- **Fix:** Fixes a 500 error in the Home Assistant automation editor when opening automations created from the `pivot_announce_bank.yaml` or `pivot_timer.yaml` blueprints after the `mute_entity` input was added. The `entity` selector with `default: ""` was invalid — changed to a `text` selector so an empty value is accepted. Also tightens the mute condition to handle `null` as well as empty string (`not mute_entity or ...`).

### v0.0.49
- **New:** Per-bank **Announce Value** switches (`switch.{suffix}_bank_N_announce_value`, one per bank). When enabled, the value of the bank's assigned entity is announced via TTS after the knob settles (~600 ms debounce). Supports `climate` (speaks temperature), `cover` (speaks position), `light`, `media_player`, and `fan` (speak knob value as percent), and `number` (speaks entity state with unit). Off by default.
- **New:** Global **Mute Announcements** switch (`switch.{suffix}_mute_announcements`). When on, all spoken announcements are suppressed — no other behaviour changes. Off by default.

### v0.0.48
- **New (Script blueprint):** Adds the **Pivot Timer Toggle** script blueprint. Install once — the script is shared across all Pivot devices, with `device_suffix` and `bank` passed as runtime variables by each dashboard card.

### v0.0.47
- **Fix (Timer blueprint):** Duration announcement now plays reliably when the knob is turned slowly. The debounce condition was comparing `states(duration_entity)` against `trigger.event.data.duration` — if the HA entity state hadn't updated within the 150 ms debounce window, the comparison silently failed and no announcement played. Duration is now read directly from the original trigger event data, which is always available immediately.

### v0.0.46
- **Fix (Timer blueprint):** Bank entity check is now case-insensitive. `Timer`, `TIMER`, etc. are all accepted alongside the recommended lowercase `timer`.

### v0.0.45
- **Change (Timer blueprint):** Reverted the `/5s` gauge sync added in v0.0.44. With multiple devices each running an automation every 5 seconds, the idle overhead outweighs the benefit. The single `/30s` trigger is restored.

### v0.0.44
- **Change (Timer blueprint):** Added a `/5s` gauge sync trigger for smoother gauge updates while running. Reverted in v0.0.45.

### v0.0.43
- **Change (Timer blueprint):** Long press cancel is now restricted to the timer bank. Previously, a long press on any bank would cancel the timer — this conflicted with long press being reserved for custom actions on other banks. The cancel now only fires when `number.{suffix}_active_bank` matches the configured timer bank number.

### v0.0.42
- **New (Timer blueprint):** Adds a **Silent Mode** boolean input (off by default). When enabled, the blueprint sets the `timer_silent_mode` switch on the device before triggering the alarm — suppressing the alarm sound while leaving the LED ring pulse and "stop" wake word intact. Requires firmware v0.0.14 or later; gracefully ignored on older firmware.

### v0.0.41
- **New (Timer blueprint):** A dedicated cancel button can now be added to the dashboard. The blueprint accepts `pivot_button_press` with `press_type: long_press` as a secondary cancel trigger alongside the physical long press — enabling a dashboard script to cancel in the same way the physical button does, including the *"Timer cancelled"* TTS announcement. See the [Timer page](/timer#cancel-button) for setup.

### v0.0.40
- **Fix (Timer blueprint):** Long press cancel now works reliably. Adds a **Button Event Entity** input (pre-filtered to event entities on the selected Pivot device) and uses a direct state trigger on it instead of relying on `pivot_button_press`.

### v0.0.39
- **Change (Timer blueprint):** Replaced separate **Media Player** and **Timer Ringing Switch** entity inputs with a single **Pivot Device** picker. The media player, timer ringing switch, and button event entity are now derived automatically from the selected device using `device_entities()`.

### v0.0.38
- **Fix (Timer blueprint):** Long press cancel now reliably triggers the automation. The previous blueprint used a single unfiltered `pivot_button_press` trigger and relied on conditions to match the right device and press type — causing long press traces to be silently displaced from the 5-entry history by gauge sync and other events before they could be inspected.

### v0.0.37
- **Fix (Timer blueprint):** Dashboard alarm dismissal now works correctly. The previous approach constructed `switch.{suffix}_timer_ringing` to identify the alarm switch, but the ESPHome device name and the Pivot suffix are independent and don't have to match. A new optional **Timer Ringing Switch** input lets you pick the correct entity directly.

### v0.0.36
- **New (Timer blueprint):** A dashboard button can now start, pause, resume, and dismiss the timer alarm by firing a `pivot_button_press` event. See the [Timer page](/timer) for setup instructions. Requires firmware v0.0.13.

### v0.0.34
- **Change:** Pivot-owned YAML files are now written to a `/config/pivot/` subfolder instead of the `/config/` root.
- **Fix:** Automation filenames had a double `pivot_` prefix (`pivot_pivot_{suffix}_announcements.yaml`). Files are now named `pivot_{suffix}_announcements.yaml` as intended.
- **Fix:** The duplicate include-line check was comparing exact strings — a format change between versions could bypass it and append a second entry. The check now uses a filename-based marker.
- **Fix:** The write order for `scripts.yaml`/`automations.yaml` was not atomic. Each write now creates the new file first, then rewrites the parent YAML in one pass, then deletes any legacy files.
- **Change:** The include-line write is now self-healing. On every load, Pivot strips all existing entries for its keys and writes the correct single line.

### v0.0.33
- **Fix (Timer blueprint):** Long press cancel now works from any bank on the device.
- **Fix (Timer blueprint):** Long press is now correctly restricted to `running` and `paused` states.
- **New (Timer blueprint):** Paused timers auto-cancel after 15 minutes.

### v0.0.32
- **Fix (Timer blueprint):** Timer alarm now dismisses reliably on the first press. The alarm is now handed off entirely to device firmware (v0.0.11+): the blueprint sets `timer_state` to `alerting`, the firmware starts the built-in alarm, and dismiss is handled natively in firmware.
- **Change (Timer blueprint):** Switched back to `mode: queued`. Queue depth increased to 10.
- **Change (Timer blueprint):** Long press during `alerting` state is now ignored.

### v0.0.31
- **Fix (Timer blueprint):** Finish alert dismiss is now more reliable. The dismiss branch no longer requires the button press to match the configured bank number — any press on the device dismisses the alarm.

### v0.0.30
- **Fix (Timer blueprint):** Finish alert now dismisses immediately on any press. The blueprint now uses `mode: parallel` so a button press runs the dismiss branch immediately, regardless of what the alert loop is doing.
- **Fix (Timer blueprint):** The "1 minute timer — press to start" announcement no longer plays after dismissing the finish alert.
- **Fix (Timer blueprint):** A press during the optional TTS finish announcement now also dismisses immediately.
- **Fix (Timer blueprint):** TTS Entity input is now an entity picker (dropdown) instead of a plain text field.
- **New:** Added `alerting` as a valid `timer_state` select option.

### v0.0.29
- **Fix:** Passive banks (scene, script, switch, input_boolean) now show no gauge — when switching to a passive bank the LED gauge immediately goes to 0.

### v0.0.26
- **Fix (Timer blueprint):** Pressing to dismiss the finish alert no longer immediately re-starts the timer.
- **Change:** Timer duration maximum reduced from 120 to 60 minutes.

### v0.0.25
- **Fix (Timer blueprint):** Timer finish alert now reliably dismisses on first button press.
- **Fix (Timer blueprint):** Long press now reliably cancels the timer.
- **Change (Timer blueprint):** Finish alert flash timing increased to 500 ms on / 300 ms off.
- **New (Timer blueprint):** TTS announcements on start, pause, and resume.
- **Fix:** Clamp timer duration to a minimum of 1 minute when set via the knob.

### v0.0.24
- **New:** Timer banks now respond to the knob when the timer is idle. Turning the knob sets the timer duration and the gauge shows the selected duration as a proportion of the maximum. When the knob stops moving, the selected time is announced via TTS. Requires the Timer blueprint to also be updated to v0.0.24.

### v0.0.23
- **New:** Added **Dim LEDs When Idle** switch. Requires firmware v0.0.9.

### v0.0.22
- **New:** `input_number` and `number` entities are now supported.

### v0.0.21
- **Fix:** `bank_N_configured_color` entities introduced in v0.0.20 were incorrectly marked as disabled by default. Changed to `entity_category: diagnostic`.

### v0.0.20
- **New:** Added `bank_N_configured_color` text entities (one per bank, hidden). Required by firmware v0.0.7.

### v0.0.19
- **Fix:** Announcements automation reverted to `mode: single`.

### v0.0.18
- **Fix:** Timer finish alert now reliably dismisses on first button press.

### v0.0.17
- **Fix:** LED gauge now shows correctly on timer banks.

### v0.0.16

> **After updating:** two manual steps are required:
> 1. If you use the timer, go to **Settings → Devices & Services → Pivot → your device → Configure** and set the bank entity for your timer bank to `timer` (lowercase).
> 2. Open your Pivot Timer automation and re-save it so it picks up the updated blueprint logic.

- **Fix:** Switches now correctly restore their state after a Home Assistant restart.
- **Fix:** Timer blueprint now requires the bank entity to be set to `timer`.
- **Change:** Timer finish flash speed increased to 150 ms per flash.
- **Fix:** Dismiss press is now checked at the start of each alert loop iteration.
- **Change:** Announcements automation now uses `mode: restart`.
- **Fix:** Bank toggle script no longer attempts to toggle `timer` as a real entity.

### v0.0.15
- **New:** Timer finish now repeats — gauge flashes 3 times and the finish sound plays in a loop (up to 10 times) until dismissed.
- **Change:** Finish sound is now hardcoded to the VPE built-in `timer_finished.flac`.
- **New:** Optional TTS message now plays once before the alert loop begins.

### v0.0.14
- **Fix:** Single press no longer causes `idle → running → paused` on reflashed firmware.

### v0.0.13
- **Fix:** `switch` and `input_boolean` entities now correctly mark their bank as passive.

### v0.0.12
- **Fix:** Changed bank-toggle script and timer blueprint to `mode: single`.

### v0.0.11
- **Fix:** Pressing once after a cancel/reset no longer immediately puts the timer into a paused state.

### v0.0.10
- **Fix:** Gauge now holds the correct remaining percentage when the timer is paused.

### v0.0.9
- **Fix:** Timer blueprint now triggers correctly on single press.

### v0.0.8
- **Fix:** Timer entities now appear correctly under the device.
- **Change:** Timer entity set is now `number.{suffix}_timer_duration`, `select.{suffix}_timer_state`, and `text.{suffix}_timer_end`.
- **Change:** Pivot Timer blueprint rewritten to be fully self-contained.

### v0.0.7
- **New:** Timer helper feature — three new per-device entities provisioned per device, disabled by default.
- **New:** Pivot Timer blueprint.
- **New:** Timer docs page.

### v0.0.6
- **Fix:** Resolved oscillation loop when adjusting brightness via the knob.

### v0.0.5
- **Fix:** Resolved a feedback loop in live entity sync where lights reporting intermediate brightness values during a transition could cause the LED gauge to oscillate.

### v0.0.4
- **Fix:** Turning off Mirror light colour now restores the colour set in the bank's colour picker.
- **New:** Bank value now stays in sync when the assigned entity is changed externally.

### v0.0.3
- Added **Mirror light colour** per-bank switch.
- Removed secondary control value entity.

### v0.0.2
- Setup mode screen: descriptions and note added.
- Bank assignment screen: removed colour names from bank labels.
- Triple press announcement now says entity name only.
- Bank change announcements now suppressed when bank is changed externally.
- Added icon files to repo root for HACS.

### v0.0.1
- Initial release.

---

## Firmware

### v0.0.21
- **New:** In control mode, turning the knob while the voice assistant is speaking adjusts the device volume instead of the active bank value. The LED ring switches to the voice mode volume display for the duration. Triggered only by VA TTS replies — integration announcements (bank values, bank changes) are unaffected.

### v0.0.20
- **Fix:** Updated component pins for ESPHome 2026.4.0 compatibility. `audio`, `media_player`, `mdns`, `file`, and `speaker_source` components are now merged into ESPHome core and no longer pinned externally. `sendspin`, `media_source`, and `const` are now sourced from the updated PR #14933. `http_request` pin updated to the current PR #12429 commit. Audio pipeline migrated to the new `audio_file` platform and updated `speaker_source` pipeline API. No functional changes.

### v0.0.19
- **Breaking change:** Bank entity ID subscriptions updated from 0-based (`bank_0_*`) to 1-based (`bank_1_*` through `bank_4_*`) to match integration v0.0.75. Must be updated together with the integration — the two are not compatible across this change.

### v0.0.15
- **Change:** Single press in control mode no longer calls `script.{suffix}_bank_toggle` directly. Toggle is now handled by the integration natively on the `button_press_event` trigger. Requires integration v0.0.59+.
- **Fix:** `button_press_event` now only fires after media checks — if the press was used to stop an announcement or pause media, HA is no longer notified (preventing a spurious entity toggle).

### v0.0.14
- **New:** Adds a `Timer Silent Mode` switch. When on, the alarm sound is suppressed at the end of a Pivot timer — the LED ring still pulses and the "stop" wake word still activates, only the sound is silenced. Off by default. Requires a reflash.

### v0.0.13
- **New:** The `timer_ringing` switch is now exposed to Home Assistant, allowing a dashboard button or automation to dismiss the timer alarm by calling `switch.turn_off`. Requires a reflash.

### v0.0.12
- **New:** A button press in control mode now cancels an active voice assistant session without firing a `pivot_button_press` event.

### v0.0.11
- **New:** Timer alarm is now handled entirely in firmware. When the blueprint sets `timer_state` to `alerting`, the firmware plays the alarm from local flash storage, pulses the LED ring, and enables the "stop" wake word. When dismissed, the firmware calls back to HA to reset the state. Requires a firmware reflash.

### v0.0.10
- **Fix:** Passive banks (scene, script, switch, input_boolean) no longer show the full LED ring when **Show Control Value** is enabled. Passive banks now show no gauge.

### v0.0.9
- **New:** "Dim LEDs When Idle" feature. When enabled, the control mode gauge dims to 50% brightness after 2 seconds of no interaction. Requires integration v0.0.23.

### v0.0.8
- **Fix:** Switches such as Show Control Value and Control Mode now apply correctly after a restart without needing to be toggled.

### v0.0.7
- **Fix:** Bank Indicator now always shows the colour set in the bank's colour picker, regardless of mirror state. Requires integration update to v0.0.20.
- **Change:** Removed the `ha_initialized` startup guard and `update_configured_colour_N` workaround scripts.

### v0.0.6
- **Fix:** Double-pressing to toggle control mode now plays the sound first, then speaks the announcement.
- **Change:** Triple-pressing in normal/voice mode no longer plays the triple-press sound effect.

### v0.0.5
- **New:** Bank Indicator quadrant now shows the bank's configured colour when switching banks.
- **Fix:** `bank_mirror_r_0` global was missing from the firmware.
- **Fix:** Configured-colour backup was being overwritten with the light's colour due to a race condition.

### v0.0.4
- **Fix:** `single_press` added to `button_press_event` entity — the firmware now fires this event on every valid single press. Requires a firmware reflash.

### v0.0.3
- Removed secondary scroll (hold+turn in Normal mode) — restored to stock behaviour.
- Removed unused mirror firmware globals.

### v0.0.1
- Initial release.
