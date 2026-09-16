# Gesture-Based System Equalizer

A webcam, your hand, and a 10-band system-wide audio equalizer — no plugin, no mixer hardware, just [MediaPipe](https://github.com/google-ai-edge/mediapipe) hand tracking driving [Equalizer APO](https://sourceforge.net/projects/equalizerapo/) directly.

![The Gesture Based Equalizer](./The%20Gesture%20based%20Equalizer.png)

> The image above is a title card, not a screenshot of the app running. A real screenshot/GIF of the camera view + live sliders is the next thing to capture — see [Status](#status--limitations).

## What it does

Point a normal webcam at your hand and control a real, system-wide 10-band equalizer on Windows:

- **Left hand** — hold up a finger-count combination to select one of 10 EQ bands (Sub-Bass through Air).
- **Right hand** — pinch (thumb + index finger together) and move up/down to raise or lower the gain (±24 dB) on whichever band is selected. An "L-shape" hold (thumb and index spread apart) locks a band so it stops responding to pinch.
- **Left hand, held gestures** — thumb+pinky held ~0.8s exits and keeps the current EQ; thumb+pinky+index held ~0.8s exits and resets every band back to 0 dB.

The desktop window also shows a conventional slider for every band (mouse-adjustable, not gesture-only) and a live camera preview with the hand skeleton drawn on it, plus buttons to save/recall a preset, randomize, or reset to flat.

Every gain change gets written straight into Equalizer APO's `config.txt` as real parametric filter lines, so it's actually shaping system audio output in real time, not just a UI mockup.

## Who this is for

Built as a personal tool and as a concrete demonstration of turning a noisy computer-vision signal into a stable, real-time control surface for a real piece of external software. Useful to anyone who wants touch-free EQ control on Windows, and as a code sample for evaluating applied CV / desktop-app engineering.

## Role

Sole builder — solo project, no collaborators. (Verified from this repo's own commit history: every commit is authored under one account.)

## How it works (architecture)

```
Webcam ──▶ OpenCV (cv2.VideoCapture, 640×360)
              │
              ▼
     MediaPipe Hands (up to 2 hands, 21 landmarks each)
              │
              ▼
   Gesture decoding (pure landmark geometry, no ML classifier)
     • fingers_up(): compares each fingertip's y to its knuckle's y
       (and thumb tip's x to its joint's x) to get an extended-finger set
     • LEFT hand finger-set  → GESTURE_TO_BAND lookup → selects 1 of 10 bands
     • RIGHT hand thumb–index distance → pinch detection → gain adjustment
              │
              ▼
   State machine (runs on a background QThread, separate from the GUI thread)
     • hold-durations + a cooldown before any gesture "commits"
       (rejects single-frame misreads)
     • exponential smoothing (α = 0.6) on the live gain value
       (rejects per-frame jitter in landmark position)
              │
              ├──▶ PyQt5 GUI: sliders, camera preview, info/preset labels
              │      (Qt signals cross from the worker thread to the GUI thread)
              │
              └──▶ Equalizer APO config.txt (10 parametric filter lines)
                     → Equalizer APO applies this system-wide, to all Windows audio
```

Everything here is software; the only physical component is a standard off-the-shelf webcam. There is no custom hardware, microcontroller, or wiring involved in this particular project.

## Verified stack

Confirmed by reading the actual source and by installing the exact pinned requirements into a clean virtual environment this pass:

- Python 3
- [OpenCV](https://opencv.org/) (`opencv-python`) — camera capture, frame handling
- [MediaPipe](https://github.com/google-ai-edge/mediapipe) `Hands` — hand landmark extraction
- [PyQt5](https://www.riverbankcomputing.com/software/pyqt/) — desktop GUI, background worker thread, cross-thread signals
- [NumPy](https://numpy.org/)
- [Equalizer APO](https://sourceforge.net/projects/equalizerapo/) — the actual system-wide EQ engine this project writes into. **Not bundled** — install it separately before running.

## Setup

```bash
git clone https://github.com/mukndd/gesture-based-eq.git
cd gesture-based-eq
pip install -r requirements.txt
```

1. Install [Equalizer APO](https://sourceforge.net/projects/equalizerapo/) on Windows and select your playback device during its setup.
2. Confirm its config path matches `C:\Program Files\EqualizerAPO\config\config.txt` — that path is currently hardcoded in `gesture_eq.py`; edit `EQUALIZER_APO_CONFIG` at the top of the file if your install location differs.
3. Run it:

```bash
python gesture_eq.py
```

**Verified this pass:** `pip install -r requirements.txt` into a fresh venv (Python 3.12) installs cleanly, `gesture_eq.py` compiles, and all four dependencies (`PyQt5`, `mediapipe`, `cv2`, `numpy`) import successfully. The end-to-end gesture-tracking runtime (camera + live hand tracking + APO write) was not re-exercised in that pass — it relies on the state machine described above, which is what's actually in the shipped source.

## The hardest problem here

Turning noisy, continuous hand-landmark coordinates into 10 discrete, stable band selections plus a smooth continuous gain value — without a canned gesture-recognition library and without visible jitter in the output.

The approach:
- **Selection is categorical, not continuous.** A finger-count combination is far more robust to per-frame landmark noise than trying to read a continuous position, so band selection uses discrete gesture matching instead of a fuzzy classifier.
- **Every state change requires a hold, not an instant match.** Pinch-adjust needs ~120ms sustained before it engages; the lock gesture needs ~280ms; the two exit gestures need ~800ms. This trades a small amount of latency for rejecting transient misreads.
- **The continuous value itself is smoothed**, not applied raw — each new pinch position nudges the gain toward its target (`α = 0.6`) rather than snapping to it, which is what keeps the EQ from visibly jittering while your hand naturally shakes.

## Known limitations / failure modes

Read directly out of the current source, not aspirational:

- The Equalizer APO config path is hardcoded, not configurable from the UI or an environment variable. If APO isn't installed at that exact path, the write silently fails to a console `print` — the GUI gives no visible error.
- The webcam index is hardcoded to `0` with no picker and no handling for a missing/busy camera beyond an infinite retry loop.
- Gesture recognition is pure landmark geometry (finger-tip-vs-knuckle comparisons), not a trained classifier — it can misread hands at extreme angles, in low light, or wearing gloves. There's no per-user calibration step.
- Mouse-driven sliders and gesture control write to the same shared state with no visual distinction between the two input sources beyond the info label.
- No automated tests exist for the gesture logic.

## Status & limitations

**Live/current (this repo):** the 10-band gesture EQ described above — one-handed selection, one-handed pinch adjust, lock, save/recall preset, randomize, reset. Confirmed working by source inspection and a clean dependency install.

**In progress, unreleased:** a second-generation version exists only as a local prototype and adds two-handed control (right hand drives the EQ via an on-screen slider overlay, left hand controls Spotify playback/volume/seek) plus visual slider overlays drawn directly onto the camera feed. It is **not published** and is not ready to be, because:
- it currently hardcodes a real Spotify API client ID/secret directly in the source — that must move to environment variables (or a local, gitignored config file) before this can ever be committed;
- using it requires the end user to register their own Spotify Developer app, configure a redirect URI, complete a browser OAuth flow, and hold a Spotify **Premium** subscription (the Spotify Web API's playback-control endpoints require Premium) — a meaningfully higher setup bar than the current version's plain `pip install`;
- there's no setup documentation for any of that yet.

None of this changes what's actually live: what's in this repository today is the fully self-contained gesture EQ, with no external accounts or API keys required.

**Missing before this is fully "recruiter-proof":** a real screenshot or short screen recording of the app actually running (camera feed + live sliders responding to a hand) — the current image in this README is a title card, not proof of it working.

## Next steps

- Capture a real screenshot/GIF of the running app.
- Make the Equalizer APO path and camera index configurable instead of hardcoded.
- Before ever publishing the two-handed/Spotify version: move its credentials to environment variables and write setup docs for the Spotify Developer app registration + OAuth flow.

## What this demonstrates

A real-time computer-vision input pipeline (camera → landmarks → decoded gesture) wired into a responsive desktop GUI across threads (Qt signals/slots bridging a background worker thread and the UI thread), and — the part that doesn't show up in a typical "I called an API" project — driving a real third-party system through its actual file-based configuration interface (Equalizer APO's filter syntax) rather than through a convenient SDK. The core engineering problem solved is turning an inherently noisy sensor stream into something usable and stable in real time, using hold-durations and smoothing rather than a pre-built gesture library.

## License

MIT — see [LICENSE](./LICENSE).
