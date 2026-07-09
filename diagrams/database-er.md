# Database ER Diagram

## Entity Relationship Diagram

```mermaid
erDiagram
  USERS ||--o| COACHES : has
  USERS ||--o| ATHLETES : may_have
  COACHES ||--o{ COACH_ATHLETES : manages
  ATHLETES ||--o{ COACH_ATHLETES : assigned_to
  COACHES ||--o{ PRACTICE_SESSIONS : creates
  ATHLETES ||--o{ PRACTICE_SESSIONS : attends
  PRACTICE_SESSIONS ||--o{ VIDEOS : contains
  VIDEOS ||--o{ AI_REVIEWS : reviewed_by
  AI_REVIEWS ||--o{ AI_OBSERVATIONS : includes
  AI_REVIEWS ||--o{ COACH_REVIEWS : approved_by
  ATHLETES ||--o{ DRILL_ASSIGNMENTS : receives
  DRILLS ||--o{ DRILL_ASSIGNMENTS : assigned_as
  ATHLETES ||--o{ TIMELINE_EVENTS : has
```

## Users

Authentication identity for coaches and future athletes.

## Coaches

Coach profile linked to a user account.

## Athletes

Athlete profile, sport context, goals, and injury notes.

## Practice Sessions

Training sessions associated with a coach and athlete.

## Videos

Video metadata and storage references.

## AI Observations

Structured AI-generated review details.

## Coach Reviews

Coach edits, approvals, rejections, and final feedback.

## Drills

Reusable training activities.

## Assignments

Drills assigned to athletes with status and due dates.

## Timeline Events

Chronological record of athlete activity and feedback.
