# System Overview Diagram

## High-Level Architecture Diagram

```mermaid
flowchart LR
  Frontend[React Frontend]
  Auth[Auth Service]
  Athlete[Athlete Service]
  Media[Media Service]
  AI[AI Review Service]
  DB[(PostgreSQL)]
  Storage[(Cloud Storage)]
  Provider[AI Provider]

  Frontend --> Auth
  Frontend --> Athlete
  Frontend --> Media
  Frontend --> AI
  Auth --> DB
  Athlete --> DB
  Media --> DB
  AI --> DB
  Media --> Storage
  AI --> Provider
```

## Frontend

React/Vite app used by coaches and later athletes.

## Backend Services

FastAPI services split by auth, athletes, media, and AI review.

## PostgreSQL

Primary relational data store for users, athletes, sessions, metadata, reviews, and assignments.

## Cloud Storage

Stores uploaded videos and generated media artifacts.

## AI Provider

External AI API used for MVP review generation.
