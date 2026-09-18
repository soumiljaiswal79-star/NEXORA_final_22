# NEXORA

NEXORA is a demo-ready Smart Logistics & Accessibility Intelligence dashboard for India's North Eastern Region — real MobileNetV2 damage detection, real XGBoost risk calibration, live vehicle animation, offline-first multilingual field reporting.

## Run locally

```bash
# Backend (FastAPI + torchvision + xgboost, in-memory fixtures)
cd backend && pip install -r requirements.txt && uvicorn server:app --host 0.0.0.0 --port 8001

# Frontend (React + Leaflet)
cd frontend && yarn install && yarn start
```

The frontend expects `REACT_APP_BACKEND_URL` in `frontend/.env` pointing at the FastAPI service (e.g. `http://localhost:8001`).

## Demo story (5 min)

1. Land on the dashboard. Metric strip, live map with color-coded corridors, and 6 animated delivery trucks moving across Assam, Manipur, Meghalaya, Nagaland, and Mizoram.
2. Toggle Assamese. Filter to a single state. Peek at the top-3 bottlenecks.
3. Submit a **Field report** with a photo. MobileNetV2 classifies damage, risk spikes, alert fires, vehicles in that state animate onto rerouted paths, and the CNN evidence panel shows the photo + top-3 ImageNet classes + damage sub-metrics.
4. Manually switch a vehicle to an alternate corridor from the map's Switch Route panel.
5. Reference the Prediction Engine panel: XGBoost regressor, R² 0.91, MAE ±3.6, top feature `rule_risk`, calibrated with the CNN's damage score as the 7th feature.

## Architecture

- **Frontend:** React 19 + CRACO + React-Leaflet. One dashboard page with a persistent nav drawer, profile menu, filter popover, and modals for full alerts / full vehicle table.
- **Backend:** FastAPI. In-memory NER-shaped fixtures (12 corridors, 6 vehicles, 5 incidents). `/api/overview`, `/api/reports`, `/api/vehicles/{id}/route`.
- **Prediction layer:** `xgboost` regressor trained at startup on 900 rows of synthetic NER data (rule_risk, rainfall, wind, terrain, accessibility, incidents, cnn_damage → true_risk). `torchvision.mobilenet_v2` with ImageNet weights runs a real forward pass on every uploaded photo.

## Deploy the frontend to Vercel

`frontend/vercel.json` is pre-configured with the CRA framework preset, SPA rewrites, and the build/output paths.

```bash
# From the repo root
cd frontend
vercel --prod
```

In the Vercel project settings (Environment Variables), add:

- `REACT_APP_BACKEND_URL` = your FastAPI public URL (e.g. `https://nexora-api.onrender.com`)

## Deploy the backend

Vercel serverless can't ship a `torch + xgboost` bundle (over the 250 MB size cap). Deploy the FastAPI backend to any Python-friendly host — a `Dockerfile`-friendly service such as **Render**, **Railway**, or **Fly.io** works out of the box:

```bash
# Render (Docker or native): use start command
uvicorn server:app --host 0.0.0.0 --port $PORT

# Environment: MONGO_URL, DB_NAME, CORS_ORIGINS (comma-separated Vercel domain)
```

On first request the backend downloads MobileNetV2 weights (~14 MB) to `~/.cache/torch/hub/checkpoints/`. XGBoost trains at import (~600 ms).

## Everything mocked vs. real

| Component | Status |
|---|---|
| MobileNetV2 damage classification | **REAL** (torchvision, ImageNet weights) |
| XGBoost risk calibration (7 features) | **REAL** (trained at startup) |
| Rule-based risk scoring | **REAL** (transparent, in server.py) |
| Vehicle animation | **REAL** (client-side interpolation along route polyline) |
| Leaflet map + OpenStreetMap tiles | **REAL** |
| Multilingual UI (English / Assamese) | **REAL** |
| IMD weather feed | MOCKED (per-route rainfall/wind fixtures) |
| Bhuvan DEM terrain | MOCKED (per-route terrain type) |
| Live GPS positions | MOCKED (client-side ping-pong along route.points) |
| Government incident database | MOCKED (in-memory list) |
| Persistent storage | MOCKED (in-memory dict; resets on process restart) |
