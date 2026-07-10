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
- Use nested routes only when ownership is clear, such as `/athletes/{athlete_id}/timeline`.
- Use action routes sparingly, such as `/videos/{video_id}/complete-upload`.

## Service-Wise Endpoints

### Auth Service

- `POST /auth/signup`
- `POST /auth/login`
- `GET /auth/me`
- `POST /auth/refresh`

### Athlete Service

- `GET /athletes`
- `POST /athletes`
- `GET /athletes/{athlete_id}`
- `PATCH /athletes/{athlete_id}`
- `DELETE /athletes/{athlete_id}`
- `GET /athletes/{athlete_id}/timeline`

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

Review requests include athlete, session, uploaded-video IDs, a review type, and bounded textual context. Structured results contain summary, observations, strengths, improvement areas, recommended drills, and limitations. API responses never include raw provider output, credentials, passwords, or storage URLs.

## Request/Response Examples

Signup request:

```json
{
  "email": "coach@example.com",
  "password": "secure-password",
  "role": "coach"
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
# Internal Timeline Ingestion

`POST /internal/v1/athletes/{athlete_id}/timeline-events` requires `X-Service-Name` and `X-Service-Token`. The body carries a producer UUID `event_id`, category, visibility, actor, occurrence time, source identity, safe metadata, and `schema_version: 1`. First ingestion returns `201`; an identical retry returns `200`; reuse with changed content returns `409`. Public coach queries support category, type, source, visibility, and date filters.
