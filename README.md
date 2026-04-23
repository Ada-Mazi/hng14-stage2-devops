# HNG Stage 2 — Containerized Microservices

A job processing system with three services: frontend, API, and worker.

## Prerequisites

- Docker Desktop installed and running
- Docker Compose v2+
- Git

## Quick Start

    git clone https://github.com/Ada-Mazi/hng14-stage2-devops
    cd hng14-stage2-devops
    cp .env.example .env
    docker compose up --build

Visit http://localhost:3000 to submit jobs.

## Services

| Service | Port | Description |
|---------|------|-------------|
| Frontend | 3000 | Job submission UI |
| API | 8000 (internal) | FastAPI job manager |
| Worker | internal | Background job processor |
| Redis | internal only | Job queue |

## Endpoints

- POST /submit — Submit a new job
- GET /status/:id — Check job status

## Successful Startup

You should see all 4 containers running:

    redis    - healthy
    api      - healthy
    worker   - started
    frontend - started

## Environment Variables

See .env.example for all required variables.
