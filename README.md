# 💓 Webcam Heart Rate Estimation using rPPG

### Non-Contact Pulse Estimation with Computer Vision & Signal Processing

A webcam-based research prototype that estimates heart rate by analyzing subtle color variations in facial skin caused by blood circulation.

The project explores **remote photoplethysmography (rPPG)** — extracting pulse-related physiological information from ordinary video without requiring direct-contact sensors.

It was developed collaboratively as part of a **B.Tech final-year project at Symbiosis University of Applied Sciences in 2021**.

> **Research disclaimer:** This is an educational research prototype. It is not a medical device and should not be used for diagnosis, treatment, or clinical heart-rate monitoring.

---

## 🎯 Project Goal

Traditional heart-rate measurement usually requires physical contact through devices such as:

* ECG electrodes
* fingertip pulse sensors
* smartwatches
* chest straps

This project investigates whether an ordinary webcam can recover a pulse-related signal from facial video by detecting very small changes in skin-color intensity over time.

---

## 🧠 How It Works

The pipeline is:

```text
Webcam Video
     ↓
Face Detection
     ↓
Forehead Region of Interest
     ↓
Average Color Intensities
     ↓
Temporal Signal Buffer
     ↓
Moving-Average Smoothing
     ↓
Detrending
     ↓
Normalization
     ↓
FastICA
     ↓
FFT
     ↓
Physiological Frequency Range
     ↓
Dominant Frequency
     ↓
Heart-Rate Estimate (BPM)
```

---

## 📷 1. Webcam Capture

OpenCV continuously captures frames from the computer's default webcam.

```python
cv2.VideoCapture(0)
```

The system processes the incoming video in real time.

---

## 🙂 2. Face Detection

A Haar cascade classifier is used to identify a face in each frame.

The detected face is normalized to a consistent size before extracting a smaller forehead region.

The forehead was selected because exposed facial skin provides a useful region for measuring temporal changes in color intensity.

---

## 🎨 3. Color-Signal Extraction

For every processed frame, the system calculates the average intensity of the color channels within the forehead region.

This creates a temporal signal:

```text
Frame 1 → [B, G, R]
Frame 2 → [B, G, R]
Frame 3 → [B, G, R]
...
```

Although these variations are difficult to perceive visually, part of the temporal variation can contain a pulse-related component.

---

## 🔬 4. Signal Preprocessing

The raw color signals contain:

* illumination changes
* movement artifacts
* camera noise
* slow baseline drift
* non-physiological variation

The preprocessing pipeline applies:

### Moving-Average Smoothing

Reduces short-term noise in the raw channel signals.

### Detrending

Removes slow-moving baseline changes.

### Normalization

Centers and scales each signal before component separation.

---

## 🧩 5. Independent Component Analysis

The normalized color signals are passed through **FastICA**.

```text
Observed Color Signals
        ↓
     FastICA
        ↓
Independent Components
```

The goal is to separate mixed temporal signals into components that may contain more identifiable physiological periodicity.

---

## 🌊 6. Frequency-Domain Analysis

The selected ICA component is transformed from the time domain into the frequency domain using a **Fast Fourier Transform (FFT)**.

The implementation focuses on frequencies between approximately:

```text
0.8 Hz – 4.0 Hz
```

which correspond to:

```text
48 – 240 BPM
```

The strongest spectral peak within this range is selected as the estimated pulse frequency.

---

## ❤️ 7. Heart-Rate Estimation

The dominant frequency is converted to beats per minute:

```text
BPM = dominant_frequency × 60
```

Recent estimates are stored in a rolling buffer to produce a smoother displayed heart-rate value.

---

## 🏗️ Architecture

```text
┌─────────────────────┐
│       Webcam        │
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│   Face Detection    │
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│    Forehead ROI     │
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│ Color Intensities   │
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│ Signal Preprocessing│
│ Smooth / Detrend    │
│ Normalize           │
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│      FastICA        │
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│        FFT          │
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│ Dominant Frequency  │
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│   Estimated BPM     │
└─────────────────────┘
```

---

## 🛠️ Tech Stack

| Area                 | Technology           |
| -------------------- | -------------------- |
| Language             | Python               |
| Computer Vision      | OpenCV               |
| Numerical Processing | NumPy                |
| Signal Processing    | SciPy                |
| Component Separation | scikit-learn FastICA |
| Frequency Analysis   | FFT                  |
| Visualization        | Matplotlib           |
| Development          | Jupyter Notebook     |

---

## 📁 Repository Structure

```text
webcam-heart-rate-rppg/
│
├── docs/
│   ├── FinalProject_Report.pdf
│   ├── Major_Project_Synopsis.docx
│   └── SRS_Major_Project.pdf
│
├── webcam_rppg.ipynb
├── haarcascade_frontalface_default.xml
├── requirements.txt
└── README.md
```

---

## 🚀 Getting Started

### 1. Clone

```bash
git clone https://github.com/Gravity-2010/webcam-heart-rate-rppg.git
cd webcam-heart-rate-rppg
```

### 2. Create a virtual environment

```bash
python -m venv .venv
```

Linux/macOS:

```bash
source .venv/bin/activate
```

Windows:

```bash
.venv\Scripts\activate
```

### 3. Install dependencies

```bash
pip install -r requirements.txt
```

### 4. Start Jupyter

```bash
jupyter notebook
```

Open:

```text
webcam_rppg.ipynb
```

and run the notebook.

A connected webcam is required.

---

## ⚠️ Experimental Conditions

rPPG performance can be affected substantially by environmental conditions.

For more stable measurements:

* Keep the face relatively still.
* Use consistent frontal lighting.
* Avoid strong shadows.
* Avoid rapidly changing illumination.
* Keep the face visible to the camera.
* Allow enough frames to accumulate before interpreting the estimate.

---

## ⚠️ Limitations

This is an exploratory prototype rather than a validated heart-rate monitor.

Important limitations include:

### Motion Sensitivity

Head movement can introduce color and spatial variations much larger than the pulse signal.

### Lighting Sensitivity

Changes in room lighting can alter RGB intensity and interfere with pulse extraction.

### Camera Characteristics

Frame rate, compression, exposure control, and sensor quality affect signal quality.

### Face Detection

Haar-cascade detection can become unstable under changes in pose, occlusion, or lighting.

### Fixed Forehead ROI

The implementation uses a simple fixed forehead region after face normalization rather than advanced facial landmark tracking.

### No Clinical Validation

The repository does not provide a controlled validation study against a medical-grade ECG or pulse-oximeter reference.

Therefore, estimated BPM values should be treated as experimental outputs rather than clinical measurements.

---

## 🔮 Possible Improvements

Potential improvements include:

* Facial-landmark-based ROI tracking
* Motion compensation
* Adaptive ROI selection
* Stronger signal-quality metrics
* More robust spectral filtering
* POS or CHROM rPPG algorithms
* Real-time signal-quality feedback
* Controlled comparison with reference heart-rate sensors
* Evaluation across different lighting conditions
* Evaluation across multiple participants
* Packaging the notebook pipeline into a standalone application

---

## 📄 Project Documentation

Additional original project documentation is available under:

```text
docs/
```

including the final report, project synopsis, and software-requirements documentation.

---

## 👥 Project Context

Developed collaboratively as a **B.Tech final-year project in 2021**.

The project provided early hands-on experience with:

* computer vision
* biomedical signal processing
* frequency-domain analysis
* physiological sensing
* real-time webcam processing

---

## 📌 Repository Status

**Archived completed academic project**

This repository is preserved as an earlier exploration of non-contact physiological sensing and computer-vision-based signal processing.
