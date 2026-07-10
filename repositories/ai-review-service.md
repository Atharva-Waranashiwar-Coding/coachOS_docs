# AI Review Service

## Responsibility

The AI Review Service owns asynchronous coaching review requests, sanitized context snapshots, structured provider output, coach revisions, lifecycle state, and timeline outbox events. Athlete profile, practice session, and video records remain owned by their source services.

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

## Future Improvements

- Queue-backed workers and distributed locks
- Provider routing, cost accounting, and tracing
- Provider-neutral vision pipeline with explicit consent and media policy
- Coach revision editor and drill-assignment handoff
