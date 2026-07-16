# Athlete Service

## Service Responsibility

The Athlete Service owns athlete profiles, explicit Auth identity links, coach ownership, goals, injury notes, reusable drills, athlete drill assignments, progress state, and athlete timelines.

## Athlete Profile Model

Core fields include name, date of birth, sport, position, dominant side, goals, injury notes, and development status.

## Coach-Athlete Relationship

The MVP uses one primary coach per athlete. Only that coach can invite, resend, or disable athlete account access.

## Athlete Identity Link

`athlete_user_links` maps a local athlete profile to an external Auth Service user ID. Status is `invited`, `active`, or `disabled`. Partial unique indexes allow one current link per athlete and per Auth user. Athlete self-service dependencies resolve the active link from the JWT and never accept `athlete_id` from the browser.

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
- Coach invitation `invite`, `invite/resend`, and `access/disable` endpoints
- `GET /api/v1/athlete/me`, `/dashboard`, `/timeline`, and `/goals`
- `GET /api/v1/athlete/drill-assignments` and assignment detail
- Athlete assignment `start`, `progress`, and `complete` action endpoints
- `GET|POST /api/v1/drills`
- `GET|PATCH|DELETE /api/v1/drills/{drill_id}`
- `POST /api/v1/drills/{drill_id}/restore`
- `GET|POST /api/v1/athletes/{athlete_id}/drill-assignments`
- `GET|PATCH /api/v1/athletes/{athlete_id}/drill-assignments/{assignment_id}`
- Assignment `start`, `complete`, and `cancel` action endpoints

## Tables Owned

- `athletes`
- `athlete_user_links`
- `athlete_goals`
- `timeline_events`
- `drills`
- `drill_assignments`
- `drill_assignment_activities`

## Validation Rules

- Coach must own or have access to athlete.
- Required profile fields must be present on create.
- Injury notes should be private to authorized coach users.
- Timeline references must point to valid known event types.
- Private drills are accessible only to their owner; system drills are read-only.
- Review assignments must resolve against an accessible immutable approved review.
- Assignment source snapshots do not change when a library drill changes.
- Athlete self queries always use the resolved profile and enforce athlete-visible timeline policy.
- Athlete responses omit injury notes, private coach notes, cancellation reasons, and unsafe metadata.
- Athlete progress cannot decrease; 100 is reserved for the completion action.
- Athletes cannot cancel assignments.

## Testing Plan

- Athlete CRUD
- Coach access control
- Profile validation
- Timeline ordering
- Goal creation and updates
- Drill ownership, search, archive, restore, and system read-only behavior
- Library, approved-review, mapped-review, and custom assignment modes
- Assignment lifecycle, rollback, privacy, and timeline events
- Invitation creation, resend, disable, and Auth activation callback
- Athlete identity isolation and wrong-role rejection
- Athlete dashboard, feedback fallback, goals, timeline, assignment actions, and safe serialization

## Future Improvements

- Multi-coach support
- Team/group support
- Progress tags and searchable notes
- Athlete profile editing with explicit field-level policy
- Extract a Drill Service only when independent scale, marketplace ownership, or deployment needs justify it
# Timeline Ingestion

Athlete Service authenticates internal callers separately from user JWTs, enforces per-service event allowlists, validates athlete existence, detects idempotency conflicts, and serves filtered coach timeline queries. Local profile and goal events use the same canonical model without HTTP or an outbox.

## AI Review Integration

`AIReviewServiceClient` forwards the current coach JWT to `GET /api/v1/reviews/{review_id}/approved`. Athlete Service verifies approval, athlete identity, and recommendation index before beginning local writes. `5xx` and transport failures return `503`; inaccessible upstream resources are hidden as `404`.

Drill lifecycle emits local `drill_assigned`, `drill_started`, `drill_completed`, and `drill_cancelled` events. Timeline metadata excludes coach notes, completion notes, cancellation reasons, complete instructions, and AI rationale.

## Athlete Dashboard

The dashboard aggregates Athlete Service-owned profile, active goals, drill status counts, progress status, and recent athlete-visible timeline events. It requests a small approved-feedback summary from AI Review Service and degrades gracefully if that service is unavailable.

## Athlete Drill Activity

Assignment activity records identify the actor as `coach` or `athlete` and store note visibility explicitly. Athlete start, progress, and completion actions write activity and timeline state in one transaction. Completion is idempotent and records optional actual sets, repetitions, and duration.
