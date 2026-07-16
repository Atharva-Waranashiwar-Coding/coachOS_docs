# Database

## Database Choice: PostgreSQL

CoachOS uses PostgreSQL because the domain is relational: coaches own athletes, athletes have sessions, sessions have videos, and videos have reviews, drills, assignments, and timeline events.

## Core Tables

- `users`
- `account_invitations`
- `athletes`
- `athlete_user_links`
- `practice_sessions`
- `videos`
- `ai_reviews`
- `review_results`
- `review_revisions`
- `review_generation_jobs`
- `ai_observations`
- `coach_reviews`
- `drills`
- `drill_assignments`
- `timeline_events`

## Table Responsibilities

- `users`: login identity, email, nullable password hash for pending accounts, role, and lifecycle status
- `account_invitations`: hashed single-use athlete setup tokens, expiration, use state, and external athlete correlation
- `athletes`: athlete profile, sport, position, goals, injury notes
- `athlete_user_links`: Athlete Service mapping between a profile and an external Auth Service user
- `practice_sessions`: session date, type, notes, owner
- `videos`: storage key, upload status, metadata, linked session
- `ai_reviews`: request inputs, safe context snapshot, provider metadata, and lifecycle status
- `review_results`: structured generated coaching draft
- `review_revisions`: append-only coach corrections
- `review_generation_jobs`: retryable asynchronous review work
- `ai_observations`: structured strengths, issues, and recommendations
- `coach_reviews`: coach edits, approvals, and final feedback
- `drills`: reusable drill definitions
- `drill_assignments`: drills assigned to athletes
- `timeline_events`: feed of profile updates, uploads, reviews, and assignments

## Relationships

- A user is a coach, athlete, or future admin Auth Service identity.
- A coach can manage many athletes.
- An athlete has one primary coach in the MVP.
- An athlete profile can have one current invited or active Auth Service identity link.
- A practice session belongs to an athlete and coach.
- A video belongs to a practice session.
- An AI review belongs to a video.
- Coach reviews approve or override AI reviews.
- Drill assignments belong to athletes and can reference coach reviews.

## Foreign Keys

Foreign keys exist only inside a service-owned database:

- Auth Service: `account_invitations.user_id -> users.id`
- Athlete Service: `athlete_user_links.athlete_id -> athletes.id`
- Athlete Service: goals, assignments, assignment activities, and timeline records reference local athlete or assignment rows
- Media Service: videos reference local practice sessions
- AI Review Service: review results, revisions, jobs, and approved snapshots reference local reviews

Auth user IDs, source review IDs, video IDs, practice session IDs, and producer event IDs are external identifiers when stored by another service. CoachOS does not create cross-database foreign keys.

## Indexes

- Unique index on `users.email`
- Unique index on `account_invitations.token_hash`
- Indexes on invitation user and expiration
- Index on `athletes.primary_coach_id`
- Partial unique indexes on current `athlete_user_links.athlete_id` and `athlete_user_links.auth_user_id`
- Index on `practice_sessions.athlete_id, created_at`
- Index on `videos.practice_session_id`
- Index on `ai_reviews.video_id`
- Index on `timeline_events.athlete_id, created_at`
- Composite insight indexes on drill assignment athlete/due/status and athlete/completion time
- Composite insight indexes on goal athlete/target/status and athlete/completion time
- Composite insight index on timeline athlete/visibility/occurrence time

## Migration Strategy

Each service owns its Alembic migrations. Migrations should be forward-only during MVP development unless a reset is explicitly planned.

## Future Scaling Notes

Run one isolated PostgreSQL database per backend service in development and production. The current Docker Compose deployment uses four PostgreSQL containers and persistent volumes so credentials, lifecycle, backups, and migrations remain independent. Use object storage for video files and keep only metadata in Media PostgreSQL.

Logical backups use custom-format `pg_dump` files and SHA-256 sidecars per database. Full-platform recovery should use a coordinated backup set. PostgreSQL dumps do not include MinIO objects.

## Athlete Account Tables

`users.status` uses `pending`, `active`, and `disabled`. A pending athlete account may have no password hash until invitation acceptance. Public coach signup creates an active account. Invitation acceptance hashes the submitted password, marks the token used, activates the user, and activates the Athlete Service identity link.

`account_invitations.token_hash` stores a SHA-256 digest of the opaque token. Raw invitation tokens exist only in the outbound invitation URL. Resending invalidates prior unused tokens and rate limits repeated issuance.

`athlete_user_links` stores `athlete_id`, external `auth_user_id`, invitation email, status, inviter, and lifecycle timestamps. The active/invited partial unique indexes prevent one identity from controlling multiple current athlete profiles and prevent multiple current identities for one athlete.

## Athlete Assignment Activity

`drill_assignment_activities.actor_type` records `coach` or `athlete`. `note_visibility` records `coach_only` or `athlete_visible`. Optional actual sets, repetitions, and duration capture completion details without changing the assignment's original target snapshot.

# Timeline And Outbox

`timeline_events` stores canonical events ordered by `occurred_at`, `created_at`, and ID. A partial unique index on non-null `external_event_id` enforces producer idempotency. Category, visibility, source, entity, and athlete/time indexes support coach queries. Producer `outbox_events` tables index status/availability, creation time, and aggregate identity and retain bounded retry state without cross-service foreign keys.

## Coach Review Workflow Tables

- `review_revisions` stores immutable coach changes; `ai_reviews.latest_revision_number` is the optimistic concurrency token.
- `approved_review_snapshots` stores one immutable, athlete-safe content copy per review.
- `review_rejections` stores a coach-only structured category and private reason.
- `review_audit_events` stores safe actor/action metadata and never full review text or coach notes.

## Drill Tables

`drills` stores private coach-owned reusable definitions with category, difficulty, JSONB equipment and tags, optional target defaults, visibility, and soft-archive state. Owner, category, difficulty, status, lower-title, and GIN tag indexes support library access and filtering.

`drill_assignments` belongs to an athlete and optionally references a library drill and approved AI review recommendation. Required title and instruction snapshots remain immutable relative to their source. Constraints enforce priority `1..5`, progress `0..100`, paired review ID/index, non-negative recommendation index, positive targets, valid dates, and terminal completed/cancelled state.

`drill_assignment_activities` records assigned, started, updated, progress-updated, completed, and cancelled actions. It is assignment-specific audit history; the athlete timeline receives only safe summaries.

Foreign keys stay inside Athlete Service: assignments reference `athletes` and optional `drills`; activities reference assignments. `source_review_id` and Auth Service user IDs are external identifiers, not cross-database foreign keys.

## Progress Insight Storage

Stage 10 adds no analytics snapshot tables. Athlete Service calculates progress insights from authoritative assignment, activity, goal, and timeline records. AI Review and Media remain authoritative for approved feedback and practice/video activity.

Optional `taxonomy_code` fields live inside structured strength and improvement-area payloads. Historical approved snapshots may omit them. Athlete Service normalizes recurring labels with taxonomy codes first, a versioned alias file second, and a conservative normalized title fallback.

The Stage 10 Athlete Service migration adds only missing composite indexes:

- `drill_assignments (athlete_id, due_date, status)`
- `drill_assignments (athlete_id, completed_at)`
- `athlete_goals (athlete_id, target_date, status)`
- `athlete_goals (athlete_id, completed_at)`
- `timeline_events (athlete_id, visibility, occurred_at)`

Materialized views, snapshots, and a separate analytics database are deferred until production measurements justify their consistency and operational cost.
