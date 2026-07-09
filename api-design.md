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

- `POST /reviews`
- `GET /reviews/{review_id}`
- `PATCH /reviews/{review_id}`
- `POST /reviews/{review_id}/approve`
- `POST /reviews/{review_id}/reject`

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
