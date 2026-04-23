# Bug Fixes Documentation

## Fix 1 — api/main.py, Line 6
**Problem:** `redis.Redis(host="localhost")` — hardcoded to localhost. Inside Docker containers, services communicate via service names on a shared network, not localhost. This caused the API to fail to connect to Redis when containerized.
**Fix:** Changed to `redis.Redis(host=os.environ.get("REDIS_HOST", "redis"))` to read from environment variable.

## Fix 2 — worker/worker.py, Line 5
**Problem:** `redis.Redis(host="localhost")` — same hardcoded localhost issue as the API. Worker could not connect to Redis inside Docker.
**Fix:** Changed to `redis.Redis(host=os.environ.get("REDIS_HOST", "redis"))` to read from environment variable.

## Fix 3 — frontend/app.js, Line 5
**Problem:** `API_URL = "http://localhost:8000"` — hardcoded to localhost. Inside Docker, the frontend container cannot reach the API via localhost.
**Fix:** Changed to `process.env.API_URL || "http://api:8000"` to use environment variable with sensible default.

## Fix 4 — api/main.py — Missing queue name consistency
**Problem:** API used `r.lpush("job", job_id)` (queue name: "job") but worker used `r.brpop("job", ...)`. While the names matched, the queue name "job" is ambiguous and conflicts with the job hash key pattern `job:{id}`.
**Fix:** Renamed queue to "jobs" in both api/main.py and worker/worker.py for clarity and to avoid confusion with hash keys.

## Fix 5 — api/main.py — Missing health check endpoint
**Problem:** No /health endpoint existed, making it impossible for Docker and load balancers to verify the service is running.
**Fix:** Added `GET /health` endpoint returning `{"status": "ok"}`.

## Fix 6 — frontend/app.js — Missing health check endpoint
**Problem:** No /health endpoint in the frontend service.
**Fix:** Added `GET /health` endpoint returning `{"status": "ok"}`.

## Fix 7 — worker/worker.py — No graceful shutdown handling
**Problem:** Worker had no signal handlers. When Docker sends SIGTERM to stop the container, the worker would be killed abruptly mid-job, potentially corrupting job state.
**Fix:** Added `signal.signal(SIGTERM)` and `signal.signal(SIGINT)` handlers for graceful shutdown.

## Fix 8 — All services — No non-root user
**Problem:** All services ran as root inside containers — a critical security risk.
**Fix:** Added non-root user creation and USER instruction in all three Dockerfiles.

## Fix 9 — requirements.txt files — Unpinned dependencies
**Problem:** Neither api/requirements.txt nor worker/requirements.txt had pinned versions, making builds non-reproducible.
**Fix:** Added pinned versions in Dockerfiles using --no-cache-dir and multi-stage builds for reproducibility.
