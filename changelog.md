---
layout: page
title: Changelog
permalink: /changelog/
---

## Compatibility

| Firmware | Integration | ESPHome Device Builder | Home Assistant |
| --- | --- | --- | --- |
| v0.0.21 | v0.0.78 | 2026.4.0+ | 2024.4.0+ |

**Always run the latest firmware and integration together.** If you update the integration, check the firmware changelog for any matching firmware release.

> **Home Assistant 2024.4.0 or later is required.** The integration's bundled blueprints use the `action:` key and `trigger:` shorthand syntax introduced in that release. The integration itself will install on older versions, but the blueprints will silently fail to validate.

---

## Integration

<details markdown="1">
<summary>v0.0.78</summary>

- **Fix:** Climate and cover value announcements now speak the commanded value rather than reading from the entity attribute. Previously the announcement fired ~200 ms after the debounced service call, which was not enough time for cloud devices to confirm the new state — so the spoken temperature or position was always the previous value. Climate now computes the target temperature from the knob position using the entity's min/max range; cover announces the commanded position directly.

</details>

<details markdown="1">
<summary>v0.0.77</summary>

- **Fix:** Climate, cover, and media player service calls are now debounced (400 ms) when the knob is turned quickly. Previously, rapid knob turns sent a service call on every tick, which could cause cloud-connected devices (e.g. Tuya, MagiqTouch) to drop commands or go temporarily offline. The actual command now only fires once the knob settles. Visual feedback (LED ring colour, `pivot_knob_turn` events) is unaffected and still updates in real-time.

</details>

<details markdown="1">
<summary>v0.0.76</summary>

- **Fix:** Configure screen bank entity assignments were shifted by one and Bank 4 was unreachable. The options flow was using the 0-based form field key directly as the entity ID key, producing `bank_0_entity` through `bank_3_entity` entity lookups that no longer exist after the v0.0.75 rename. Form key and entity ID key are now decoupled with the correct `+1` offset applied only to the entity ID lookup.


</details>

<details markdown="1">
<summary>v0.0.75</summary>

- **Breaking change:** Bank entity IDs are now 1-based to match user-facing bank numbers. `bank_0_*` entities are now `bank_1_*`, `bank_1_*` are now `bank_2_*`, and so on through `bank_4_*`. Existing entity registry entries for renamed entities will be orphaned and must be removed. Any custom automations or dashboards referencing the old entity IDs will need to be updated. The firmware, integration, dashboard YAML, and blueprints are all updated together in this release.


</details>

<details markdown="1">
<summary>v0.0.74</summary>

- **Fix:** All Pivot devices failed to load on startup with `AttributeError: 'EntityRegistry' object has no attribute 'async_entries_for_device'`. The button event entity lookup was using an instance method form of the API that was added in a later HA version. Reverted to the stable module-level `er.async_entries_for_device(registry, device_id)` call, which works across all supported versions. The crash occurred after bank control listeners were registered but before they were stored, so knob control continued to work while button toggle and announcements were completely non-functional.


</details>

<details markdown="1">
<summary>v0.0.73</summary>

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


</details>

<details markdown="1">
<summary>v0.0.72</summary>

- **Fix:** Climate temperature mapping now reads `min_temp`, `max_temp`, and `target_temp_step` directly from the thermostat entity's attributes instead of assuming a hardcoded 16–30 °C range. Works correctly with Fahrenheit thermostats, non-standard ranges, and any step size — no configuration required.
- **Change:** All legacy migration code that read and wrote `scripts.yaml` and `automations.yaml` has been removed. This eliminates a category of fragile file-mutation behaviour that was a holdover from the removed Automatic (managed) mode.


</details>

<details markdown="1">
<summary>v0.0.71</summary>

- **Fix:** Turning off **System Announcements** no longer silences value announcements (knob turn). System Announcements and Value Announcements are independent — System Announcements controls bank change and control mode spoken announcements only; per-bank Announce Value switches control value announcements. These were incorrectly coupled.
- **Fix:** `input_number` entities now correctly trigger value announcements and speak their value with unit after the knob settles. They were inadvertently excluded from the announcement domain list despite being a fully supported active domain.
- **Fix:** Switching to a timer bank while the timer is idle no longer causes a spurious write back to the timer duration. A missing sync context meant the bank value update was treated as a physical knob turn, producing a rounding-affected duration write and a false `pivot_timer_duration_set` event.
- **Fix:** Minimum required Home Assistant version corrected to 2024.4.0 in HACS metadata (was incorrectly listed as 2024.1.0, which could lead to a confusing install failure on older versions).


