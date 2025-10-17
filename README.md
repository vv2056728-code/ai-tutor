SocrAI Advanced Tutor (v2) — Professional Step-by-Step Setup & Deployment Guide

Below is a clean, professional, step-by-step guide to get socratic_tutor_advanced_v2.py running locally, prepare it for production, test it, and maintain it. Each section is explicit and actionable so you (or another engineer) can follow it end-to-end.

1 — Prerequisites (what you need first)

Python — Python 3.10 or 3.11 installed and available on PATH.

pip — Python package installer.

Git — for cloning and version control (optional but recommended).

OpenAI API key — a valid key for model calls (kept secret).

Recommended: a virtual environment tool (venv or virtualenv) to isolate dependencies.

2 — Clone project & prepare workspace

Create a project directory and go there (or clone if you’ve committed to GitHub):

mkdir socrai-advanced-tutor
cd socrai-advanced-tutor


Place socratic_tutor_advanced_v2.py into this directory. If using Git:

git init
git add socratic_tutor_advanced_v2.py
git commit -m "Add SocrAI Advanced Tutor v2 single-file app"

3 — Create and activate Python virtual environment

Create venv:

python -m venv .venv


Activate:

macOS / Linux:

source .venv/bin/activate


Windows (PowerShell):

.venv\Scripts\Activate.ps1


Confirm python refers to your venv’s interpreter:

python -V

4 — Install dependencies

Create a requirements.txt (recommended minimal set) — you can add exact pinned versions later:

streamlit
fastapi
uvicorn[standard]
openai
requests
pydantic


Install:

pip install -r requirements.txt
# Optional:
pip install streamlit-agraph

5 — Local run (development/testing)

v2 is implemented as a single file that launches a FastAPI backend thread and the Streamlit frontend in the same process.

Run the app:

streamlit run socratic_tutor_advanced_v2.py


Streamlit will open in a browser (usually http://localhost:8501).

The FastAPI backend will run on http://localhost:8000 (spawned by the script).

First-time UI steps

Open the sidebar, paste your OpenAI API key, click Save API Key.

Choose Persona and Mode, type a Topic and argument, and click Start Dialogue.

6 — Common verification & smoke tests

In browser, confirm Streamlit UI loads and the sidebar inputs are visible.

Enter API key and start a simple dialogue — ensure you receive AI responses and no server errors.

Test endpoints manually (optional):

# example using curl (replace YOUR_KEY)
curl -X POST "http://localhost:8000/api/dialogue" \
  -H "Authorization: Bearer YOUR_KEY" \
  -H "Content-Type: application/json" \
  -d '{"mode":"Gentle","persona":"Socrates","topic":"What is justice?","student_text":"Justice is giving everyone what they deserve."}'


Check /api/trace and /api/summary with GET.

7 — Hardening for production (recommended changes)

Do not expose your raw script with in-memory storages in production. Replace or change the following:

Secret management

NEVER store API keys client-side. Use server-side secret storage (environment variables or vault).

Remove UI direct API key saving; accept a server-side API key or use an auth flow.

Separate services

Split into two processes (recommended):

FastAPI backend (socrAI_backend/) serving /api/* and handling OpenAI calls server-side.

Streamlit frontend (socrAI_frontend/) calling backend endpoints (no direct OpenAI calls).

Benefits: security, scaling, rate limiting, logging, and easier deployment.

Persistence

Replace in-memory lists (CONVERSATIONS, REASONING_TRACE, SESSION_PROFILES) with a database: PostgreSQL, SQLite (small scale), or a hosted DB.

Add proper record ownership and access controls.

Async & concurrency

Make OpenAI calls async (or use task queues like Celery/RQ) to avoid blocking the server under load.

Use HTTP timeouts and retry logic for external calls.

Rate limiting & quotas

Enforce per-user rate limits and overall API usage caps to avoid runaway costs.

Observability

Add structured logging, error reporting (Sentry), metrics (Prometheus + Grafana).

Security

Serve behind HTTPS (TLS).

Implement authentication & authorization (JWT, OAuth) for users / API access.

Input validation and sanitization for user content.

8 — Deploying to production (high-level options)

Option A — Docker + cloud VM

Write Dockerfile with a two-service layout (backend + frontend) or single container with process manager.

Deploy to cloud VM (AWS EC2, GCP Compute Engine, Azure VM) behind an Nginx reverse proxy with TLS.

Option B — Container orchestration

Create Docker images and deploy to Kubernetes (EKS/GKE/AKS) with separate deployments & autoscaling.

Option C — PaaS

Deploy FastAPI to a platform like Render, Fly.io, or Heroku for the backend.

Deploy Streamlit app to Streamlit Cloud or a separate service; configure it to call the backend.

9 — CI/CD & automated checks

Create a simple GitHub Actions workflow to:

Run flake8 / ruff for linting.

Run pytest for tests.

Build and publish Docker images on tags or merges to main.

Add unit tests for:

API endpoints (mock OpenAI).

Key utilities (term extraction, parsing JSON responses).

Integration tests for happy path and common failures.

10 — Testing & QA recommendations

Unit tests for small functions and edge cases (e.g., resilience to malformed model output).

Integration tests hitting the FastAPI endpoints using a mocked OpenAI client.

Manual exploratory tests: long conversations, concurrency, API key missing, model timeouts.

Load testing with a small tool (wrk, locust) against the backend to model scale.

11 — Troubleshooting common issues

Streamlit shows blank / script errors: check terminal for stack traces; ensure packages installed and Python version correct.

OpenAI errors: verify API key, network connectivity, and model availability. Add retries and timeouts.

Port already in use: confirm no other service uses :8000 or :8501; change ports if needed.

Long replay delays: time.sleep in Streamlit blocks the session — acceptable for demo but avoid in production.

12 — Recommended repository files (minimal)
README.md
socratic_tutor_advanced_v2.py
requirements.txt
.gitignore
Dockerfile         # optional for containerization
tests/              # unit & integration tests
deploy/             # deployment scripts (k8s yaml, helm, etc.)


Example .gitignore lines:

.venv/
__pycache__/
*.pyc
.env

13 — Maintenance & roadmap suggestions

Replace single-file architecture with modular packages (backend, frontend, shared schemas).

Add role-based access and user accounts.

Add analytics dashboard for model usage, costs, and top concepts.

Add more persona and style templates and a plugin system for custom analyses (e.g., domain-specific tutors).

14 — Legal & ethical considerations

Display clear disclaimer in the UI: model outputs are suggestions and may be imperfect.

Make privacy policy and data-retention rules explicit if you store student dialogue or analytics.

Consider consent for storing and using conversational data for training/analytics.

15 — Quick checklist (one-page)

 Python 3.10+ and virtualenv created

 requirements.txt installed

 Streamlit run OK (streamlit run ...)

 OpenAI API key saved in UI (or moved to server-side)

 Endpoints /api/dialogue, /api/trace, /api/summary, /api/extract_terms tested

 Exports (JSON/TXT) validated

 Add secrets & database before production

 Add logs, monitoring, and rate limiting

16 — Example commands summary (copy/paste)
# Setup
python -m venv .venv
source .venv/bin/activate      # or .venv\Scripts\Activate.ps1 on Windows
pip install -r requirements.txt

# Run locally (development)
streamlit run socratic_tutor_advanced_v2.py

# Test endpoint example (replace YOUR_KEY)
curl -X POST "http://localhost:8000/api/dialogue" \
  -H "Authorization: Bearer YOUR_KEY" \
  -H "Content-Type: application/json" \
  -d '{"mode":"Gentle","persona":"Socrates","topic":"What is justice?","student_text":"Justice is fai
