---
layout: home
---

Pivot turns your Home Assistant Voice Preview Edition into a physical control dial for Home Assistant while preserving its core voice functionality. It provides four colour-coded banks, each assigned to a different controllable entity, script, or scene. Turn to adjust the entity assigned to the active bank, or press to toggle or activate it.

Rather than opening apps, navigating dashboards, or relying on voice commands, you can simply reach out and control your home directly. The LED ring shows which bank is active and provides real-time feedback on its current value (e.g. brightness, volume, or temperature) and, where relevant, its RGB colour. Support for timers is also in development.

Pivot can be enabled or disabled at any time with a double press, allowing you to switch between physical control and the standard Voice Preview Edition behaviour whenever you need.

---

## A better way to control Home Assistant

- **No apps, no menus**  
  Just turn to adjust, press to activate.

- **Always know what you’re controlling**  
  Each bank represents a different entity, with clear visual feedback from the LED ring and optional spoken announcements when switching.

- **Fast and instinctive**  
  Make adjustments in seconds, without breaking your flow.

- **Works with (almost) anything**  
  Works with controllable entities out of the box and can be extended even further through custom automations. If Home Assistant can control it, Pivot can too.

- **Fully local and customisable**  
  Built on ESPHome and Home Assistant, with no cloud dependency.

---

## Designed for real use

Pivot adds a physical layer of control to Home Assistant for the things you use most often — not just to adjust them, but to quickly see their current state.

Use it for the things you change (or check) often:
- Lighting brightness/state
- Thermostat temperature
- Speaker volume
- Fan speed
- Blinds position
- Timers (in development)

With each bank assigned to something useful, Pivot gives you a faster way to control an entity and/or understand its current state — without opening a dashboard or asking a voice assistant.

---

## Simple interaction

- Turn → adjust the current bank  
- Press → activate / toggle  
- Press + turn → switch banks

---

## How it works

Pivot consists of two parts that work together, both of which are required for a seamless experience:

**[Pivot firmware](/pivot/firmware/)** — custom ESPHome firmware you flash to your VPE. It adds four control banks, knob turn handling, LED colour feedback, and sends events to Home Assistant.

**[Pivot integration](/pivot/integration/)** — a HACS integration for Home Assistant that provisions all the required entities, listens for knob turns, and depending on your setup mode, can create the required scripts and automations automatically.

---

## Get started

New to Pivot? Follow the [getting started guide](/pivot/getting-started/) — it walks you through flashing the firmware, installing the integration, and assigning entities.

---

## Repositories

| Repository | Description |
| --- | --- |
| [alistairmerritt/pivot-integration](https://github.com/alistairmerritt/pivot-integration) | HACS integration |
| [alistairmerritt/pivot-firmware](https://github.com/alistairmerritt/pivot-firmware) | ESPHome firmware |
| [alistairmerritt/pivot](https://github.com/alistairmerritt/pivot) | This documentation site |