</details>

<details markdown="1">
<summary>v0.0.70</summary>

- **Fix:** Switching to a timer bank now announces "Timer" correctly. The bank change announcement was skipped for timer banks because the bank entity value `timer` contains no dot — which was the guard used to detect a real entity. Timer banks are now handled separately and announce "Timer" explicitly.


</details>

<details markdown="1">
<summary>v0.0.69</summary>

- **Fix:** Value announcements (knob turn) now fire correctly. `hass.async_call_later` is not a valid method on the `hass` object — the debounce call threw an exception on every knob turn, silently preventing announcements. Replaced with `async_call_later(hass, delay, callback)` from `homeassistant.helpers.event`.


</details>

<details markdown="1">
<summary>v0.0.68</summary>

- **Change:** Announcements are now handled natively by the integration. The **Pivot — Announce** blueprint has been removed. To enable announcements, go to **Settings → Devices & Services → Pivot → your device → Configure**, pick a TTS service and speaker, and tick **Enable spoken announcements**. The integration announces bank names on bank change and triple press, and speaks entity values after the knob settles (gated per-bank by the Announce Value switches). The same TTS and speaker settings are shared with the Timer blueprint.


</details>

<details markdown="1">
<summary>v0.0.66</summary>

- **Change:** Manual setup mode removed. Blueprints are always installed. The configure dialog now shows only TTS and speaker settings, with a note that advanced users can build custom automations using Pivot's events.


</details>

<details markdown="1">
<summary>v0.0.65</summary>

- **Fix:** Value announcements no longer fire when an entity is changed externally (dashboard, motion, assist, etc.) after a button press or bank sync. `_sync_value_from_entity` was calling `number.set_value` without a context, so the resulting state change had `parent_id=None` and was treated as a physical knob turn, firing `pivot_knob_turn` and triggering the announce blueprint.


</details>

<details markdown="1">
<summary>v0.0.64</summary>

- **Fix:** **Pivot — Timer** blueprint no longer triggers start/pause on button presses from non-timer banks. The single press condition now checks that the pressed bank matches the timer bank number.


</details>

<details markdown="1">
<summary>v0.0.63</summary>

- **Debug:** Added detailed debug logging to the button press listener — logs which entity is being watched, skipped reconnect transitions, and the resolved press type, control mode, and bank entity on every press.


</details>

<details markdown="1">
<summary>v0.0.62</summary>

- **Fix:** Corrected the fallback device_id lookup for older config entries. The previous fallback incorrectly matched the ESPHome device name against device registry identifiers (which are MAC addresses). Now mirrors the same `host`/`name` lookup the config flow uses.


</details>

<details markdown="1">
<summary>v0.0.61</summary>

- **Fix:** Button event entity lookup now uses `device_class: button` instead of a string match on the entity ID, making it reliable regardless of how the ESPHome device is named in HA. Falls back to any `event`-domain entity on the device if device class is absent.
- **Fix:** Added fallback device_id lookup for config entries set up before `device_id` was stored — resolves via the ESPHome device name. Failures are now logged at WARNING level instead of being silently dropped.


</details>

<details markdown="1">
<summary>v0.0.60</summary>

- **Fix:** Button toggle no longer fires spuriously when the ESPHome device reconnects to Home Assistant. Previously, the `button_press_event` entity restoring its last state on reconnect could cause an unintended entity toggle.


</details>

<details markdown="1">
<summary>v0.0.59</summary>

- **Change:** Button toggle is now handled natively by the integration — the **Pivot — Bank Toggle** script blueprint is no longer needed. Requires a firmware reflash (v0.0.15+). Existing users: delete the `{suffix}_bank_toggle` script from Home Assistant after reflashing.


</details>

<details markdown="1">
<summary>v0.0.58</summary>

- **Change:** **Pivot — Timer** blueprint no longer requires a Bank Number input. The blueprint automatically detects which bank is reserved as a timer by reading the bank entity text entities at runtime. If no timer bank is reserved when the automation is created, the blueprint is inert and activates automatically once one is assigned — no changes to the blueprint needed.


</details>

