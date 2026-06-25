# 🎯 YOLOv8 Real-Time Weapon Detection System

> An AI-powered object detection microservice built with YOLOv8 and Flask, integrated into a full Symfony live streaming platform — automatically terminating streams when dangerous objects are detected.

![Python](https://img.shields.io/badge/Python-3.10-blue?style=flat-square&logo=python)
![YOLOv8](https://img.shields.io/badge/YOLOv8-Ultralytics-purple?style=flat-square)
![Flask](https://img.shields.io/badge/Flask-2.x-black?style=flat-square&logo=flask)
![OpenCV](https://img.shields.io/badge/OpenCV-4.x-green?style=flat-square&logo=opencv)
![Symfony](https://img.shields.io/badge/Symfony-6.x-000000?style=flat-square&logo=symfony)
![License](https://img.shields.io/badge/License-MIT-yellow?style=flat-square)

---

## 📌 Project Overview

This project is composed of two parts:

1. **A Python/Flask detection microservice** (`weapon_detector_api.py`) that receives video frames and uses YOLOv8n to detect dangerous objects in real time.
2. **A Symfony web platform** (EyeTwin) where coaches can go live via OBS or webcam — the stream is automatically terminated if a dangerous object is detected.

The system captures a frame every **5 seconds**, sends it to the Flask API, and if a dangerous object is detected with confidence > 50%, the live stream is immediately ended and an alert is shown to the coach.

---

## 🚨 Detected Dangerous Objects

| Object | Object | Object |
|--------|--------|--------|
| knife | gun | rifle |
| pistol | scissors | sword |
| baseball bat | bottle | fork |

---

## 📊 Detection Results (Sample)

| Object | Confidence |
|--------|------------|
| Bus | 87.3% |
| Person | 86.6% |
| Person | 85.3% |
| Person | 82.5% |
| Stop Sign | 25.5% |

- ✅ **Total objects detected:** 6
- ✅ **Average confidence:** 65.5%
- ✅ **Best detection:** Bus (87.3%)
- ✅ **Avg inference speed:** ~291ms/frame on CPU

---

## 🏗️ Architecture

```
Browser / OBS
      │
      │  (frame every 5s via canvas.toDataURL)
      ▼
┌─────────────────────┐
│   Symfony Backend   │  ← PHP, Doctrine ORM, CSRF protection
│  /api/moderation/   │
│    check/{id}       │
└────────┬────────────┘
         │  POST /detect  (base64 JPEG)
         ▼
┌─────────────────────┐
│   Flask API         │  ← Python, port 5001
│  weapon_detector    │
│  _api.py            │
└────────┬────────────┘
         │
         ▼
┌─────────────────────┐
│   YOLOv8n Model     │  ← yolov8n.pt (COCO 80 classes)
│   + OpenCV/NumPy    │
└─────────────────────┘
         │
         ▼
  { dangerous: true,
    dangerous_objects: [...],
    all_detections: [...] }
         │
         ▼
  Symfony ends the stream
  → Banner shown to coach
  → Redirect after 4s
```

---

## 📁 Project Structure

```
yolov8-object-detection/
├── objectdetect.ipynb        # Jupyter notebook — initial detection experiments
├── weapon_detector_api.py    # Flask REST API — weapon detection endpoint
├── yolov8n.pt                # YOLOv8 nano model weights (not tracked in git)
├── detection_report.png      # Results visualization (bar + pie chart)
├── .gitignore                # Excludes model weights & checkpoints
└── README.md                 # Project documentation
```

---

## 🛠️ Tech Stack

| Layer | Tool | Purpose |
|-------|------|---------|
| AI Model | YOLOv8n (Ultralytics) | Object detection — 80 COCO classes |
| CV Processing | OpenCV + NumPy | Frame decoding, image processing |
| Detection API | Python / Flask | REST microservice on port 5001 |
| Web Platform | Symfony 6 (PHP) | Full MVC web app, Doctrine ORM |
| Frontend | Twig + Vanilla JS | UI, webcam capture, HLS playback |
| Live Streaming | HLS.js | OBS → RTMP → HLS stream playback |
| Webcam Live | WebRTC / getUserMedia | Browser-native streaming, no OBS needed |
| Notebook | Jupyter | Initial experiments & visualization |

---

## 🚀 How to Run

### 1. Clone the repository
```bash
git clone https://github.com/imenhammami12/yolov8-object-detection.git
cd yolov8-object-detection
```

### 2. Create and activate environment
```bash
conda create -n yolov8 python=3.10 -y
conda activate yolov8
```

### 3. Install dependencies
```bash
pip install ultralytics flask opencv-python numpy jupyter matplotlib pandas
```

### 4. Download model weights
```bash
# yolov8n.pt is downloaded automatically on first run
# Or manually place yolov8n.pt in the project root
```

### 5. Start the Flask detection API
```bash
python weapon_detector_api.py
```
You should see:
```
✅ Starting Weapon Detection API on port 5001...
 * Running on http://0.0.0.0:5001
```

### 6. (Optional) Launch Jupyter for experiments
```bash
jupyter notebook
# Open objectdetect.ipynb
```

---

## 🔌 API Reference

### `POST /detect`
Receives a base64-encoded JPEG frame and returns detection results.

**Request body:**
```json
{
  "image": "<base64-encoded JPEG string>"
}
```

**Response:**
```json
{
  "dangerous": true,
  "dangerous_objects": [
    { "label": "knife", "confidence": 0.872 }
  ],
  "all_detections": [
    { "label": "person", "confidence": 0.941 },
    { "label": "knife", "confidence": 0.872 }
  ]
}
```

### `GET /health`
Returns the API status.
```json
{ "status": "ok", "model": "yolov8n" }
```

---

## ⚙️ How the Symfony Integration Works

1. The coach starts a live stream (OBS or webcam)
2. Every 5 seconds, the frontend captures a frame using `canvas.drawImage()`
3. The frame is base64-encoded and POSTed to `/live/api/moderation/check/{id}`
4. Symfony forwards it to the Flask API at `http://localhost:5001/detect`
5. If `dangerous: true` → Symfony sets stream status to `ended`, saves `endedAt`
6. The frontend receives `action: "stream_ended"` → shows moderation banner → redirects

---

## 🔮 Next Steps

- [ ] Train a custom dataset on Roboflow for better weapon-specific accuracy
- [ ] Replace YOLOv8n with YOLOv8s/m for higher confidence scores
- [ ] Add email/SMS notification to coach on detection
- [ ] Deploy Flask API with Gunicorn + Nginx in production
- [ ] Add detection history log per stream

---

## 👩‍💻 Author

**Imen Hammami**
- GitHub: [@imenhammami12](https://github.com/imenhammami12)
- Email: imen.hammami@esprit.tn
- Location: Tunisia 🇹🇳

---

## 📄 License

This project is licensed under the MIT License.
