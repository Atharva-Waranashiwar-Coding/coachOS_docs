# Auth Service

## Service Responsibility

The auth service owns user identity, coach signup, login, role claims, password hashing, and JWT issuance.

## Auth Flow

1. Coach submits signup or login credentials.
2. Service validates input and password.
3. Service returns JWT with user ID and role.
4. Other services validate the token before serving protected resources.

## User Roles

- `coach`: MVP primary user
- `athlete`: later athlete login support
- `admin`: future operational role

## Tables Owned

- `users`
- `coaches`
- Future: `refresh_tokens`, `password_resets`

## Endpoints

- `POST /auth/signup`
- `POST /auth/login`
- `GET /auth/me`
- `POST /auth/refresh`
- `POST /auth/logout`

## Environment Variables

- `APP_NAME`
- `ENVIRONMENT`
- `DATABASE_URL`
- `JWT_SECRET`
- `JWT_EXPIRES_MINUTES`

## Local Run Steps

```bash
pip install -r requirements.txt
uvicorn main:app --reload
```

## Testing Plan

- Signup validation
- Duplicate email rejection
- Password hashing verification
- Login success and failure
- JWT claim generation
- Protected route access

## Future Improvements

- Refresh tokens
- Password reset
- Email verification
- OAuth login
- Athlete self-service login