<details markdown="1">
<summary>v0.0.57</summary>

- **New:** TTS service and media player are now configured once in the integration settings and shared automatically across all blueprints. Two new diagnostic text entities (`text.{device_suffix}_tts_entity`, `text.{device_suffix}_media_player_entity`) are written from the config entry on every setup and reload.
- **Change:** **Pivot — Timer** blueprint no longer has a TTS Entity input — TTS is read automatically from the integration settings.
- **Change:** "Speak bank and mode changes" toggle removed from the configure dialog. The System Announcements switch (`switch.{device_suffix}_announcements`) now uses normal restore behaviour — defaults on at first creation, then user-controlled from the device page.


</details>

<details markdown="1">
<summary>v0.0.54</summary>

- **Fix:** Blueprint files are now bundled inside `custom_components/pivot/blueprints/` so HACS ships them correctly. Previously they lived at the repo root (`blueprints/`) which HACS does not install — `_install_blueprints` was silently copying nothing.
- **Change:** Legacy per-device blueprint files (e.g. `pivot_ha_voice_orange_announcements.yaml`) and backup files (`automations.yaml.pivot_backup`, `scripts.yaml.pivot_backup`) left by a previous Automatic mode installation are now removed automatically on startup.


</details>

<details markdown="1">
<summary>v0.0.53</summary>

- **Fix:** Fixed an `IndentationError` introduced in v0.0.52 — a missing `return` inside the `context.parent_id` guard in the timer bank branch left an empty `if` block, preventing the integration from loading.


</details>

<details markdown="1">
<summary>v0.0.52</summary>

- **Change:** Automatic (managed) mode removed. The integration no longer writes to `scripts.yaml`, `automations.yaml`, or `/config/pivot/`. Any device previously configured in Automatic mode is migrated to Blueprint mode automatically on next restart, and the files Pivot wrote are cleaned up.
- **New:** `pivot_bank_changed` event added. Fired whenever the active bank changes, with `suffix`, `bank`, and `bank_entity` fields.
- **Fix:** Value announcements no longer fire when an entity is changed externally (motion, dashboard, automation). The `pivot_knob_turn` event now only fires when the change originates from a physical knob turn — detected via `context.parent_id`.


</details>

<details markdown="1">
<summary>v0.0.51</summary>

- **Fix:** Bank entity text fields no longer revert to the configured default on every Home Assistant restart. Previously, `async_setup_entry` would overwrite whatever value the entity had restored from state storage with the value stored in the config entry — so any direct edits made outside the configuration flow would be lost on restart. The startup write loop has been removed; entities now restore correctly via Home Assistant's built-in state persistence, and the configure flow continues to write values immediately when the user reconfigures.


</details>

<details markdown="1">
<summary>v0.0.50</summary>

- **Fix:** Fixes a 500 error in the Home Assistant automation editor when opening automations created from the `pivot_announce_bank.yaml` or `pivot_timer.yaml` blueprints after the `mute_entity` input was added. The `entity` selector with `default: ""` was invalid — changed to a `text` selector so an empty value is accepted. Also tightens the mute condition to handle `null` as well as empty string (`not mute_entity or ...`).


</details>

<details markdown="1">
<summary>v0.0.49</summary>

- **New:** Per-bank **Announce Value** switches (`switch.{suffix}_bank_N_announce_value`, one per bank). When enabled, the value of the bank's assigned entity is announced via TTS after the knob settles (~600 ms debounce). Supports `climate` (speaks temperature), `cover` (speaks position), `light`, `media_player`, and `fan` (speak knob value as percent), and `number` (speaks entity state with unit). Off by default.
- **New:** Global **Mute Announcements** switch (`switch.{suffix}_mute_announcements`). When on, all spoken announcements are suppressed — no other behaviour changes. Off by default.


</details>

<details markdown="1">
<summary>v0.0.48</summary>

- **New (Script blueprint):** Adds the **Pivot Timer Toggle** script blueprint. Install once — the script is shared across all Pivot devices, with `device_suffix` and `bank` passed as runtime variables by each dashboard card.


</details>

<details markdown="1">
<summary>v0.0.47</summary>

- **Fix (Timer blueprint):** Duration announcement now plays reliably when the knob is turned slowly. The debounce condition was comparing `states(duration_entity)` against `trigger.event.data.duration` — if the HA entity state hadn't updated within the 150 ms debounce window, the comparison silently failed and no announcement played. Duration is now read directly from the original trigger event data, which is always available immediately.


