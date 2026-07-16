# Service Interactions

## Login Flow

```mermaid
sequenceDiagram
  participant UI as Role-aware Frontend
  participant Auth as Auth Service
  participant DB as PostgreSQL
  UI->>Auth: POST /auth/login
  Auth->>DB: Load user by email
  DB-->>Auth: User record
  Auth-->>UI: JWT with user ID, email, and role
  UI->>Auth: GET /auth/me
  Auth-->>UI: Authenticated user
  UI->>UI: Select coach or athlete shell
```

## Coach Invites Athlete

```mermaid
sequenceDiagram
  participant Coach as Coach UI
  participant Athlete as Athlete Service
  participant Auth as Auth Service
  participant ADB as Athlete DB
  participant UDB as Auth DB
  Coach->>Athlete: POST /athletes/{id}/invite
  Athlete->>ADB: Verify primary coach and create invited link
  Athlete->>Auth: POST /internal/v1/athlete-users
  Auth->>UDB: Create pending athlete user and hashed token
  Auth-->>Athlete: Auth user ID and invitation URL
  Athlete->>ADB: Save external Auth user ID
  Athlete-->>Coach: Invitation status
```

## Athlete Accepts Invitation

```mermaid
sequenceDiagram
  participant AthleteUI as Athlete Browser
  participant Auth as Auth Service
  participant Athlete as Athlete Service
  participant UDB as Auth DB
  participant ADB as Athlete DB
  AthleteUI->>Auth: POST /auth/invitations/accept
  Auth->>UDB: Verify token hash, expiry, and unused state
  Auth->>UDB: Hash password and activate user
  Auth->>Athlete: POST /internal/v1/athlete-user-links/{auth_user_id}/activate
  Athlete->>ADB: Activate matching identity link
  Auth-->>AthleteUI: Active athlete user
```

## Athlete Login And Profile Resolution

```mermaid
sequenceDiagram
  participant UI as Athlete UI
  participant Auth as Auth Service
  participant Athlete as Athlete Service
  participant DB as Athlete DB
  UI->>Auth: POST /auth/login
  Auth-->>UI: Athlete JWT
  UI->>Athlete: GET /api/v1/athlete/me with JWT
  Athlete->>Athlete: Require athlete role
  Athlete->>DB: Resolve active link by Auth user ID
  DB-->>Athlete: Athlete profile
  Athlete-->>UI: Athlete-safe profile
```

## Athlete Dashboard

```mermaid
sequenceDiagram
  participant UI as Athlete UI
  participant Athlete as Athlete Service
  participant AI as AI Review Service
  participant DB as Athlete DB
  UI->>Athlete: GET /api/v1/athlete/dashboard
  Athlete->>DB: Profile, goals, drills, progress, timeline
  Athlete->>AI: GET athlete-visible feedback summary
  alt AI Review available
    AI-->>Athlete: Approved feedback summary
  else AI Review unavailable
    AI--xAthlete: Timeout or 5xx
    Athlete->>Athlete: Continue without feedback summary
  end
  Athlete-->>UI: Dashboard
```

## Athlete Reads Approved Feedback

```mermaid
sequenceDiagram
  participant UI as Athlete UI
  participant AI as AI Review Service
  participant Athlete as Athlete Service
  participant DB as AI Review DB
  UI->>AI: GET /api/v1/athlete/reviews
  AI->>Athlete: Resolve athlete identity with JWT
  Athlete-->>AI: Current athlete ID
  AI->>DB: Query approved + athlete_visible + athlete ID
  DB-->>AI: Immutable snapshots
  AI-->>UI: Athlete-safe feedback schemas
```

## Athlete Starts And Completes Drill

```mermaid
sequenceDiagram
  participant UI as Athlete UI
  participant Athlete as Athlete Service
  participant DB as Athlete DB
  UI->>Athlete: POST assignment/start
  Athlete->>DB: Verify resolved athlete owns assignment
  Athlete->>DB: Save in_progress and athlete activity
  UI->>Athlete: POST assignment/progress
  Athlete->>DB: Save monotonic progress and safe activity
  UI->>Athlete: POST assignment/complete
  Athlete->>DB: Set completed, progress=100, activity, timeline
  Athlete-->>UI: Completed assignment
```

