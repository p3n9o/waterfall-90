# waterfall-90

**A browser-based waterfall control console for the Xiegu G90 + DE‑19 digital interface.**

waterfall‑90 turns a Chromium-based browser tab into a lightweight CAT/PTT/waterfall control surface for the Xiegu G90 transceiver, using **Web Serial**, **WebHID**, and the **Web Audio API** to talk to the radio directly — no drivers, no desktop app, no data ever leaving the page. On top of standard rig control, it includes an experimental **image/text-to-waterfall transmitter** that converts a photo or a line of text into an audio spectrogram, so it redraws as a picture on a receiving station's waterfall display.

> This is a hobbyist project for licensed amateur radio operators. Check your licence conditions (permitted modes, bandwidth, power, station identification) before transmitting anything with it.
> **Almost 100% fully vibe-coded with Claude AI.**

---

## Features

### Rig connectivity
- **CAT control over Web Serial** — speaks Icom CI‑V, since the G90 behaves as an IC‑7100 / IC‑756PRO clone (default address `70`, controller `E0`, 19200 baud, configurable).
- **Read/set VFO frequency** directly from the browser, with a live MHz readout.
- **Multiple PTT methods**, selectable per setup:
  - CI‑V command (`1C 00`) over the CAT connection
  - Serial **RTS** line (hardware PTT)
  - Serial **DTR** line (hardware PTT)
  - **WebHID GPIO** (CM108-style USB sound-card interfaces, configurable GPIO bit)
- **DE‑19 audio device connection** — separate input (receive audio for the waterfall) and output (mic/data audio to the radio) devices, with explicit device selection via `setSinkId` for multi-soundcard machines.
- Automatic **RTS/DTR de-assertion on connect** so simply opening the serial port can never accidentally key the transmitter.

### Live receive waterfall
- Real-time scrolling waterfall rendered from live audio input.
- Configurable **gain**, **min/max frequency window**, **scroll speed**, and **FFT size** (1024–8192).
- Four colour palettes: Phosphor green, Amber, Ice blue, Classic (blue→red).

### Image & text → waterfall transmit
- Drop or select a **photo**, or type a line of **text**, and transmit it as an audio picture that redraws in a receiving station's waterfall.
- Renders audio using a proper **inverse-STFT / overlap-add synthesis** (Hann-windowed, 75% overlap, phase-continuous, cross-faded between columns) rather than naive tone on/off switching — this avoids the broadband splatter and clicking that simple tone-based encoders produce.
- **Auto-fit & auto-orientation**: time-columns are automatically matched to the source image/text aspect ratio so the transmission isn't wasted on silent letterbox padding.
- **Auto-detected brightness inversion** for typical dark-ink-on-light-paper photos, with manual override.
- **Flip horizontal / flip vertical** options applied at render time, so you can match the orientation convention of a receiving station's waterfall.
- Adjustable **image encoding resolution**: time columns (temporal detail) and frequency rows (spatial/tonal detail), independently tunable against the configured passband.
- Adjustable **transmit speed** (seconds per column) trading detail against on-air time, with a live estimated-duration readout.
- Adjustable **brightness curve (gamma)**.
- **Preview to speaker** before committing to transmit over the air.
- **Live TX preview** on the same waterfall canvas used for receive, using the same axis convention, so you can watch your own picture "draw" as it transmits.
- **Per-column power normalization** plus global peak normalization and a soft-knee limiter, keeping the signal's crest factor low so it sits under the rig's ALC threshold instead of pumping/splattering it.

### Test tuning
- Transmit a constant, unmodulated carrier at a configurable frequency for antenna/amp tuning.
- Configurable **safety cutoff timer** (5–120s) that auto-releases PTT.

### Safety
- **Adaptive PTT safety timeout** — defaults to a 2-minute failsafe for open-ended keying (e.g. the manual hold-to-talk button), but automatically stretches to cover the full length of a known-duration transmission (rendered image or configured tune cutoff), so long transmissions are never cut off early while still guaranteeing the rig can never be left stuck on TX.
- Visible **panic / force PTT release** button the moment PTT is engaged, which forces every keying mechanism (CAT, serial lines, HID) low regardless of which method is currently active.
- Automatic PTT release on mouse-up anywhere on the page, tab blur, or tab visibility change — not just on the button itself.
- Live **TX audio level meter** and an ALC-behaviour warning for dense, high-content images that can drive the rig harder than typical voice/FT8 signals.

### Interface
- Two-tab layout: **Radio** (VFO, PTT, waterfall, image/text transmit, tuning) and **Settings** (waterfall display, CAT control, audio devices, image encoding, test tuning).
- Automatic **browser capability warning** if Web Serial and/or WebHID aren't available, with waterfall and image/audio tools still usable without hardware.
- Runs **entirely client-side** — no backend, no telemetry, nothing leaves the page.

---

## Requirements

- A **Chromium-based desktop browser** (Chrome, Edge, or Opera) served over **HTTPS** (or `localhost`) for Web Serial and WebHID support.
- A **Xiegu G90** transceiver and a **DE‑19** (or compatible CM108-style) digital interface, connected via USB.
- Only one application can hold the CAT port or audio device at a time — close WSJT‑X, SDR Console, OmniRig, etc. before connecting here, or bridge through `rigctld`.

## Getting started

1. Open `index.html` in a supported browser (served over HTTPS, or opened locally where the browser permits it).
2. On the **Radio** tab, connect **CAT** (Web Serial), your **PTT interface** (WebHID) if needed, and the **DE‑19 audio** device.
3. Check **Settings** for your CAT baud rate/framing, CI‑V addresses, PTT method, and audio device selection — defaults match a stock G90/DE‑19 setup.
4. Start the waterfall, tune your frequency, and transmit — either by holding the PTT button, sending an image/text as a waterfall picture, or keying a test carrier.

## Disclaimer

This tool directly keys a transmitter. Always verify your station is operating within your licence class, band plan, and power limits before transmitting, and keep an eye on the ALC/TX meter when sending images to avoid splatter.
