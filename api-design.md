# API Design

## API Standards

- Use REST-style JSON APIs.
- Use nouns for resource paths.
- Use plural resource names.
- Use `snake_case` for JSON fields unless frontend conventions require otherwise.
- Return consistent error envelopes.

## Base URLs

- Auth service: `http://localhost:8001`
- Athlete service: `http://localhost:8002`
- Media service: `http://localhost:8003`
- AI review service: `http://localhost:8004`
- Frontend: `http://localhost:5173`

## Authentication Headers

Protected endpoints require:

```http
Authorization: Bearer <jwt>
```

## Error Response Format

```json
{
  "error": {
    "code": "validation_error",
    "message": "One or more fields are invalid.",
    "details": {}
  }
}
```

## Pagination Format

Requests:

```http
GET /athletes?page=1&page_size=20
```

Responses:

```json
{
  "items": [],
  "page": 1,
  "page_size": 20,
  "total": 0
}
```

## Endpoint Naming Rules

- Use `/health` for service health checks.
- Use `/auth/signup` and `/auth/login` for auth actions.
- Use `/athletes/{athlete_id}` for athlete resources.
- Use `/api/v1/athlete/*` for athlete self-service resources resolved from the JWT.
- Use nested routes only when ownership is clear, such as `/athletes/{athlete_id}/timeline`.
- Use action routes sparingly, such as `/videos/{video_id}/complete-upload`.

## Service-Wise Endpoints

### Auth Service

- `POST /auth/signup`
- `POST /auth/login`
- `GET /auth/me`
- `POST /auth/invitations/accept`
- `POST /internal/v1/athlete-users`
- `POST /internal/v1/athlete-users/{auth_user_id}/resend`
- `POST /internal/v1/athlete-users/{auth_user_id}/disable`

Public signup does not accept a role and always creates a coach. Internal endpoints require service identity headers and are not exposed to browsers.

### Athlete Service

- `GET /athletes`
- `POST /athletes`
- `GET /athletes/{athlete_id}`
- `PATCH /athletes/{athlete_id}`
- `DELETE /athletes/{athlete_id}`
- `GET /athletes/{athlete_id}/timeline`
- `POST /api/v1/athletes/{athlete_id}/invite`
- `GET /api/v1/athletes/{athlete_id}/invite`
- `POST /api/v1/athletes/{athlete_id}/invite/resend`
- `POST /api/v1/athletes/{athlete_id}/access/disable`
- `GET /api/v1/athlete/me`
- `GET /api/v1/athlete/dashboard`
- `GET /api/v1/athlete/timeline`
- `GET /api/v1/athlete/goals`
- `GET /api/v1/athlete/drill-assignments`
- `GET /api/v1/athlete/drill-assignments/{assignment_id}`
- `POST /api/v1/athlete/drill-assignments/{assignment_id}/start`
- `POST /api/v1/athlete/drill-assignments/{assignment_id}/progress`
- `POST /api/v1/athlete/drill-assignments/{assignment_id}/complete`

Coach invitation endpoints require primary-coach access. Athlete self endpoints never accept an athlete ID and never expose injury notes, coach-only notes, private timeline metadata, or another athlete's records.

### Media Service

- `POST /videos/upload-url`
- `POST /videos/{video_id}/complete-upload`
- `GET /videos/{video_id}`
- `GET /athletes/{athlete_id}/videos`
- `POST /practice-sessions`

### AI Review Service

- `POST /api/v1/reviews` returns `202`; callers should supply `Idempotency-Key`.
- `GET /api/v1/reviews`, `/athletes/{athlete_id}/reviews`, and `/videos/{video_id}/reviews` support review queues.
- `GET /api/v1/reviews/{review_id}` and `/status` return the draft and asynchronous status.
- `PATCH /api/v1/reviews/{review_id}/draft` appends a coach revision.
- `POST /api/v1/reviews/{review_id}/approve`, `/reject`, `/retry`, and `/cancel` enforce lifecycle transitions.
- `GET /api/v1/athlete/reviews` lists immutable approved athlete-visible feedback for the authenticated athlete.
- `GET /api/v1/athlete/reviews/{review_id}` returns a dedicated athlete-safe detail contract.