## Athlete Timeline

```mermaid
sequenceDiagram
  participant UI as Athlete UI
  participant Athlete as Athlete Service
  participant DB as Athlete DB
  UI->>Athlete: GET /api/v1/athlete/timeline
  Athlete->>DB: Resolve active identity link
  Athlete->>DB: Query athlete ID and athlete_visible only
  DB-->>Athlete: Safe chronological events
  Athlete-->>UI: Paginated timeline
```

## Revision And Approval Flow

```mermaid
sequenceDiagram
  participant UI as Coach UI
  participant AI as AI Review Service
  participant O as Outbox
  participant T as Athlete Timeline
  UI->>AI: POST revision (expected revision number)
  AI-->>UI: 201 revision or 409 stale revision
  UI->>AI: POST preview
  AI-->>UI: Athlete-safe content only
  UI->>AI: POST approve (visibility, confirmation)
  AI->>O: Commit immutable snapshot and approval event
  O->>T: Deliver safe timeline event
```

## Athlete Creation Flow

```mermaid
sequenceDiagram
  participant UI as Frontend
  participant Athlete as Athlete Service
  participant DB as PostgreSQL
  UI->>Athlete: POST /athletes
  Athlete->>DB: Insert athlete and relationship
  DB-->>Athlete: Athlete record
  Athlete-->>UI: Created athlete
```

## Video Upload Flow

```mermaid
sequenceDiagram
  participant UI as Frontend
  participant Media as Media Service
  participant Storage as Cloud Storage
  participant DB as PostgreSQL
  UI->>Media: POST /videos/upload-url
  Media->>DB: Create pending video
  Media->>Storage: Request signed URL
  Storage-->>Media: Signed URL
  Media-->>UI: Upload URL and video ID
  UI->>Storage: Upload video
  UI->>Media: POST /videos/{id}/complete-upload
  Media->>DB: Mark uploaded
```

## AI Review Flow

```mermaid
sequenceDiagram
  participant UI as Frontend
  participant AI as AI Review Service
  participant Provider as AI Provider
  participant DB as PostgreSQL
  UI->>AI: POST /reviews
  AI->>DB: Load video and athlete context
  AI->>Provider: Generate review
  Provider-->>AI: Structured review
  AI->>DB: Store draft review
  AI-->>UI: Review draft
```

## Coach Approval Flow

```mermaid
sequenceDiagram
  participant UI as Frontend
  participant AI as AI Review Service
  participant DB as PostgreSQL
  UI->>AI: PATCH /reviews/{id}
  UI->>AI: POST /reviews/{id}/approve
  AI->>DB: Save coach edits and approved status
  AI-->>UI: Approved review
```

## Drill Assignment Flow

```mermaid
sequenceDiagram
  participant UI as Frontend
  participant Athlete as Athlete Service
  participant DB as PostgreSQL
  UI->>Athlete: POST /athletes/{id}/drill-assignments
  Athlete->>DB: Insert assignment and timeline event
  Athlete-->>UI: Drill assignment
```

## Assign From Library

```mermaid
sequenceDiagram
  participant Coach
  participant UI as Frontend
  participant Athlete as Athlete Service
  participant DB as Athlete DB
  Coach->>UI: Select library drill and assignment options
  UI->>Athlete: POST drill-assignments mode=library
  Athlete->>DB: Verify drill and primary coach
  Athlete->>DB: Insert snapshot, activity, and drill_assigned
  Athlete-->>UI: 201 assignment
```

## Assign Approved Recommendation

