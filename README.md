# CCTV Event Monitor

A React + FastAPI application for reviewing CCTV footage with YOLO detection, persistent tracking IDs, event inference, and SQLite storage.

## Features

- Upload MP4, AVI, MOV, or MKV footage from the web interface.
- Detect `person`, `backpack`, `bottle`, `phone`, and a `box` fallback (COCO's `suitcase` class is normalized to `box`).
- Track objects across frames with Ultralytics `ByteTrack` or `BoT-SORT`.
- Infer entering, leaving, appearing, disappearing, placed, picked up, and unattended events.
- Store every event with timestamp, object, tracking ID, type, confidence, and details in SQLite.
- Search the timeline or ask questions such as `When was the backpack last seen?`.
- Select an event to start playback at its timestamp.
- Spatial activity heatmap overlay.
- AI security incident reports with risk scoring.

## Architecture

- **Frontend**: React + Vite + TypeScript (`frontend/`)
- **Backend**: FastAPI (`src/api.py`)
- **AI/ML**: Ultralytics YOLO + ByteTrack/BoT-SORT (`src/analyzer.py`, `src/events.py`)
- **Database**: SQLite (`src/db.py`)

## Setup

### Backend

Python 3.10+ is recommended.

```powershell
py -m venv .venv
.\.venv\Scripts\Activate.ps1
python -m pip install --upgrade pip
pip install -r requirements.txt
```

### Frontend

Node.js 18+ is required.

```powershell
cd frontend
npm install
```

## Running the Application

### Development Mode

Run the backend (from project root):

```powershell
uvicorn src.api:app --reload --port 8000
```

Run the frontend dev server (from `frontend/`):

```powershell
npm run dev
```

The frontend will be available at `http://localhost:5173` with API proxying to the backend.

### Production Mode

Build the frontend:

```powershell
cd frontend
npm run build
```

Run the backend (it will serve the frontend build):

```powershell
uvicorn src.api:app --host 0.0.0.0 --port 8000
```

Open `http://localhost:8000` in your browser.

## Data and Database

Uploaded videos are saved under `data/uploads/`, and the SQLite database is `data/events.db`. These paths are ignored by Git. Each analysis creates a video row and its associated event rows.

## Tests

```powershell
pytest
```

The event tests do not need model weights or a video file. To test the full pipeline, upload a short clip through the web interface.

## API Endpoints

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/api/health` | Service health and stats |
| POST | `/api/upload` | Upload video file |
| GET | `/api/videos` | List all videos |
| GET | `/api/videos/{id}` | Get video details |
| DELETE | `/api/videos/{id}` | Delete video and events |
| GET | `/api/events/{video_id}` | List events (optional `?search=`) |
| GET | `/api/stats/{video_id}` | Event statistics |
| POST | `/api/analyze` | Start analysis job |
| GET | `/api/jobs` | List analysis jobs |
| WS | `/ws/progress/{job_id}` | Real-time progress updates |
| POST | `/api/query` | Natural language event search |
| POST | `/api/report` | Generate audit report |

## Notes

- The event detector is heuristic: `placed` and `picked_up` are inferred from the object's distance to a tracked person, and `unattended` is emitted after eight seconds without nearby-person contact. Adjust `EventDetector` thresholds for the camera angle and scene.
- On first analysis, Ultralytics downloads the selected YOLO weights. The default is `yolo11s.pt`.
