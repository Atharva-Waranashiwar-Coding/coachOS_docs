# Architecture

## System Overview

CoachOS uses one React frontend with role-aware coach and athlete application shells. Separate FastAPI services own authentication, athlete management, media handling, and AI review. Services communicate over authenticated HTTP APIs and persist their own data in PostgreSQL.

## Why Microservices

The project is split by product responsibility so each domain can evolve independently. Auth, athlete profiles, media upload, and AI review have different scaling, security, and dependency needs.

## Service List

- Auth service: coach signup, invite-only athlete accounts, login, roles, JWT issuance
- Athlete service: athlete profiles, auth identity links, relationships, goals, drills, and timeline
- Media service: video upload flow, storage metadata, practice sessions
- AI review service: AI summary generation, observations, coach review workflow
- Frontend: coach and athlete web experience
- Infra: local development, deployment, CI/CD, monitoring

## Frontend/Backend Interaction

The frontend calls each backend service through REST APIs. Authenticated requests include a bearer token from Auth Service. Route guards select a coach or athlete shell from the authenticated role. Coach routes can use athlete IDs after the backend verifies ownership; athlete routes never accept an athlete ID and instead resolve the profile from the JWT user ID.

The athlete dashboard is a frontend composition over dedicated athlete-safe APIs. Athlete Service returns profile, goals, assignments, progress status, and timeline data. AI Review Service returns approved feedback. A failure in the optional AI summary request does not prevent the Athlete Service dashboard from loading its owned data.

## Database Strategy

PostgreSQL is the primary database. Each service should own its tables and migrations. In early local development, one PostgreSQL instance can host separate databases or schemas per service.

## Authentication And Athlete Invitation Flow

1. A coach signs up publicly or an existing coach logs in.
2. The coach creates an athlete profile in Athlete Service.
3. The primary coach invites the athlete by email.
4. Athlete Service creates an `athlete_user_links` row and calls Auth Service through an internal authenticated endpoint.
5. Auth Service creates or reuses a pending athlete user and issues a single-use invitation token. Only the token hash is stored.
6. The athlete opens the invitation URL, sets a password, and activates the account.
7. Auth Service calls Athlete Service to activate the matching identity link.
8. Coach and athlete users log in through the same endpoint. JWTs include user ID, email, role, and expiration.
9. Each protected service validates the JWT and applies role-aware authorization.

Public signup always creates a coach. Athlete accounts cannot self-register or choose an arbitrary role.

## Identity Linking

Auth Service owns login identities. Athlete Service owns athlete profiles and the explicit `auth_user_id -> athlete_id` mapping in `athlete_user_links`. The Auth user ID is an external identifier, not a database foreign key. Partial unique indexes allow only one invited or active link per athlete and one invited or active link per Auth user.

Athlete self-service dependencies require an `athlete` JWT role, locate the active link by Auth user ID, and return `404` or authorization errors without exposing another athlete's existence.

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
3. AI review service validates ownership with Athlete and Media APIs and stores a sanitized context snapshot with a review job.
4. A dedicated worker sends only structured textual context and metadata to the AI provider; it never sends raw video bytes or storage URLs.
5. Generated observations are validated and stored as a coach-only draft; provider failure retries with bounded backoff.
6. Coach edits, approves, rejects, retries, or cancels the review. Approval is the athlete-visible timeline transition.
7. Approved feedback becomes visible in athlete-facing workflows.

## Athlete Feedback Visibility

AI Review Service exposes dedicated athlete response schemas and queries only immutable approved snapshots whose visibility is `athlete_visible` and whose athlete ID matches the caller's resolved profile. Provider metadata, confidence, evidence, coach notes, rejection reasons, and raw operational fields are not serialized by athlete endpoints.

## Athlete Drill Completion Flow

Athletes can read only their own assignments and can start, increase progress, or complete an assignment. Athlete Service derives the athlete from the JWT, records the actor as `athlete`, stores note visibility explicitly, and appends safe timeline events in the same transaction. Athletes cannot cancel assignments; cancellation remains a coach action. Completion is idempotent and sets progress to 100.

## Deployment Overview

Local development starts with Docker Compose. Production can use containerized services behind an API gateway or ingress, managed PostgreSQL, object storage, and a hosted frontend.

## Future Architecture Upgrades

- Service-to-service auth
- Event-driven review generation
- Background workers for long-running video analysis
- Measured caching for expensive read paths
- Centralized logging and tracing
- Dedicated API gateway

# Unified Timeline Delivery

Athlete Service owns the canonical append-only timeline. Producer services commit domain state and an outbox row atomically, then separate workers deliver events over authenticated internal HTTP. Delivery is eventually consistent: domain operations never roll back because Athlete Service is unavailable. Stable producer event IDs make retries idempotent; no service writes another service's database.

## Human Approval Boundary

AI output is a coach-only baseline. The AI Review Service stores append-only coach revisions and requires an explicit confirmation before it creates an immutable approved snapshot. Visibility is selected at approval and defaults to `coach_only`; workers cannot publish feedback automatically. Optimistic concurrency compares the expected revision number and returns `409` rather than overwriting a newer coach edit.

## Drill And Assignment Architecture

The Athlete Service owns the MVP drill library, athlete drill assignments, assignment activity, and drill timeline events. A separate Drill Service is intentionally deferred until independent scaling, ownership, or deployment requirements justify another service boundary.

An AI recommendation is advisory content owned by the AI Review Service. It never becomes an assignment automatically. When a coach explicitly assigns a recommendation, Athlete Service fetches the immutable approved snapshot through the AI Review API using the coach bearer token, verifies the athlete and recommendation index, and then creates local records.

Assignments always store title, description, and instruction snapshots. Later edits or archival of a library drill do not rewrite athlete history. A recommendation can be assigned without a library record, saved as a new private drill, or mapped to an accessible existing drill while retaining the approved recommendation snapshot.

Assignment creation and lifecycle transitions write the assignment, activity record, and local timeline event in one Athlete Service transaction. External review retrieval happens before database writes so no transaction remains open during the HTTP call.

## Progress Insight Aggregation

Athlete Service owns MVP progress aggregation because it already owns coach-athlete authorization, drills, goals, and the canonical timeline. It calculates local metrics from grouped repository queries and requests bounded summaries from AI Review and Media through authenticated service APIs. No service reads another service's database.

The coach request flow is:

1. Athlete Service verifies coach access and resolves a start-inclusive, end-exclusive UTC period.
2. Local drill, goal, activity, and timeline records are loaded in grouped queries.
3. Athlete Service sends one or more bounded batch requests to AI Review and Media, never one request per athlete.
4. Domain-specific services calculate explicit metrics, comparisons, recurring labels, and rule-based attention flags.
5. The API returns local results even when an upstream service fails, with availability booleans and safe warning codes.

Trends are deterministic rate comparisons. AI is not used to assign direction, rank athletes, predict future performance, or produce a hidden score. Recurring feedback describes how often a structured area appears in approved reviews; it does not claim proven improvement or decline.

The MVP calculates insights on request and relies on database indexes plus the frontend's short TanStack Query cache. There is no Analytics Service, Redis cache, warehouse, or snapshot table. Extraction should be considered only after measured latency, query volume, independent ownership, or historical reporting needs exceed this design. A future extracted service must continue consuming source-service APIs or events rather than cross-database joins.
