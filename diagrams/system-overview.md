# System Overview Diagram

## High-Level Architecture Diagram

```mermaid
flowchart LR
  Frontend[React Frontend<br/>Coach and Athlete Shells]
  Auth[Auth Service]
  Athlete[Athlete Service]
  Media[Media Service]
  AI[AI Review Service]
  AuthDB[(Auth PostgreSQL)]
  AthleteDB[(Athlete PostgreSQL)]
  MediaDB[(Media PostgreSQL)]
  AIDB[(AI Review PostgreSQL)]
  Storage[(Cloud Storage)]
  Provider[AI Provider]

  Frontend --> Auth
  Frontend --> Athlete
  Frontend --> Media
  Frontend --> AI
  Athlete -->|Provision invited identity| Auth
  Auth -->|Activate identity link| Athlete
  AI -->|Resolve athlete identity| Athlete
  Athlete -->|Approved feedback summary| AI
  Auth --> AuthDB
  Athlete --> AthleteDB
  Media --> MediaDB
  AI --> AIDB
  Media --> Storage
  AI --> Provider
```

## Frontend

React/Vite app with separate role-aware coach and athlete routes, layouts, navigation, and typed API modules.

## Backend Services

FastAPI services split by auth, athletes, media, and AI review.

## PostgreSQL

Each service owns its PostgreSQL tables and Alembic migrations. External service identifiers are stored as values without cross-database foreign keys.

## Cloud Storage

Stores uploaded videos and generated media artifacts.

## AI Provider

External AI API used for MVP review generation.