```mermaid
sequenceDiagram
  participant Coach
  participant UI as Frontend
  participant Athlete as Athlete Service
  participant AI as AI Review Service
  participant DB as Athlete DB
  Coach->>UI: Select one approved recommendation
  UI->>Athlete: POST mode=review with review ID and index
  Athlete->>AI: GET approved review with coach JWT
  AI-->>Athlete: Immutable approved snapshot
  Athlete->>Athlete: Verify athlete, approval, and index
  Athlete->>DB: Insert assignment snapshot, activity, timeline
  Athlete-->>UI: 201 assignment
```

## Save Recommendation To Library

```mermaid
sequenceDiagram
  participant UI as Frontend
  participant Athlete as Athlete Service
  participant AI as AI Review Service
  participant DB as Athlete DB
  UI->>Athlete: POST mode=review save_to_library=true
  Athlete->>AI: Fetch approved recommendation
  AI-->>Athlete: Trusted recommendation
  Athlete->>DB: Transaction: insert private drill
  Athlete->>DB: Insert assignment, activity, timeline
  Athlete-->>UI: Assignment with drill_id
```

## Complete Assignment

```mermaid
sequenceDiagram
  participant UI as Frontend
  participant Athlete as Athlete Service
  participant DB as Athlete DB
  UI->>Athlete: POST assignment/complete
  Athlete->>DB: Set completed and progress=100
  Athlete->>DB: Insert completed activity and safe timeline event
  Athlete-->>UI: Completed assignment
```

## Cancel Assignment

```mermaid
sequenceDiagram
  participant UI as Frontend
  participant Athlete as Athlete Service
  participant DB as Athlete DB
  UI->>Athlete: POST assignment/cancel with private reason
  Athlete->>DB: Set cancelled
  Athlete->>DB: Store private activity and coach-only timeline event
  Athlete-->>UI: Cancelled assignment
```
# Timeline Delivery

```mermaid
sequenceDiagram
  participant M as Media Service
  participant O as Media Outbox
  participant W as Outbox Worker
  participant A as Athlete Service
  M->>O: Commit video_uploaded with upload
  W->>O: Claim SKIP LOCKED
  W->>A: POST internal timeline event
  A-->>W: 201 created or 200 duplicate
  W->>O: Mark published
```

```mermaid
sequenceDiagram
  participant AI as AI Review Service
  participant O as AI Outbox
  participant A as Athlete Service
  AI->>O: Commit coach_review_approved
  O->>A: Publish athlete-visible event
  A-->>O: Idempotent success
```

```mermaid
sequenceDiagram
  participant W as Worker
  participant A as Athlete Service
  participant O as Outbox
  W-xA: Timeout or 5xx
  W->>O: Increment attempt and schedule backoff
  W->>A: Retry same event_id
```

## Athlete Progress Insight Request

```mermaid
sequenceDiagram
  participant UI as Coach Frontend
  participant Athlete as Athlete Service
  participant AI as AI Review Service
  participant Media as Media Service
  participant DB as Athlete DB
  UI->>Athlete: GET athlete insights with range and compare
  Athlete->>Athlete: Verify coach-athlete access and UTC boundaries
  Athlete->>DB: Load grouped drills, goals, activities, timeline
  par Safe upstream summaries
    Athlete->>AI: Approved review insight request
    Athlete->>Media: Practice and upload activity request
  end
  Athlete->>Athlete: Calculate metrics, trends, recurrence, flags
  Athlete-->>UI: Combined response with completeness
```

## Coach-Wide Progress Insight Request

```mermaid
sequenceDiagram
  participant UI as Coach Frontend
  participant Athlete as Athlete Service
  participant AI as AI Review Service
  participant Media as Media Service
  UI->>Athlete: GET coach insights
  Athlete->>Athlete: Resolve current coach athlete IDs
  Athlete->>AI: POST bounded approved-review batch
  Athlete->>Media: POST bounded activity batch
  Athlete->>Athlete: Aggregate without ranking or scoring
  Athlete-->>UI: Overview and attention preview
```

If AI Review or Media times out, Athlete Service keeps local drill and goal results, suppresses dependent flags, and returns `partial: true` with a safe warning code. It does not expose raw exceptions or retry one request per athlete.
