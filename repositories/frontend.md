# Frontend

## Frontend Stack

The frontend uses React, TypeScript, Vite, and CSS modules or a shared style layer selected during implementation.

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

## Routing Plan

Use public auth routes and protected app routes. Route guards should check auth state before rendering coach-only pages.

## API Integration Strategy

Use a shared API client that injects auth headers and normalizes error responses from all services.

## State Management

Start with React state and context for auth. Add a dedicated store only when cross-page state becomes difficult to manage.

## Auth Handling

Store JWT according to the selected security model, attach it to API requests, and clear it on logout or token expiration.

## UI Standards

Use consistent forms, tables, status badges, timeline items, upload states, and review approval controls.

## Testing Plan

- Auth route rendering
- Protected route behavior
- API client error handling
- Athlete CRUD UI
- Video upload state handling
- Review approval UI

## Future Improvements

- Athlete mobile-first experience
- Offline-friendly drill tracking
- Rich video annotation UI
- Role-based navigation
