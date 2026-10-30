![SIH 2026](https://img.shields.io/badge/SIH%202026-Rank%2013%2F150%20(Top%209%25)-blue)
![Python](https://img.shields.io/badge/Python-3.11+-green)
![YOLOv8](https://img.shields.io/badge/YOLOv8-mAP%400.5%3D0.75-orange)
![FastAPI](https://img.shields.io/badge/FastAPI-Backend-teal)
![React](https://img.shields.io/badge/React-Dashboard-blue)
![License](https://img.shields.io/badge/License-MIT-yellow)

# 🚌 AI-Powered Mobile Urban Intelligence Platform

> Transforming public transport buses into mobile urban sensing units using edge AI + centralized analytics.

**Problem Statement ID:** 26124  
**Domain:** Smart Cities / Urban Mobility / Computer Vision  
**SIH 2026 Internal Hackathon — Ranked 13th out of 150 teams (Top 9%)**

---

## 📖 Overview

Urban public transport buses traverse every major road daily and increasingly carry multiple cameras. This project turns them into **intelligent sensing platforms** that detect:

- 🕳️ **Road defects** — potholes, cracks, damaged surfaces
- 🚧 **Missing infrastructure** — dividers, zebra crossings, traffic signs
- 🚶 **Vulnerable pedestrians** — school zones, crossing risks
- 🚗 **Traffic incidents** — hit-and-run, rash driving (with ANPR)
- 🚦 **Congestion** — vehicle density, traffic bottlenecks

Data is processed **on the edge** (onboard the bus) and only lightweight metadata is sent to a **central command center** — achieving **99% bandwidth reduction** over raw video transmission.

---

## ✨ Key Features

### Onboard Edge AI
- **Multi-model inference:** YOLOv8 (general) + custom-trained pothole detector
- **Custom pothole model:** mAP@0.5 = **0.75**, Precision = **0.82**, Recall = **0.67**
- **Trained via transfer learning** on annotated Indian road dataset
- **Multi-class detection:** potholes, pedestrians, vehicles, missing infrastructure
- **ByteTrack** for deduplication across frames
- **Offline-first:** events buffer in SQLite when connectivity drops
- **Low bandwidth:** ~2 KB JSON/event vs ~50 MB video clip

### Central Command Center
- **REST API** (FastAPI) with OpenAPI/Swagger docs
- **WebSocket** live event streaming
- **8-page dashboard** (React + Vite + Leaflet):
  - Command Center (KPIs + live feed)
  - Live GIS Map (event markers, clusters)
  - Traffic Analytics (speed trends)
  - Road Conditions (hazard distribution)
  - Incidents (ANPR, dispatch actions)
  - Fleet Monitoring (real-time bus tracker)
  - Evidence Vault (frame snapshots)
  - Admin (simulator controls, diagnostics)
- **ANPR module** for hit-and-run incidents
- **Simulator** for scale testing & demo

---

## 🏗️ Architecture

```text
┌─────────────────────────────────────────────────────────────┐
│                    EDGE AI (Onboard Bus)                     │
│                                                              │
│   Video Feed ──▶ YOLOv8 Model ──▶ Event Generator           │
│   (4 cameras)    (multi-task)     + GPS + Timestamp         │
│                                         │                    │
│                          ┌──────────────┘                    │
│                          ▼                                   │
│              ┌───────────────────────┐                       │
│              │ Offline Buffer (SQLite)│ (network fallback)   │
│              └───────────┬───────────┘                       │
└──────────────────────────┼───────────────────────────────────┘
                           │ ~2 KB JSON per event
                           ▼
┌─────────────────────────────────────────────────────────────┐
│              CENTRAL COMMAND CENTER (Cloud)                  │
│                                                              │
│   FastAPI ──▶ SQLite DB ──▶ React Dashboard                 │
│   Ingestion   (PostGIS-     + Leaflet GIS                   │
│   + WebSocket  ready)       + Analytics                     │
└─────────────────────────────────────────────────────────────┘
```

---

## 🛠️ Tech Stack

| Layer | Technology |
|---|---|
| **Edge AI** | Python, YOLOv8 (Ultralytics), OpenCV, ByteTrack, SQLite |
| **Backend** | FastAPI, SQLAlchemy, Pydantic, Uvicorn, WebSocket |
| **Frontend** | React, Vite, Leaflet, Recharts |
| **Database** | SQLite (PostGIS-ready for production) |
| **Deployment** | Docker, docker-compose |

---

## 📂 Project Structure

```text
mobile_urban_intelligence/
├── edge/                       # Onboard AI pipeline
│   ├── detector.py             # YOLOv8 multi-model inference
│   ├── tracker.py              # ByteTrack deduplication
│   ├── gps_simulator.py        # GPS route waypoints
│   ├── event_engine.py         # Event schema + severity
│   ├── uploader.py             # HTTP + offline buffer
│   ├── pipeline.py             # Frame orchestration
│   ├── run.py                  # CLI entry point
│   ├── config.py               # Thresholds + paths
│   ├── models/                 # Trained .pt weights
│   └── data/                   # Video inputs (gitignored)
│
├── backend/                    # Central command API
│   ├── app/
│   │   ├── main.py             # FastAPI app
│   │   ├── api/                # Route handlers
│   │   ├── db/                 # SQLAlchemy models
│   │   ├── services/           # Business logic
│   │   ├── ai/                 # Server-side AI hooks
│   │   └── sim/                # Fleet simulator
│   └── evidence/               # Uploaded snapshots
│
├── frontend/                   # React dashboard
│   ├── src/
│   │   ├── pages/              # 8 dashboard pages
│   │   ├── components/         # Reusable UI
│   │   └── services/           # Backend client
│   └── package.json
│
├── docker-compose.yml
└── README.md
```

---

## 🚀 Quick Start

### Prerequisites
- Python 3.11+ (3.12 recommended)
- Node.js 18+
- (Optional) CUDA-compatible GPU for faster inference

### 1. Clone & Setup

```bash
git clone https://github.com/RakeshChoudhary49/mobile-urban-intelligence.git
cd mobile-urban-intelligence

# Backend
cd backend
python -m venv .venv
.venv\Scripts\activate          # Windows
pip install -r requirements.txt

# Edge
cd ../edge
pip install -r requirements.txt

# Frontend
cd ../frontend
npm install
```

### 2. Start the Central Server

```bash
cd backend
uvicorn app.main:app --reload --host 127.0.0.1 --port 8000
```

API docs: **http://127.0.0.1:8000/docs**

### 3. Start the Dashboard

```bash
cd frontend
npm run dev
```

Dashboard: **http://localhost:5173**

### 4. Run Edge AI on a Single Bus

```bash
cd edge
python run.py --video "data/bus_01.mp4" --bus-code BUS-101 --api-url http://127.0.0.1:8000
```

### 5. Run the Fleet (4 buses in parallel)

```bash
# Terminal 1
python run.py --video "data/bus_01.mp4" --bus-code BUS-101

# Terminal 2
python run.py --video "data/bus_02.mp4" --bus-code BUS-102

# Terminal 3
python run.py --video "data/bus_03.mp4" --bus-code BUS-103

# Terminal 4
python run.py --video "data/bus_04.mp4" --bus-code BUS-104
```

Watch all 4 buses stream events to one dashboard in real time.

---

## 📊 Model Performance

| Metric | Value |
|---|---|
| **mAP@0.5** | 0.75 |
| **mAP@0.5:0.95** | 0.426 |
| **Precision** | 0.82 |
| **Recall** | 0.67 |
| **Inference Speed** | ~38 ms/frame (CPU) |
| **Training** | YOLOv8n, 100 epochs, transfer learning |
| **Dataset** | Custom annotated Indian road dataset + RDD2022 |

---

## 🎯 Key Achievements

- ✅ **Top 9%** at SIH 2026 Internal Hackathon (13th / 150 teams)
- ✅ **99% bandwidth reduction** via edge processing
- ✅ **Offline-resilient** event delivery (validated during network outage)
- ✅ **4-bus fleet** streaming to single dashboard
- ✅ **ANPR** with 94% OCR confidence on incident demo
- ✅ Fully documented OpenAPI spec (`/docs`)

---

## 🔮 Future Work

- [ ] Merge full RDD2022 dataset → target mAP@0.5 of 0.85+
- [ ] Replace mock ANPR with PaddleOCR production model
- [ ] Fine-tune vehicle counting for congestion heatmaps
- [ ] Deploy on Jetson Orin Nano for real onboard testing
- [ ] Migrate SQLite → PostGIS for spatial queries
- [ ] Add role-based auth for authorities

---

## 📜 License

This project is licensed under the **MIT License** — see [LICENSE](LICENSE) for details.

---

## 👤 Author

**Chirag Rathore**
- GitHub: https://github.com/chiragrathore937/mobile-urban-intelligence
- Email: chiragrathore937@gmail.com

*Built during SIH 2026 — Mobile Urban Intelligence Platform*
