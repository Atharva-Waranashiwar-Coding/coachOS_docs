# Athlete Service

## Service Responsibility

The athlete service owns athlete profiles, coach-athlete relationships, goals, injury notes, and athlete timelines.

## Athlete Profile Model

Core fields include name, date of birth, sport, position, dominant side, goals, injury notes, and development status.

## Coach-Athlete Relationship

MVP can start with one primary coach per athlete. The model should allow a many-to-many relationship later through `coach_athletes`.

## Timeline Ownership

The athlete service owns timeline events or provides the timeline aggregation endpoint. Events may reference videos, reviews, drills, or profile updates.

## Endpoints

- `GET /athletes`
- `POST /athletes`
- `GET /athletes/{athlete_id}`
- `PATCH /athletes/{athlete_id}`
- `DELETE /athletes/{athlete_id}`
- `GET /athletes/{athlete_id}/timeline`
- `POST /athletes/{athlete_id}/goals`

## Tables Owned

- `athletes`
- `coach_athletes`
- `athlete_goals`
- `timeline_events`

## Validation Rules

- Coach must own or have access to athlete.
- Required profile fields must be present on create.
- Injury notes should be private to authorized coach users.
- Timeline references must point to valid known event types.

## Testing Plan

- Athlete CRUD
- Coach access control
- Profile validation
- Timeline ordering
- Goal creation and updates

## Future Improvements

- Athlete account linking
- Multi-coach support
- Team/group support
- Progress tags and searchable notes
# Timeline Ingestion

Athlete Service authenticates internal callers separately from user JWTs, enforces per-service event allowlists, validates athlete existence, detects idempotency conflicts, and serves filtered coach timeline queries. Local profile and goal events use the same canonical model without HTTP or an outbox.
