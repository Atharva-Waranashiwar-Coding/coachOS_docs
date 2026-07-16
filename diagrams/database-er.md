# Database ER Diagram

## Entity Relationship Diagram

```mermaid
erDiagram
  USERS ||--o{ ACCOUNT_INVITATIONS : receives
  ATHLETES ||--o{ ATHLETE_USER_LINKS : has_history
  ATHLETES ||--o{ ATHLETE_GOALS : pursues
  ATHLETES ||--o{ PRACTICE_SESSIONS : attends
  PRACTICE_SESSIONS ||--o{ VIDEOS : contains
  AI_REVIEWS ||--o| REVIEW_RESULTS : generates
  AI_REVIEWS ||--o{ REVIEW_REVISIONS : revised_as
  AI_REVIEWS ||--o| APPROVED_REVIEW_SNAPSHOTS : approves_as
  ATHLETES ||--o{ DRILL_ASSIGNMENTS : receives
  DRILLS ||--o{ DRILL_ASSIGNMENTS : assigned_as
  DRILL_ASSIGNMENTS ||--o{ DRILL_ASSIGNMENT_ACTIVITIES : records
  ATHLETES ||--o{ TIMELINE_EVENTS : has
```

The diagram groups service-owned entities for domain understanding. It does not imply cross-service foreign keys. `ATHLETE_USER_LINKS.auth_user_id`, media identifiers stored by AI Review Service, and source review identifiers stored by Athlete Service are external IDs.

## Users

Auth Service identity for coaches and invited athletes, including role and account status.

## Account Invitations

Auth Service hashed, expiring, single-use athlete password setup tokens.

## Athletes

Athlete profile, sport context, goals, and injury notes.

## Athlete User Links

Athlete Service link between a local athlete profile and an external Auth Service user ID.

## Practice Sessions

Training sessions associated with a coach and athlete.

## Videos

Video metadata and storage references.

## AI Observations

Structured AI-generated review details.

## Coach Reviews

Coach revisions, approvals, rejections, immutable snapshots, and final athlete visibility.

## Drills

Reusable training activities.

## Assignments

Drills assigned to athletes with status, due dates, progress, target snapshots, and actor-aware activity.

## Timeline Events

Chronological record of athlete activity and feedback.