</details>

<details markdown="1">
<summary>v0.0.46</summary>

- **Fix (Timer blueprint):** Bank entity check is now case-insensitive. `Timer`, `TIMER`, etc. are all accepted alongside the recommended lowercase `timer`.


</details>

<details markdown="1">
<summary>v0.0.45</summary>

- **Change (Timer blueprint):** Reverted the `/5s` gauge sync added in v0.0.44. With multiple devices each running an automation every 5 seconds, the idle overhead outweighs the benefit. The single `/30s` trigger is restored.


</details>

<details markdown="1">
<summary>v0.0.44</summary>

- **Change (Timer blueprint):** Added a `/5s` gauge sync trigger for smoother gauge updates while running. Reverted in v0.0.45.


</details>

<details markdown="1">
<summary>v0.0.43</summary>

- **Change (Timer blueprint):** Long press cancel is now restricted to the timer bank. Previously, a long press on any bank would cancel the timer — this conflicted with long press being reserved for custom actions on other banks. The cancel now only fires when `number.{suffix}_active_bank` matches the configured timer bank number.


</details>

<details markdown="1">
<summary>v0.0.42</summary>

- **New (Timer blueprint):** Adds a **Silent Mode** boolean input (off by default). When enabled, the blueprint sets the `timer_silent_mode` switch on the device before triggering the alarm — suppressing the alarm sound while leaving the LED ring pulse and "stop" wake word intact. Requires firmware v0.0.14 or later; gracefully ignored on older firmware.


</details>

<details markdown="1">
<summary>v0.0.41</summary>

