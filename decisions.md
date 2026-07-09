# Decisions

## Decision 1: PostgreSQL Over NoSQL

CoachOS uses PostgreSQL because the data model depends on clear relationships between users, coaches, athletes, sessions, videos, reviews, drills, and assignments.

## Decision 2: Microservice-Style Repos

Each major domain gets its own repository folder so service ownership, dependencies, and deployment boundaries remain clear.

## Decision 3: No Monorepo

The project is intentionally split into separate repo folders instead of a single application monorepo. This keeps future service extraction and independent deployment straightforward.

## Decision 4: FastAPI Backend

Backend services use FastAPI because it is lightweight, typed, async-ready, and provides OpenAPI documentation by default.

## Decision 5: React Frontend

The frontend uses React with Vite and TypeScript for fast local development, strong typing, and a broad ecosystem.

## Decision 6: Cloud Storage For Videos

Videos should be stored in cloud object storage. PostgreSQL stores metadata only, avoiding large binary files in the database.

## Decision 7: AI API First, Custom ML Later

The MVP uses an external AI API for review generation. Custom ML can be explored after enough domain-specific data and product usage exist.
