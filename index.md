---
layout: home
---

Pivot turns your [Home Assistant Voice Preview Edition](https://www.home-assistant.io/voice-pe/) into a physical control knob for Home Assistant.

Assign any entity — a light, fan, media player, climate control, cover, scene, or script — to each of the four colour-coded banks. Turn the knob to adjust it. Press the button to toggle or activate it. The LED ring shows you which bank is active and reflects the current value in real time.

---

## How it works

Pivot consists of two parts that work together:

**[Pivot firmware](/pivot/firmware/)** — custom ESPHome firmware you flash to your VPE. It adds four control banks, knob turn handling, LED colour feedback, and sends events to Home Assistant.

**[Pivot integration](/pivot/integration/)** — a HACS integration for Home Assistant that provisions all the required entities, listens for knob turns, and depending on your setup mode, can create the required scripts and automations automatically.

---

## Get started

New to Pivot? Follow the [getting started guide](/pivot/getting-started/) — it walks you through flashing the firmware, installing the integration, and assigning your first entities.

---

## Repositories

| Repository | Description |
| --- | --- |
| [alistairmerritt/pivot-integration](https://github.com/alistairmerritt/pivot-integration) | HACS integration |
| [alistairmerritt/pivot-firmware](https://github.com/alistairmerritt/pivot-firmware) | ESPHome firmware |
| [alistairmerritt/pivot](https://github.com/alistairmerritt/pivot) | This documentation site |
