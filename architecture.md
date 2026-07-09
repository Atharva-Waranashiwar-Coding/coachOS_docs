# Architecture

## System Overview

CoachOS uses a React frontend with separate FastAPI backend services for authentication, athlete management, media handling, and AI review. Services communicate over HTTP APIs and persist data in PostgreSQL.

## Why Microservices

The project is split by product responsibility so each domain can evolve independently. Auth, athlete profiles, media upload, and AI review have different scaling, security, and dependency needs.

## Service List

- Auth service: coach signup, login, roles, JWT issuance
- Athlete service: athlete profiles, relationships, goals, timeline
- Media service: video upload flow, storage metadata, practice sessions
- AI review service: AI summary generation, observations, coach review workflow
- Frontend: coach and athlete web experience
- Infra: local development, deployment, CI/CD, monitoring

## Frontend/Backend Interaction

The frontend calls each backend service through REST APIs. Authenticated requests include a bearer token from the auth service. The frontend should use a shared API client and typed request/response models.

## Database Strategy

PostgreSQL is the primary database. Each service should own its tables and migrations. In early local development, one PostgreSQL instance can host separate databases or schemas per service.

## Authentication Flow

1. Coach signs up or logs in through the frontend.
2. Frontend calls auth service.
3. Auth service validates credentials and returns a JWT.
4. Frontend stores the token using the selected auth strategy.
5. Subsequent API calls include `Authorization: Bearer <token>`.
6. Services validate JWT claims before returning protected data.

## Video Upload Flow

1. Coach selects an athlete and practice session.
2. Frontend requests a signed upload URL from media service.
3. Media service creates upload metadata and asks the storage provider for a signed URL.
4. Frontend uploads video directly to cloud storage.
5. Frontend confirms upload completion with media service.
6. Media service links the video to athlete, coach, and practice session.

## AI Review Flow

1. Coach selects an uploaded video.
2. Frontend requests AI review generation.
3. AI review service fetches video metadata and contextual athlete data.
4. AI review service calls the AI provider.
5. Generated observations are stored as draft review content.
6. Coach edits, approves, or rejects the review.
7. Approved feedback becomes visible in athlete-facing workflows.

## Deployment Overview

Local development starts with Docker Compose. Production can use containerized services behind an API gateway or ingress, managed PostgreSQL, object storage, and a hosted frontend.

## Future Architecture Upgrades

- Service-to-service auth
- Event-driven review generation
- Background workers for long-running video analysis
- Redis for queues and caching
- Centralized logging and tracing
- Dedicated API gateway
