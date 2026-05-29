# 🚗 Driver Drowsiness Detector

A real-time driver drowsiness detection system built with **pure OpenCV** — no external model downloads required. Detects eye closure, yawning, and head nodding using Haar cascade classifiers bundled with every `opencv-python` install.

---

## Features

- **Eye Aspect Ratio (EAR)** — detects sustained eye closure
- **PERCLOS** — rolling % of time eyes are closed over a ~10-second window
- **Yawn detection** — via smile cascade on the mouth region
- **Head nod detection** — tracks face bounding-box vertical shift over time
- **Audio alarm** — looping beep via `pygame` (optional)
- **Live HUD** — gauges, session stats, FPS counter, status badge
- **Three run modes** — webcam, video file, or headless demo

---

## Requirements

```bash
pip install opencv-python scipy pygame
````

> `pygame` is optional — the detector runs without it (no audio alarm).

---

## Usage

```bash
# Webcam (default device 0)
python drowsiness_detector.py

# Specific webcam index
python drowsiness_detector.py 1

# Video file
python drowsiness_detector.py dashcam_clip.mp4

# Headless demo (no webcam needed — writes drowsy_demo_out.mp4)
python drowsiness_detector.py --demo
```

### Keyboard Controls (webcam / video mode)

| Key | Action |
|-----|--------|
| `Q` | Quit |
| `R` | Reset all counters |
| `S` | Save snapshot as JPEG |
| `+` / `-` | Raise / lower EAR sensitivity |

---

## How It Works

### Eye Aspect Ratio (EAR)

The eye bounding box returned by the Haar eye cascade gets taller when the eye is open and flatter when closed. EAR is approximated as:

```
EAR = (box_height / box_width) × 0.70
```

When EAR drops below `EAR_THRESH` (default **0.22**) for `EAR_CONSEC_FRAMES` (default **20**) consecutive frames, the drowsiness alarm fires.

### PERCLOS

A rolling 150-frame buffer tracks what fraction of recent frames had closed eyes. If it exceeds `PERCLOS_ALERT` (default **0.35**, i.e. 35 %), the alarm fires independently of the frame counter.

### Head Nod

The vertical position of the face bounding-box centre is tracked over 40 frames. A drop greater than `HEAD_NOD_THRESH` (default **0.08** × frame height) relative to the recent baseline triggers the alarm.

### Yawn Detection

OpenCV's smile cascade is applied to the bottom 45 % of the face ROI. A sustained detection lasting `YAWN_CONSEC` (default **18**) frames counts as one yawn in the session log.

---

## Tuning

All thresholds are constants at the top of the script and can also be adjusted live with `+` / `-`:

| Constant | Default | Effect |
|---|---|---|
| `EAR_THRESH` | `0.22` | Lower = more sensitive to closure |
| `EAR_CONSEC_FRAMES` | `20` | Fewer frames = faster alarm |
| `PERCLOS_ALERT` | `0.35` | Lower = alarm on less eye closure |
| `HEAD_NOD_THRESH` | `0.08` | Lower = more sensitive to nodding |
| `YAWN_CONSEC` | `18` | Fewer frames = yawn counted sooner |

---

## Project Structure

```
drowsiness-detector/
├── drowsiness_detector.py   # Main script (all modes)
├── README.md
├── .gitignore
└── requirements.txt
```

---

## Limitations & Notes

- Haar cascades work best under **even frontal lighting**. Side profiles or strong backlighting reduce detection reliability.
- The EAR approximation from bounding boxes is less precise than landmark-based methods (e.g. dlib or MediaPipe). Use `+`/`-` to calibrate to your camera and lighting.
- Audio alarm requires a working audio device and `pygame`.

---

## License

MIT — do whatever you like, just don't fall asleep at the wheel.
