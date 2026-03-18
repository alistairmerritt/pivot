---
layout: page
title: Changelog
permalink: /changelog/
---

## Integration

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

### v0.0.3
- Removed secondary scroll (hold+turn in Normal mode) — restored to stock behaviour of changing the LED ring colour (hue cycling)
- Removed unused mirror firmware globals

### v0.0.1
- Initial release
