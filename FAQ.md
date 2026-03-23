---
layout: page
title: FAQ
permalink: /faq/
---

### General

**What is Pivot?**  
Pivot is a firmware and Home Assistant integration that turns the Home Assistant Voice Preview Edition into a four-bank physical control dial.

Each bank represents a different control, allowing you to adjust or activate an assigned entity using a simple turn and press interaction.

---

**What is a “bank”?**  
A bank is one of four control slots on the dial.

Each bank is assigned to a single entity, script, or scene, and you switch between them using press + turn.

---

**What can I control with Pivot?**  
Pivot is designed to work with controllable Home Assistant entities.

Common examples include:
- Lights
- Media players
- Fans
- Covers
- Climate entities
- Input numbers
- Scripts
- Scenes

It can also be extended further through automations.

---

**Does Pivot replace voice control?**  
No. Pivot preserves the core voice functionality of the Voice Preview Edition.

You can continue using voice as normal — Pivot simply adds a physical control layer alongside it.

---

**Does Pivot change how voice is triggered?**  
Yes. In the stock VPE firmware, a single press starts the voice input. In Pivot, that press is used for physical control instead.

Voice functionality is still preserved, including wake word activation, but the stock press-to-talk shortcut is no longer the default behaviour while Pivot's control mode is enabled.

If you would prefer a manual trigger as well, a good option is to use long press to run a Home Assistant automation that starts a conversation on the device. This is not built in by default, but can be configured manually.

---

**Can I switch between Pivot behaviour and the stock Voice Preview Edition behaviour?**  
Yes. Double press toggles Pivot control mode on or off.

When control mode is enabled, the dial controls your assigned entities using Pivot’s bank system. When control mode is disabled, the device behaves like a standard Voice Preview Edition again, including the default button actions (with exception of double press).

This makes it easy to switch between Pivot and the stock experience at any time.

---

### Setup & Installation

**Do I need to flash custom firmware?**  
Yes. Pivot requires flashing custom firmware to the Voice Preview Edition using ESPHome.

After the initial flash, updates can be installed wirelessly.

---

**Can I go back to the original firmware?**  
Yes. You can re-flash the device with the stock Home Assistant firmware at any time.

---

**Do I need experience with ESPHome?**  
No. The Getting Started guide walks through the process step-by-step.

---

**Which setup mode should I use?**  
- **Automatic (recommended)** — easiest, handles everything for you  
- **Blueprints** — more control, still guided  
- **Manual** — full control for advanced users  

If you're unsure, start with Automatic.

---

### Behaviour & Controls

**How do I use the dial?**  
- Turn → adjust the active bank  
- Press → activate or toggle  
- Press + turn → switch banks  

---

**How do I know which bank I’m on?**  
The LED ring shows the active bank using its configured colour, and banks can optionally announce the assigned entity when you switch to them.

You can also triple press the button to have Pivot announce the current bank and its assigned entity.

---

**Why do the LED colours change?**  
Pivot uses colour to communicate different things:

- **Bank colour** → shows which bank you’re selecting  
- **Entity feedback** → shows the current value or state (e.g. brightness or RGB colour)

Each bank’s configured colour can be customised, so you can choose the colours that make the most sense for your setup.

---

**What does “mirror light colour” mean?**  
When enabled, the LED ring will match the colour of the assigned light.

When disabled, the LEDs use the bank’s configured colour instead.

---

### Firmware & Updates

**How do I update the firmware?**  
In Home Assistant:
- Go to **ESPHome Device Builder**
- Select your device
- Click **Install → Wirelessly**

---

**What’s the difference between firmware and the integration?**  
- **Firmware** controls the physical behaviour (dial input, LEDs, bank switching)  
- **Integration** controls how Pivot interacts with Home Assistant (entities, automations, timers)

---

### Advanced

**Can I use multiple Pivot VPE devices?**  
Yes. Each device uses a unique identifier, allowing multiple VPE devices to run independently.

---

**Does Pivot work without internet?**  
Yes. Pivot runs entirely locally within Home Assistant and ESPHome.

---

**What happens if Home Assistant restarts?**  
Pivot restores its state and continues working once Home Assistant is available again.

---

### Timer

**Can I use Pivot as a timer?**
Yes. A bank can be assigned to a timer, allowing you to:
- Set duration with the dial
- Start/pause with a press
- Receive alerts when the timer finishes

See the [Timer](/pivot/timer/) page for setup instructions.

---

**Is the timer built-in?**
The timer feature is optional and uses additional entities and automations provided by Pivot. It requires a small amount of setup — see the [Timer](/pivot/timer/) page.

---
