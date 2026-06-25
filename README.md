# Reticle — Probe XY Offset Calibrator

A tiny, single-file web tool that measures the **XY offset between your nozzle and an inductive bed probe** using any webcam — no toolchanger, no extra electronics, no install. Point a camera up at the toolhead, line each one up on the on-screen reticle, read the machine coordinates off your printer, and the tool spits out a ready-to-paste Klipper `[probe]` block.

![License: MIT](https://img.shields.io/badge/License-MIT-green.svg)

> Inspired by the camera-assisted alignment idea behind toolchanger calibration tools, but aimed at the much more common job: finding `x_offset` / `y_offset` for a bed probe.

---

## Why

The XY offset between an inductive probe and the nozzle is usually measured with calipers, which gets you to maybe ±0.5–1&nbsp;mm. That's enough to make a bed mesh land slightly off where you think it is. A camera fixed in space gives you a repeatable reference point, so you're comparing two real machine positions instead of eyeballing a ruler.

## What it does

- Live webcam feed with a precise reticle overlay (drag to place, arrow keys to nudge, **lock** so you can't bump it).
- Zoom that **pivots on the reticle** so you can magnify the exact alignment point.
- **Multi-sample averaging** — capture each target a few times; it averages and shows you the spread, so you see your real uncertainty instead of trusting one shaky reading.
- Computes the offset using Klipper's actual sign convention and gives you copy-paste config.
- Fully self-contained: one HTML file, zero external scripts, fonts, or network calls.

## Usage

1. Aim the webcam **straight up at the nozzle**, focus, and zoom in.
2. Drag the reticle onto a spot near center and hit **Lock reticle**. From here, only jog the printer — never move the reticle.
3. Jog so the **nozzle tip** sits on the reticle. Read X/Y from Klipper's `GET_POSITION` (the `toolhead:` line), type it in, **Add nozzle sample**. Repeat 2–3× for a tighter average.
4. Jog so the **center of the sensor face** sits on the reticle. Type that X/Y, **Add probe sample**.
5. Copy the resulting `[probe]` block into your `printer.cfg` and `RESTART`.

## The sign convention (important)

Klipper defines the probe offset as **nozzle position − probe position**:

```
x_offset = nozzle_x − probe_x
y_offset = nozzle_y − probe_y
```

This tool computes it that way. Note this is *opposite* to how some other firmwares express it, so if you're porting a number from Marlin, expect the signs to flip. If your first mesh after calibrating comes out mirrored or shifted toward one corner, flip a sign and re-test — that's the classic tell.

## Tips for an accurate reading

- **Match the Z height.** Bring the nozzle and the sensor face to roughly the *same* Z when you center each one. A camera looking up at any angle introduces parallax, and a height difference turns that into XY error.
- **Mark the sensor center.** A nozzle tip is a sharp point; a round sensor face is not. A paint-pen dot on the dead center of the sensor makes alignment far less of a guess.
- **Take a few samples.** Centering by eye scatters a little. The averaging + spread readout tells you how repeatable your alignment actually was.

## What it deliberately doesn't do

No automatic circle-detection to "find" the sensor center. An inductive sensor's true sensing center isn't perfectly concentric with its visible plastic rim, so sub-pixel edge detection would be false precision. Multi-sample averaging gets you to ~0.05&nbsp;mm, which is already below what the probe's own trigger repeatability can make use of.

This tool also only handles **XY**. For `z_offset`, use Klipper's built-in `PROBE_CALIBRATE`.

## Running it

**Easiest:** just open `index.html` in a browser.

**If the camera is blocked:** browsers only grant webcam access over `https://` or `http://localhost`. Serve it locally:

```bash
python3 -m http.server
# then open http://localhost:8000/
```

**Deploy on GitHub Pages:**

Settings → Pages → Build and deployment → Source: **Deploy from a branch** → Branch: **main** / **/(root)** → Save. Because the tool is named `index.html`, it loads directly at your Pages URL — and Pages is served over HTTPS, so the webcam works with no extra steps.

## Browser support

Needs a browser with `getUserMedia` and `ResizeObserver` (any current Chrome, Firefox, Edge, or Safari). Works on mobile too, though a laptop is usually easier for reading the printer's coordinates side-by-side.

## Contributing

Issues and PRs welcome. It's intentionally a single file with no build step — keep it that way unless there's a strong reason not to.

## License

[MIT](LICENSE) © 2026 CephalonShade
