# 🎚️ Gesture-Based Equalizer  
### Control your system audio with hand gestures — powered by MediaPipe & PyQt5  

A full-featured **10-band equalizer** that responds to **hand gestures** captured via webcam.  
It integrates with **Equalizer APO** for system-wide audio control and features an elegant GUI with a live camera feed, preset saving, smooth slider animations, and gesture-based band control.

---

## 🧠 Overview

**Gesture-Based EQ** lets you:
- Adjust music frequencies with your hands.
- Save, apply, and randomize presets.
- Visually control your equalizer in real-time with smooth transitions.
- Use your webcam as a gesture sensor — no additional hardware needed.

---

## ⚙️ Requirements

### 1. Install **Equalizer APO**  
You’ll need Equalizer APO for system-wide sound control.  
Download and install from:  
🔗 **[Equalizer APO on SourceForge](https://sourceforge.net/projects/equalizerapo/)**  

During setup:
- Select your **playback device** (e.g., headphones, speakers).  
- Restart your PC once installation is complete.

---

### 2. Install **Python** and Dependencies  
Make sure Python 3.8+ is installed, then open a terminal and run:

```bash
pip install PyQt5 mediapipe opencv-python numpy
```

Or install everything at once:
```bash
pip install -r requirements.txt
```

---

## 🚀 How to Run

Clone this repository and start the equalizer:

```bash
git clone https://github.com/mukndd/gesture-based-eq.git
cd gesture-based-eq
python gesture_eq.py
```

Allow camera access when prompted.  
Once launched, you’ll see:
- A 10-band EQ with live sliders.
- A live camera feed inside the GUI.
- Control buttons for presets and system actions.

---

## 🖥️ GUI Controls

| Button | Description |
|--------|--------------|
| **Save Current Preset** | Saves your current slider positions. |
| **Apply Saved Preset** | Smoothly animates all sliders to your saved values (“Magic Mode”). |
| **Randomize** | Randomly changes all bands between -20 dB and +20 dB with smooth motion. |
| **Set to Default** | Resets all sliders to 0 dB (neutral sound). |

---

## ✋ Gesture Controls

| Hand | Gesture | Function | Notes |
|------|----------|-----------|-------|
| **Left Hand** | (1), (1,2), (1,2,3), (1,2,3,4), (1,2,3,4,5), (5), (1,5), (1,2,5), (1,2,3,5), () | Selects 10 EQ bands from Sub-Bass → Air | () = Fist |
| **Left Hand (1,4)** | Hold for 0.8s | Save current preset & exit | “Keep” exit |
| **Left Hand (1,4,5)** | Hold for 0.8s | Reset sliders to default & exit | “Reset” exit |
| **Right Hand** | Pinch (index + thumb) | Adjust selected band up/down | Smooth movement |
| **Right Hand (1,5)** | “L” shape hold | Lock / unlock selected band | Prevent accidental changes |

🧩 **Finger numbering:**  
Thumb = 5, Index = 1, Middle = 2, Ring = 3, Pinky = 4

---

## 🧩 How It Works

- **MediaPipe** detects your hand and tracks fingers in real time.  
- **PyQt5** handles the GUI, camera embedding, and animation.  
- **OpenCV** processes the webcam feed.  
- **Equalizer APO** applies real audio EQ changes at the system level.  

Your hand gestures select and adjust each frequency band from **62 Hz (Sub-Bass)** to **16 kHz (Air)**.

---

## 🔄 Restoring Audio Defaults

If you want to remove Equalizer APO or reset your sound:
1. Open the **Equalizer APO Configurator**.
2. Deselect your playback device.
3. Restart your PC — your audio returns to normal.

---

## 💡 Troubleshooting

- **APO Write Failed (Permission Denied):**  
  Run Python or the `.exe` as **Administrator** — needed for writing to `C:\Program Files\EqualizerAPO`.  
  The equalizer will still visually update even if APO writing fails.  

- **No Camera Feed:**  
  Make sure no other program is using your webcam.  
  Restart the app and check your camera permissions.

- **No Effect on Audio:**  
  Reopen **Equalizer APO Configurator**, make sure your output device is *checked* and restart your PC.

---

## 📦 Files in This Repo

| File | Purpose |
|------|----------|
| `gesture_eq.py` | Main equalizer and gesture control program |
| `eq_state.json` | Stores real-time slider and lock state |
| `saved_preset.json` | Stores your saved preset for reuse |
| `requirements.txt` | Quick install of all dependencies |
| `README.md` | Documentation and setup guide |

---

## 🧰 Tech Stack

**Languages:** Python 3  
**Libraries:**  
- PyQt5  
- OpenCV  
- MediaPipe  
- NumPy  

---

## 🪄 License

MIT License © 2025 Mukund G  
Feel free to use, fork, or contribute.  

If this project helped you, a ⭐ on the repo or a share is always appreciated.
