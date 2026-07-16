# AI Review Service

## Responsibility

The AI Review Service owns asynchronous coaching review requests, sanitized context snapshots, structured provider output, coach revisions, immutable approved snapshots, athlete-safe feedback reads, lifecycle state, and timeline outbox events. Athlete profile, practice session, and video records remain owned by their source services.

## Generation Flow

1. A coach submits an uploaded video, athlete, and practice session with optional notes, focus areas, transcript, and frame observations.
2. The API validates ownership through Athlete and Media APIs with the coach JWT, captures safe metadata, then writes `ai_reviews`, `review_generation_jobs`, and `ai_review_requested` in one transaction.
3. `review_worker` claims the job, passes only text and metadata to the configured provider, validates the provider response against the structured Pydantic schema, and persists `review_results`.
4. The review becomes `generated`; coaches can revise, approve, reject, retry a terminal failure, or cancel pending work.
5. `outbox_publisher` delivers requested/generated/failed and coach action events to Athlete Service's internal timeline endpoint. Only approval emits athlete-visible feedback.

Raw video, storage URLs, provider prompts, raw provider output, and credentials are not sent to the model or included in timeline payloads. The provider prompt bans claims of raw-video viewing and medical advice.

## States

- `pending`, `processing`, `generated`, `failed`, `cancelled`, `approved`, `rejected`
- Jobs use `pending`, `processing`, `completed`, `failed`, `cancelled` and bounded exponential retries.

## Endpoints

- `POST /api/v1/reviews` with `Idempotency-Key`
- `GET /api/v1/reviews`, `/athletes/{athlete_id}/reviews`, `/videos/{video_id}/reviews`
- `GET /api/v1/reviews/{id}`, `/api/v1/reviews/{id}/status`
- `PATCH /api/v1/reviews/{id}/draft`
- `POST /api/v1/reviews/{id}/approve`, `/reject`, `/retry`, `/cancel`
- `GET /api/v1/athlete/reviews`
- `GET /api/v1/athlete/reviews/{review_id}`

## Tables Owned

- `ai_reviews`: request inputs, safe context snapshot, provider metadata, and lifecycle status
- `review_results`: structured generated coaching draft
- `review_revisions`: append-only coach corrections
- `review_generation_jobs`: retryable asynchronous review work
- `outbox_events`: reliable delivery to Athlete Service

## Environment and Local Run

Use `DATABASE_URL`, `JWT_SECRET_KEY`, `ATHLETE_SERVICE_URL`, `MEDIA_SERVICE_URL`, internal timeline credentials, `OPENAI_API_KEY`, and `REVIEW_JOB_*` settings from `.env.example`.

```bash
alembic upgrade head
uvicorn app.main:app --reload --port 8004
python -m app.workers.review_worker
python -m app.workers.outbox_publisher
```

## Testing Plan

- Request and context ownership validation
- Idempotency and lifecycle transitions
- Structured provider parsing and prompt safety
- Job retries, cancellation, and failure event creation
- Outbox visibility and idempotent timeline delivery
- Athlete identity resolution through Athlete Service
- Approved and athlete-visible filtering
- Cross-athlete isolation and athlete-safe response serialization

## Future Improvements

- Queue-backed workers and distributed locks
- Provider routing, cost accounting, and tracing
- Provider-neutral vision pipeline with explicit consent and media policy
- Coach revision editor and drill-assignment handoff

## Coach Review Workflow

Generated results are coach-only. Coach revisions use expected revision numbers and are immutable. Approval stores one immutable snapshot with a selected visibility and excludes coach notes; rejection stores a private category/reason. The service records safe audit events for generation, revision, preview, approval, and rejection. The approval/rejection transactions also write their timeline outbox rows.

## Approved Recommendation Contract

`GET /api/v1/reviews/{review_id}/approved` returns the immutable snapshot plus `review_id`, `athlete_id`, `status: approved`, visibility, approval time, and structured recommended drills. Each recommendation includes name, description, reason, optional frequency, difficulty, and optional safety note.

The endpoint remains coach-authenticated and ownership-aware. Athlete Service consumes it through HTTP with the current coach JWT; it never reads the AI Review database. Recommendations remain advisory until a coach explicitly submits an Athlete Service assignment request.

## Athlete Feedback Contract

Athlete endpoints resolve the current athlete by forwarding the athlete JWT to Athlete Service. Queries require the exact athlete ID, review status `approved`, an immutable approved snapshot, and visibility `athlete_visible`.

Dedicated athlete schemas return summary, observations, strengths, improvement areas, recommended drills, an optional athlete message, approval time, and allowlisted session context. They exclude confidence, evidence, provider metadata, prompts, raw output, private coach notes, rejection reasons, and audit details.

## Progress Insight Contract

The service exposes an approved-review insight endpoint for one athlete and a bounded internal batch endpoint for Athlete Service. Queries include only immutable approved snapshots inside the requested half-open UTC period.

Strength and improvement-area schemas support nullable `taxonomy_code`. Insight responses include structured titles, descriptions, priorities, taxonomy codes, visibility, approval timestamps, review type, recommendations, and source session/video IDs. They exclude coach notes, generated and rejected reviews, prompts, provider metadata, token usage, confidence/evidence internals, raw model output, and revision history.

Batch requests require Athlete Service internal authentication and reject lists larger than `INSIGHT_MAX_BATCH_ATHLETES`. Coach-authorized single-athlete reads verify access through Athlete Service.

## Production Operations

The API runs Alembic before accepting traffic; review and outbox workers disable migrations and wait for readiness. The service exposes liveness, database readiness, and Prometheus metrics, and all processes emit JSON logs. AI Review PostgreSQL is isolated and independently backed up through the infra scripts.
