# System Overview Diagram

## High-Level Architecture Diagram

```mermaid
flowchart LR
  User[Coach or Athlete]
  Edge[Nginx HTTPS Edge]
  Frontend[React Frontend<br/>Coach and Athlete Shells]
  Auth[Auth Service]
  Athlete[Athlete Service]
  Media[Media Service]
  AI[AI Review Service]
  AuthDB[(Auth PostgreSQL)]
  AthleteDB[(Athlete PostgreSQL)]
  MediaDB[(Media PostgreSQL)]
  AIDB[(AI Review PostgreSQL)]
  Storage[(Private MinIO or S3)]
  Provider[AI Provider]
  Prometheus[Prometheus]
  Grafana[Grafana]
  Promtail[Promtail]
  Loki[Loki]

  User --> Edge
  Edge --> Frontend
  Edge --> Auth
  Edge --> Athlete
  Edge --> Media
  Edge --> AI
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
  Prometheus --> Auth
  Prometheus --> Athlete
  Prometheus --> Media
  Prometheus --> AI
  Grafana --> Prometheus
  Grafana --> Loki
  Promtail --> Loki
```

## Frontend

React/Vite app with separate role-aware coach and athlete routes, layouts, navigation, and typed API modules.

## Backend Services

FastAPI services split by auth, athletes, media, and AI review.

## PostgreSQL

Each service owns its PostgreSQL tables and Alembic migrations. External service identifiers are stored as values without cross-database foreign keys.

## Cloud Storage

Stores uploaded videos in a private S3-compatible bucket. PostgreSQL backups do not include these objects.

## AI Provider

External AI API used for MVP review generation.

## Operations

Docker Compose runs the edge, applications, isolated databases, storage, workers, and observability stack on one generic Docker host. Nginx is the only public application entrypoint.