- **New (Timer blueprint):** A dedicated cancel button can now be added to the dashboard. The blueprint accepts `pivot_button_press` with `press_type: long_press` as a secondary cancel trigger alongside the physical long press — enabling a dashboard script to cancel in the same way the physical button does, including the *"Timer cancelled"* TTS announcement. See the [Timer page](/timer#cancel-button) for setup.


</details>

<details markdown="1">
<summary>v0.0.40</summary>

- **Fix (Timer blueprint):** Long press cancel now works reliably. Adds a **Button Event Entity** input (pre-filtered to event entities on the selected Pivot device) and uses a direct state trigger on it instead of relying on `pivot_button_press`.


</details>

<details markdown="1">
<summary>v0.0.39</summary>

- **Change (Timer blueprint):** Replaced separate **Media Player** and **Timer Ringing Switch** entity inputs with a single **Pivot Device** picker. The media player, timer ringing switch, and button event entity are now derived automatically from the selected device using `device_entities()`.


</details>

<details markdown="1">
<summary>v0.0.38</summary>

- **Fix (Timer blueprint):** Long press cancel now reliably triggers the automation. The previous blueprint used a single unfiltered `pivot_button_press` trigger and relied on conditions to match the right device and press type — causing long press traces to be silently displaced from the 5-entry history by gauge sync and other events before they could be inspected.


</details>

<details markdown="1">
<summary>v0.0.37</summary>

- **Fix (Timer blueprint):** Dashboard alarm dismissal now works correctly. The previous approach constructed `switch.{suffix}_timer_ringing` to identify the alarm switch, but the ESPHome device name and the Pivot suffix are independent and don't have to match. A new optional **Timer Ringing Switch** input lets you pick the correct entity directly.


</details>

<details markdown="1">
<summary>v0.0.36</summary>

- **New (Timer blueprint):** A dashboard button can now start, pause, resume, and dismiss the timer alarm by firing a `pivot_button_press` event. See the [Timer page](/timer) for setup instructions. Requires firmware v0.0.13.


</details>

<details markdown="1">
<summary>v0.0.34</summary>

- **Change:** Pivot-owned YAML files are now written to a `/config/pivot/` subfolder instead of the `/config/` root.
- **Fix:** Automation filenames had a double `pivot_` prefix (`pivot_pivot_{suffix}_announcements.yaml`). Files are now named `pivot_{suffix}_announcements.yaml` as intended.
- **Fix:** The duplicate include-line check was comparing exact strings — a format change between versions could bypass it and append a second entry. The check now uses a filename-based marker.
- **Fix:** The write order for `scripts.yaml`/`automations.yaml` was not atomic. Each write now creates the new file first, then rewrites the parent YAML in one pass, then deletes any legacy files.
- **Change:** The include-line write is now self-healing. On every load, Pivot strips all existing entries for its keys and writes the correct single line.


</details>

<details markdown="1">
<summary>v0.0.33</summary>

- **Fix (Timer blueprint):** Long press cancel now works from any bank on the device.
- **Fix (Timer blueprint):** Long press is now correctly restricted to `running` and `paused` states.
- **New (Timer blueprint):** Paused timers auto-cancel after 15 minutes.


</details>

<details markdown="1">
<summary>v0.0.32</summary>

- **Fix (Timer blueprint):** Timer alarm now dismisses reliably on the first press. The alarm is now handed off entirely to device firmware (v0.0.11+): the blueprint sets `timer_state` to `alerting`, the firmware starts the built-in alarm, and dismiss is handled natively in firmware.
- **Change (Timer blueprint):** Switched back to `mode: queued`. Queue depth increased to 10.
- **Change (Timer blueprint):** Long press during `alerting` state is now ignored.


</details>

<details markdown="1">
<summary>v0.0.31</summary>

- **Fix (Timer blueprint):** Finish alert dismiss is now more reliable. The dismiss branch no longer requires the button press to match the configured bank number — any press on the device dismisses the alarm.


</details>

<details markdown="1">
<summary>v0.0.30</summary>

- **Fix (Timer blueprint):** Finish alert now dismisses immediately on any press. The blueprint now uses `mode: parallel` so a button press runs the dismiss branch immediately, regardless of what the alert loop is doing.
- **Fix (Timer blueprint):** The "1 minute timer — press to start" announcement no longer plays after dismissing the finish alert.
- **Fix (Timer blueprint):** A press during the optional TTS finish announcement now also dismisses immediately.
- **Fix (Timer blueprint):** TTS Entity input is now an entity picker (dropdown) instead of a plain text field.
- **New:** Added `alerting` as a valid `timer_state` select option.


</details>

<details markdown="1">
<summary>v0.0.29</summary>

- **Fix:** Passive banks (scene, script, switch, input_boolean) now show no gauge — when switching to a passive bank the LED gauge immediately goes to 0.


</details>

<details markdown="1">
<summary>v0.0.26</summary>

- **Fix (Timer blueprint):** Pressing to dismiss the finish alert no longer immediately re-starts the timer.
- **Change:** Timer duration maximum reduced from 120 to 60 minutes.


</details>

<details markdown="1">
<summary>v0.0.25</summary>

- **Fix (Timer blueprint):** Timer finish alert now reliably dismisses on first button press.
- **Fix (Timer blueprint):** Long press now reliably cancels the timer.
- **Change (Timer blueprint):** Finish alert flash timing increased to 500 ms on / 300 ms off.
- **New (Timer blueprint):** TTS announcements on start, pause, and resume.
- **Fix:** Clamp timer duration to a minimum of 1 minute when set via the knob.


</details>

<details markdown="1">
<summary>v0.0.24</summary>

- **New:** Timer banks now respond to the knob when the timer is idle. Turning the knob sets the timer duration and the gauge shows the selected duration as a proportion of the maximum. When the knob stops moving, the selected time is announced via TTS. Requires the Timer blueprint to also be updated to v0.0.24.


</details>

<details markdown="1">
<summary>v0.0.23</summary>

- **New:** Added **Dim LEDs When Idle** switch. Requires firmware v0.0.9.


</details>

<details markdown="1">
<summary>v0.0.22</summary>

- **New:** `input_number` and `number` entities are now supported.


</details>

<details markdown="1">
<summary>v0.0.21</summary>

- **Fix:** `bank_N_configured_color` entities introduced in v0.0.20 were incorrectly marked as disabled by default. Changed to `entity_category: diagnostic`.


</details>

<details markdown="1">
<summary>v0.0.20</summary>

- **New:** Added `bank_N_configured_color` text entities (one per bank, hidden). Required by firmware v0.0.7.


</details>

<details markdown="1">
<summary>v0.0.19</summary>

- **Fix:** Announcements automation reverted to `mode: single`.


</details>

<details markdown="1">
<summary>v0.0.18</summary>

- **Fix:** Timer finish alert now reliably dismisses on first button press.


</details>

<details markdown="1">
<summary>v0.0.17</summary>

- **Fix:** LED gauge now shows correctly on timer banks.


</details>

<details markdown="1">
<summary>v0.0.16</summary>


> **After updating:** two manual steps are required:
> 1. If you use the timer, go to **Settings → Devices & Services → Pivot → your device → Configure** and set the bank entity for your timer bank to `timer` (lowercase).
> 2. Open your Pivot Timer automation and re-save it so it picks up the updated blueprint logic.

- **Fix:** Switches now correctly restore their state after a Home Assistant restart.
- **Fix:** Timer blueprint now requires the bank entity to be set to `timer`.
- **Change:** Timer finish flash speed increased to 150 ms per flash.
- **Fix:** Dismiss press is now checked at the start of each alert loop iteration.
- **Change:** Announcements automation now uses `mode: restart`.
- **Fix:** Bank toggle script no longer attempts to toggle `timer` as a real entity.


</details>

<details markdown="1">
<summary>v0.0.15</summary>

- **New:** Timer finish now repeats — gauge flashes 3 times and the finish sound plays in a loop (up to 10 times) until dismissed.
- **Change:** Finish sound is now hardcoded to the VPE built-in `timer_finished.flac`.
- **New:** Optional TTS message now plays once before the alert loop begins.


</details>

<details markdown="1">
<summary>v0.0.14</summary>

- **Fix:** Single press no longer causes `idle → running → paused` on reflashed firmware.


</details>

<details markdown="1">
<summary>v0.0.13</summary>

- **Fix:** `switch` and `input_boolean` entities now correctly mark their bank as passive.


</details>

<details markdown="1">
<summary>v0.0.12</summary>

- **Fix:** Changed bank-toggle script and timer blueprint to `mode: single`.


</details>

<details markdown="1">
<summary>v0.0.11</summary>

- **Fix:** Pressing once after a cancel/reset no longer immediately puts the timer into a paused state.


</details>

<details markdown="1">
<summary>v0.0.10</summary>

- **Fix:** Gauge now holds the correct remaining percentage when the timer is paused.


</details>

<details markdown="1">
<summary>v0.0.9</summary>

- **Fix:** Timer blueprint now triggers correctly on single press.


</details>

<details markdown="1">
<summary>v0.0.8</summary>

- **Fix:** Timer entities now appear correctly under the device.
- **Change:** Timer entity set is now `number.{suffix}_timer_duration`, `select.{suffix}_timer_state`, and `text.{suffix}_timer_end`.
- **Change:** Pivot Timer blueprint rewritten to be fully self-contained.


</details>

<details markdown="1">
<summary>v0.0.7</summary>

- **New:** Timer helper feature — three new per-device entities provisioned per device, disabled by default.
- **New:** Pivot Timer blueprint.
- **New:** Timer docs page.


</details>

<details markdown="1">
<summary>v0.0.6</summary>

- **Fix:** Resolved oscillation loop when adjusting brightness via the knob.


</details>

<details markdown="1">
<summary>v0.0.5</summary>

- **Fix:** Resolved a feedback loop in live entity sync where lights reporting intermediate brightness values during a transition could cause the LED gauge to oscillate.


</details>

<details markdown="1">
<summary>v0.0.4</summary>

- **Fix:** Turning off Mirror light colour now restores the colour set in the bank's colour picker.
- **New:** Bank value now stays in sync when the assigned entity is changed externally.


</details>

<details markdown="1">
<summary>v0.0.3</summary>

- Added **Mirror light colour** per-bank switch.
- Removed secondary control value entity.


</details>

<details markdown="1">
<summary>v0.0.2</summary>

- Setup mode screen: descriptions and note added.
- Bank assignment screen: removed colour names from bank labels.
- Triple press announcement now says entity name only.
- Bank change announcements now suppressed when bank is changed externally.
- Added icon files to repo root for HACS.


</details>

<details markdown="1">
<summary>v0.0.1</summary>

- Initial release.

</details>

---

## Firmware

<details markdown="1">
<summary>v0.0.21</summary>

- **New:** In control mode, turning the knob while the voice assistant is speaking adjusts the device volume instead of the active bank value. The LED ring switches to the voice mode volume display for the duration. Triggered only by VA TTS replies — integration announcements (bank values, bank changes) are unaffected.


</details>

<details markdown="1">
<summary>v0.0.20</summary>

- **Fix:** Updated component pins for ESPHome 2026.4.0 compatibility. `audio`, `media_player`, `mdns`, `file`, and `speaker_source` components are now merged into ESPHome core and no longer pinned externally. `sendspin`, `media_source`, and `const` are now sourced from the updated PR #14933. `http_request` pin updated to the current PR #12429 commit. Audio pipeline migrated to the new `audio_file` platform and updated `speaker_source` pipeline API. No functional changes.


</details>

<details markdown="1">
<summary>v0.0.19</summary>

- **Breaking change:** Bank entity ID subscriptions updated from 0-based (`bank_0_*`) to 1-based (`bank_1_*` through `bank_4_*`) to match integration v0.0.75. Must be updated together with the integration — the two are not compatible across this change.


</details>

<details markdown="1">
<summary>v0.0.15</summary>

- **Change:** Single press in control mode no longer calls `script.{suffix}_bank_toggle` directly. Toggle is now handled by the integration natively on the `button_press_event` trigger. Requires integration v0.0.59+.
- **Fix:** `button_press_event` now only fires after media checks — if the press was used to stop an announcement or pause media, HA is no longer notified (preventing a spurious entity toggle).


</details>

<details markdown="1">
<summary>v0.0.14</summary>

- **New:** Adds a `Timer Silent Mode` switch. When on, the alarm sound is suppressed at the end of a Pivot timer — the LED ring still pulses and the "stop" wake word still activates, only the sound is silenced. Off by default. Requires a reflash.


</details>

<details markdown="1">
<summary>v0.0.13</summary>

- **New:** The `timer_ringing` switch is now exposed to Home Assistant, allowing a dashboard button or automation to dismiss the timer alarm by calling `switch.turn_off`. Requires a reflash.


</details>

<details markdown="1">
<summary>v0.0.12</summary>

- **New:** A button press in control mode now cancels an active voice assistant session without firing a `pivot_button_press` event.


</details>

<details markdown="1">
<summary>v0.0.11</summary>

- **New:** Timer alarm is now handled entirely in firmware. When the blueprint sets `timer_state` to `alerting`, the firmware plays the alarm from local flash storage, pulses the LED ring, and enables the "stop" wake word. When dismissed, the firmware calls back to HA to reset the state. Requires a firmware reflash.


</details>

<details markdown="1">
<summary>v0.0.10</summary>

- **Fix:** Passive banks (scene, script, switch, input_boolean) no longer show the full LED ring when **Show Control Value** is enabled. Passive banks now show no gauge.


</details>

<details markdown="1">
<summary>v0.0.9</summary>

- **New:** "Dim LEDs When Idle" feature. When enabled, the control mode gauge dims to 50% brightness after 2 seconds of no interaction. Requires integration v0.0.23.


</details>

<details markdown="1">
<summary>v0.0.8</summary>

- **Fix:** Switches such as Show Control Value and Control Mode now apply correctly after a restart without needing to be toggled.


</details>

<details markdown="1">
<summary>v0.0.7</summary>

- **Fix:** Bank Indicator now always shows the colour set in the bank's colour picker, regardless of mirror state. Requires integration update to v0.0.20.
- **Change:** Removed the `ha_initialized` startup guard and `update_configured_colour_N` workaround scripts.


</details>

<details markdown="1">
<summary>v0.0.6</summary>

- **Fix:** Double-pressing to toggle control mode now plays the sound first, then speaks the announcement.
- **Change:** Triple-pressing in normal/voice mode no longer plays the triple-press sound effect.


</details>

<details markdown="1">
<summary>v0.0.5</summary>

- **New:** Bank Indicator quadrant now shows the bank's configured colour when switching banks.
- **Fix:** `bank_mirror_r_0` global was missing from the firmware.
- **Fix:** Configured-colour backup was being overwritten with the light's colour due to a race condition.


</details>

<details markdown="1">
<summary>v0.0.4</summary>

- **Fix:** `single_press` added to `button_press_event` entity — the firmware now fires this event on every valid single press. Requires a firmware reflash.


</details>

<details markdown="1">
<summary>v0.0.3</summary>

- Removed secondary scroll (hold+turn in Normal mode) — restored to stock behaviour.
- Removed unused mirror firmware globals.


</details>

<details markdown="1">
<summary>v0.0.1</summary>

- Initial release.

</details>
