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

## Decision 8: One Frontend, Two Role-Aware Shells

CoachOS uses one React deployment with separate coach and athlete layouts, route guards, navigation, and query contracts. This keeps shared authentication and UI infrastructure without mixing role-specific workflows.

## Decision 9: Explicit Athlete Identity Link

Auth Service owns users and Athlete Service owns athlete profiles. `athlete_user_links` stores the external Auth user ID in Athlete Service. Cross-service database foreign keys are prohibited.

## Decision 10: No Public Athlete Signup

Public signup creates coaches only. A primary coach invites an existing athlete profile, and the athlete activates a pending account through a single-use password setup token.

## Decision 11: Dedicated Athlete Schemas

Athlete-facing APIs use dedicated response models. Coach schemas are not filtered dynamically for athlete use because omission at serialization time is a stronger and more testable privacy boundary.

## Decision 12: Visibility Is Enforced By Backends

Frontend hiding is not authorization. Athlete timeline and feedback services query only `athlete_visible` records for the authenticated athlete.

## Decision 13: No AI Performance Score

The athlete dashboard summarizes goals, assignment completion, and approved feedback. It does not derive a numeric performance score from AI output because the current evidence does not support that level of precision.

## Decision 14: Athlete Assignment Actions Are Limited

Athletes may start assignments, increase progress, and complete them. They may not cancel assignments. Completion is idempotent, progress cannot decrease, and activity records identify the athlete as actor.
# Timeline Decisions

- Athlete Service remains timeline source of truth.
- MVP transport is internal HTTP plus transactional outboxes, with no message broker.
- Producers generate stable idempotency UUIDs once at outbox creation.
- User chronology uses domain `occurred_at`, not ingestion time.
- `coach_only` and `athlete_visible` are explicit data policy, not frontend-only decoration.

## Coach Review Decisions

- Coach approval is mandatory before any athlete-visible feedback exists.
- Approval creates an immutable snapshot; revisions are append-only and never overwritten.
- `coach_only` is the approval default. Coach notes and rejection reasons never enter athlete previews or timeline metadata.
- CoachOS uses optimistic concurrency and does not automatically merge simultaneous edits.

## Drill Assignment Decisions

- Athlete Service owns drills and assignments for the MVP; there is no dedicated Drill Service.
- AI recommendations and athlete assignments are different domain records.
- A coach must explicitly create every assignment; no worker or approval action auto-assigns recommendations.
- Assignments snapshot source content so library edits cannot rewrite athlete history.
- Archived drills remain referenced by existing assignments and cannot be selected for new assignments.
- Approved recommendation text is retrieved server-to-server and never trusted from the frontend.
