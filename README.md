# Z50 Real Sports Odds Intelligence Platform

A production-style simulation of an odds engine:

Data -> Stochastic Model -> API -> UI

## Architecture

- Node.js + Express backend API
- Python FastAPI service with weighted probability and confidence engine
- React frontend War Room dashboard (auction + prediction + explain agent)
- PostgreSQL for auction entities, rosters, history, users, and favorites

## Features

- JWT authentication (register/login)
- Synthetic Dataset Generator with Python auction seeding
- 100 generated players with attributes and market auction values
- 8 franchises with roster auction allocation (11 players each)
- 50 weighted historical matches plus upcoming fixtures
- Match listing with filters: sport, league, upcoming
- AI-generated odds from team power, form, synergy, variance, injuries, and data points
- Confidence score formula from data points and variance
- Auction view and prediction War Room dashboard
- Favorites system for authenticated users
- Comparative explain-agent endpoint

## Project Structure

- backend: Express API and DB seeding orchestrator
- ai-service: FastAPI weighted predictor + explanation agent
- frontend: React Vite dashboard
- db: relational schema + synthetic data generator

## Quick Start

### 1) Start PostgreSQL

```bash
docker compose up -d
```

### 2) AI service setup (needed for Python seed dependencies)

```bash
cd ai-service
python -m venv .venv
.venv\Scripts\activate
pip install -r requirements.txt
```

### 3) Generate synthetic auction + history data

```bash
cd backend
copy .env.example .env
npm install
npm run seed
```

This initializes schema and runs db/seed_data.py.

### 4) Run backend API

```bash
npm run dev
```

Backend runs on http://localhost:4000

### 5) Run AI service

```bash
uvicorn app.main:app --reload --port 8000
```

AI service runs on http://localhost:8000

### 6) Frontend setup

```bash
cd frontend
copy .env.example .env
npm install
npm run dev
```

Frontend runs on http://localhost:5173

## Deployment

- Production deployment guide: see DEPLOYMENT.md
- Backend supports both `AI_SERVICE_URL` and `PYTHON_SERVICE_URL` for Python service integration.
- Root seeder entrypoint is available as `seed_db.py`.

### 60-Minute Launch Path

1. Railway: provision PostgreSQL and copy `DATABASE_URL`.
2. Railway: deploy AI service from `ai-service` root.
3. Railway: deploy backend from `backend` root and set `DATABASE_URL`, `AI_SERVICE_URL` (or `PYTHON_SERVICE_URL`), and `JWT_SECRET`.
4. Vercel: deploy frontend from `frontend` root and set `VITE_API_BASE_URL` to backend URL.
5. Verify:
	- `/health` on backend
	- `/docs` on AI service
	- match cards load on frontend

## Live Demo

- Frontend: add your deployed URL here
- Backend health: add your deployed `/health` URL here
- AI docs: add your deployed `/docs` URL here

## API Endpoints

### Auth

- POST /api/auth/register
- POST /api/auth/login

### Matches

- GET /api/matches?sport=&league=&upcoming=true
- GET /api/matches/:id/odds
- GET /api/matches/:id/explain

GET /api/matches now returns AI-generated odds and probabilities for every match.

### Players (Auction)

- GET /api/players
- GET /api/players?teamId={teamId}

### Agent

- POST /api/agent/query

### Favorites (JWT required)

- GET /api/favorites
- POST /api/favorites { matchId }
- DELETE /api/favorites/:matchId

## Example AI Answer

For a matchup like Team A vs Team B, the explain endpoint returns a sentence such as:

"Team A has a 62% win probability. While Team B has the higher power profile, Team A leads consistency across the top five players. Confidence is 84% based on 15 historical matchups."

## Notes

- Odds are generated from stochastic model features and not hardcoded.
- Confidence follows data-point and variance dynamics.
- Python service exposes POST /generate-odds and POST /generate-odds-batch.
- Node backend uses batch calls plus in-memory cache to reduce repeated inference calls.
