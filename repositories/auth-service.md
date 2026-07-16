# Auth Service

## Service Responsibility

The Auth Service owns user identity, public coach signup, invite-only athlete account activation, login, role claims, password hashing, account status, and JWT issuance.

## Auth Flow

1. A coach signs up publicly, or a coach or athlete submits login credentials.
2. The service validates account status, input, and password.
3. The service returns a JWT with user ID, email, role, and expiration.
4. Other services validate the token and role before serving protected resources.

Athlete accounts are provisioned only through authenticated internal calls from Athlete Service. Auth Service stores a hash of the invitation token, accepts the password setup request, activates the user, and calls Athlete Service to activate the identity link.

## User Roles

- `coach`: public signup and coach workspace
- `athlete`: invitation-only athlete workspace
- `admin`: future operational role

## Tables Owned

- `users`
- `account_invitations`
- Future: `refresh_tokens`, `password_resets`

## Endpoints

- `POST /auth/signup`
- `POST /auth/login`
- `GET /auth/me`
- `POST /auth/invitations/accept`
- `POST /internal/v1/athlete-users`
- `POST /internal/v1/athlete-users/{auth_user_id}/resend`
- `POST /internal/v1/athlete-users/{auth_user_id}/disable`

## Environment Variables

- `APP_NAME`
- `ENVIRONMENT`
- `DATABASE_URL`
- `JWT_SECRET_KEY`
- `JWT_ALGORITHM`
- `JWT_ACCESS_TOKEN_EXPIRE_MINUTES`
- `INTERNAL_SERVICE_TOKENS`
- `ATHLETE_INVITATION_EXPIRATION_HOURS`
- `ATHLETE_INVITATION_BASE_URL`
- `ALLOW_DEV_INVITATION_URL_RESPONSE`
- `MAX_INVITATION_RESENDS_PER_HOUR`
- `ATHLETE_SERVICE_INTERNAL_URL`
- `INTERNAL_SERVICE_NAME`
- `INTERNAL_SERVICE_TOKEN`
- `UPSTREAM_TIMEOUT_SECONDS`
- `CORS_ORIGINS`
- `LOG_LEVEL`
- `METRICS_ENABLED`
- `REQUEST_ID_HEADER`
- `RUN_MIGRATIONS`

## Local Run Steps

```bash
pip install -r requirements.txt
uvicorn main:app --reload
```

The container entrypoint runs `alembic upgrade head`. Runtime endpoints are `/health/live`, `/health/ready`, and `/metrics`; JSON logs include `X-Request-ID`.

## Testing Plan

- Signup validation
- Duplicate email rejection
- Password hashing verification
- Login success and failure
- JWT claim generation
- Protected route access
- Public role-escalation rejection
- Internal service authentication
- Hashed invitation storage, expiration, single use, and resend invalidation
- Athlete activation callback and disabled account behavior

## Future Improvements

- Refresh tokens
- Password reset
- Email verification
- OAuth login
- Multi-factor authentication
- Production email delivery
