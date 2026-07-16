# Frontend

## Frontend Stack

The frontend uses React, TypeScript, Vite, React Router, TanStack Query, Axios, Zod, React Hook Form, and a shared CSS style layer.

## Folder Structure

- `src/assets`
- `src/components`
- `src/features`
- `src/hooks`
- `src/layouts`
- `src/lib`
- `src/pages`
- `src/routes`
- `src/services`
- `src/store`
- `src/styles`
- `src/types`
- `src/utils`

## Page List

- Login
- Signup
- Coach dashboard
- Athlete list
- Athlete profile
- Video upload
- AI review detail
- Drill assignment
- Athlete dashboard
- Athlete feedback list and detail
- Athlete drill list and detail
- Athlete timeline
- Athlete goals
- Athlete profile
- Invitation acceptance

## Routing Plan

Public routes include login and invitation acceptance. `RequireCoachRoute` renders the coach shell and `RequireAthleteRoute` renders the athlete shell. Wrong-role users are sent to an unauthorized page. Coaches default to `/dashboard`; athletes default to `/athlete/dashboard`.

## API Integration Strategy

Use shared Axios infrastructure for auth and error normalization, with dedicated typed modules for athlete dashboard, profile, goals, timeline, drills, and feedback. Athlete modules call only `/api/v1/athlete/*` endpoints and never send an athlete ID.

## State Management

Start with React state and context for auth. Add a dedicated store only when cross-page state becomes difficult to manage.

## Auth Handling

Store the JWT for the current browser tab, validate it with `/auth/me`, attach it to API requests, and clear credentials and query data on logout or `401`. The authenticated role selects the application shell and navigation.

## UI Standards

Use consistent forms, tables, status badges, timeline items, upload states, and review approval controls.

## Testing Plan

- Auth route rendering
- Protected route behavior
- API client error handling
- Athlete CRUD UI
- Video upload state handling
- Review approval UI
- Role restoration and coach/athlete route separation
- Invitation acceptance
- Athlete dashboard and safe feedback rendering
- Athlete assignment start, progress, and completion
- Athlete empty, loading, error, and unauthorized states

## Future Improvements

- Offline-friendly drill tracking
- Rich video annotation UI
- Secure athlete video playback after Media Service defines explicit visibility authorization

## Production Delivery

The frontend uses a multi-stage Node 22 build and an unprivileged Nginx runtime on port `8080`. Production API URLs are root-relative edge paths, so browser traffic stays same-origin. The container exposes liveness/readiness endpoints and an internal Nginx status endpoint for Prometheus export. CI runs lint, typecheck, tests, build, and image publication.

## Coach Review UI

The frontend labels original AI output, active coach draft, and approved snapshot separately. The structured editor supports additions, removals, and keyboard-accessible reordering. Preview hides private coach notes. Approval and rejection require explicit confirmation; stale saves ask the coach to reload instead of overwriting work.

## Drill And Assignment UI

Protected routes provide the drill library, create/edit/detail views, athlete drill lists, and assignment detail. The athlete profile adds a Drills tab, and approved review recommendations link into an explicit assignment dialog.

The assignment dialog has three modes: library, approved AI review, and custom. Review mode can assign without saving, save a new private drill, or map to an existing drill. Frontend route state only selects the starting recommendation; Athlete Service validates all trusted content again.

TanStack Query separates library, drill detail, athlete assignment list, assignment detail, timeline, and approved-review keys. Status mutations invalidate assignment detail/list, athlete timeline, dashboard data, and the library when a recommendation creates a drill. Completion and cancellation are never optimistic.

## Athlete UI

The athlete shell is responsive and distinct from the coach workspace. Routes include `/athlete/dashboard`, `/athlete/feedback`, `/athlete/drills`, `/athlete/timeline`, `/athlete/goals`, and `/athlete/profile`.

The dashboard shows profile context, active goals, assignment counts, recent progress, and approved feedback without inventing an AI performance score. Drill detail supports start, progress, and completion but no cancellation. Feedback pages display only immutable approved content returned by athlete-specific backend schemas.

Athlete video playback is intentionally absent because Media Service does not yet expose an athlete-visible authorization contract.

## Coach Progress Insights UI

Coach routes are `/insights`, `/athletes/:athleteId/insights`, and `/insights/attention`. The coach navigation includes Insights, and athlete profiles link directly to their insight page.

Range, comparison, custom dates, attention filters, sorting, search, and pagination live in URL search parameters. TanStack Query caches reads for three minutes, retains previous data during refetches, and insight keys are invalidated after drill, goal, and review mutations.

The UI uses accessible value-labelled bars instead of adding a chart dependency. Metric cards expose calculation help, comparisons, null states, and source-record links. Partial upstream responses show a restrained warning while preserving local metrics. Tests cover recurring labels, privacy exclusions, partial data, range requests, coach summaries, attention filtering, sorting, pagination, and athlete navigation.