Review requests include athlete, session, uploaded-video IDs, a review type, and bounded textual context. Structured results contain summary, observations, strengths, improvement areas, recommended drills, and limitations. API responses never include raw provider output, credentials, passwords, or storage URLs.

## Request/Response Examples

Signup request:

```json
{
  "email": "coach@example.com",
  "password": "secure-password"
}
```

Login response:

```json
{
  "access_token": "jwt",
  "token_type": "bearer",
  "user": {
    "id": "user-id",
    "email": "coach@example.com",
    "role": "coach"
  }
}
```

Invitation acceptance request:

```json
{
  "token": "opaque-single-use-token",
  "password": "new-secure-password",
  "password_confirmation": "new-secure-password"
}
```

Athlete progress request:

```json
{
  "completion_percentage": 60,
  "note": "Completed three controlled rounds."
}
```

Athlete assignment progress is monotonic and must remain below 100. The completion action sets 100 and is idempotent. Athletes have no cancellation endpoint.

## Athlete Response Contracts

Athlete APIs use dedicated Pydantic response models rather than reusing coach schemas. Athlete timeline queries enforce `athlete_visible` in the backend. AI feedback queries enforce all of the following: matching athlete identity, review status `approved`, immutable approved snapshot, and visibility `athlete_visible`.

Athlete feedback omits provider metadata, raw prompts, confidence values, evidence, private coach notes, audit records, and rejection details. Athlete dashboard progress is derived from assignment and goal state; it does not present an AI-generated performance score.

# Internal Timeline Ingestion

`POST /internal/v1/athletes/{athlete_id}/timeline-events` requires `X-Service-Name` and `X-Service-Token`. The body carries a producer UUID `event_id`, category, visibility, actor, occurrence time, source identity, safe metadata, and `schema_version: 1`. First ingestion returns `201`; an identical retry returns `200`; reuse with changed content returns `409`. Public coach queries support category, type, source, visibility, and date filters.

## Coach Review API

- `POST /api/v1/reviews/{id}/revisions` accepts `expected_revision_number`; stale edits return `409 STALE_REVIEW_REVISION`.
- `POST /api/v1/reviews/{id}/preview` excludes private coach notes, provider data, raw evidence, and operational limits.
- `POST /api/v1/reviews/{id}/approve` requires `confirmation: true`, visibility, and expected revision number, then creates an immutable snapshot.
- `POST /api/v1/reviews/{id}/reject` requires category, confirmation, and expected revision number. Free-text rejection reasons remain private.
- `GET /api/v1/reviews/{id}/approved` and `/audit-log` are coach-only in this stage.

## Drill And Assignment API

Drill library:

- `POST /api/v1/drills`
- `GET /api/v1/drills`
- `GET /api/v1/drills/{drill_id}`
- `PATCH /api/v1/drills/{drill_id}`
- `DELETE /api/v1/drills/{drill_id}`
- `POST /api/v1/drills/{drill_id}/restore`

Athlete assignments:

- `POST /api/v1/athletes/{athlete_id}/drill-assignments`
- `GET /api/v1/athletes/{athlete_id}/drill-assignments`
- `GET|PATCH /api/v1/athletes/{athlete_id}/drill-assignments/{assignment_id}`
- `POST .../{assignment_id}/start`
- `POST .../{assignment_id}/complete`
- `POST .../{assignment_id}/cancel`

Creation uses a discriminated `mode`: `library`, `review`, or `custom`. Review mode accepts only source IDs, an optional existing-drill mapping, assignment overrides, and `save_to_library`; the backend re-fetches approved recommendation content and ignores frontend-supplied recommendation text.

Coach transitions are `assigned -> in_progress|completed|cancelled` and `in_progress -> completed|cancelled`. Athlete transitions are limited to start, monotonic progress updates, and completion. Completed and cancelled assignments are terminal. Validation failures use the standard `422` envelope; inaccessible athletes, drills, reviews, and assignments return `404`.
