# Database

## Database Choice: PostgreSQL

CoachOS uses PostgreSQL because the domain is relational: coaches own athletes, athletes have sessions, sessions have videos, and videos have reviews, drills, assignments, and timeline events.

## Core Tables

- `users`
- `coaches`
- `athletes`
- `coach_athletes`
- `practice_sessions`
- `videos`
- `ai_reviews`
- `ai_observations`
- `coach_reviews`
- `drills`
- `drill_assignments`
- `timeline_events`

## Table Responsibilities

- `users`: login identity, email, password hash, role
- `coaches`: coach profile fields linked to user identity
- `athletes`: athlete profile, sport, position, goals, injury notes
- `coach_athletes`: relationship between coaches and athletes
- `practice_sessions`: session date, type, notes, owner
- `videos`: storage key, upload status, metadata, linked session
- `ai_reviews`: generated summary and status
- `ai_observations`: structured strengths, issues, and recommendations
- `coach_reviews`: coach edits, approvals, and final feedback
- `drills`: reusable drill definitions
- `drill_assignments`: drills assigned to athletes
- `timeline_events`: feed of profile updates, uploads, reviews, and assignments

## Relationships

- A user may be a coach or athlete identity.
- A coach can manage many athletes.
- An athlete can belong to multiple coaches later, but MVP can start with one primary coach.
- A practice session belongs to an athlete and coach.
- A video belongs to a practice session.
- An AI review belongs to a video.
- Coach reviews approve or override AI reviews.
- Drill assignments belong to athletes and can reference coach reviews.

## Foreign Keys

- `coaches.user_id -> users.id`
- `athletes.user_id -> users.id`, nullable for athlete accounts created later
- `coach_athletes.coach_id -> coaches.id`
- `coach_athletes.athlete_id -> athletes.id`
- `practice_sessions.athlete_id -> athletes.id`
- `practice_sessions.coach_id -> coaches.id`
- `videos.practice_session_id -> practice_sessions.id`
- `ai_reviews.video_id -> videos.id`
- `ai_observations.ai_review_id -> ai_reviews.id`
- `coach_reviews.ai_review_id -> ai_reviews.id`
- `drill_assignments.athlete_id -> athletes.id`
- `timeline_events.athlete_id -> athletes.id`

## Indexes

- Unique index on `users.email`
- Index on `athletes.primary_coach_id`
- Index on `coach_athletes.coach_id, athlete_id`
- Index on `practice_sessions.athlete_id, created_at`
- Index on `videos.practice_session_id`
- Index on `ai_reviews.video_id`
- Index on `timeline_events.athlete_id, created_at`

## Migration Strategy

Each service owns its Alembic migrations. Migrations should be forward-only during MVP development unless a reset is explicitly planned.

## Future Scaling Notes

Start with one PostgreSQL instance. Split into separate databases per service when operational complexity is justified. Use object storage for video files and keep only metadata in PostgreSQL.
