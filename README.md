# CoachOS Docs

Project documentation for CoachOS, a coaching platform for managing athletes, video review, AI-assisted feedback, drill assignments, and progress tracking.

## MVP Goal

Build connected coach and athlete workspaces where coaches manage athletes, video review, approved feedback, and drill assignments, while invited athletes track goals, feedback, assignments, timeline, and progress.

## Repository Links

- Auth service: `../coachos-auth-service`
- Athlete service: `../coachos-athlete-service`
- Media service: `../coachos-media-service`
- AI review service: `../coachos-ai-review-service`
- Frontend: `../coachos-frontend`
- Infrastructure: `../coachos-infra`
- Documentation: `../coachos-docs`

## Documentation Index

- [Product Requirements](prd.md)
- [Architecture](architecture.md)
- [Database](database.md)
- [API Design](api-design.md)
- [Roadmap](roadmap.md)
- [Decisions](decisions.md)
- [Auth Service](repositories/auth-service.md)
- [Athlete Service](repositories/athlete-service.md)
- [Media Service](repositories/media-service.md)
- [AI Review Service](repositories/ai-review-service.md)
- [Frontend](repositories/frontend.md)
- [Infrastructure](repositories/infra.md)
- [System Overview Diagram](diagrams/system-overview.md)
- [Service Interactions](diagrams/service-interactions.md)
- [Database ER](diagrams/database-er.md)

## How To Use These Docs

Use `prd.md` to understand product scope, `architecture.md` to understand system design, `roadmap.md` to decide build order, and repository docs to guide implementation inside each service. Update `decisions.md` whenever a technical direction changes.

## Maintenance Rules

- Keep docs current with implementation changes.
- Record technical direction changes in `decisions.md`.
- Update repository docs when service responsibilities, endpoints, or environment variables change.
- Prefer diagrams in Mermaid so they render in GitHub and stay editable.

## Current Build Stage

Stage 10: coach-facing progress insights are implemented with deterministic drill and goal metrics, approved-review normalization, activity summaries, attention flags, date comparisons, partial upstream responses, and coach-wide attention workflows.
