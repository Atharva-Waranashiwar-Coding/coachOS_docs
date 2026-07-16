# Infrastructure

## Responsibility

The infra repository owns Docker Compose orchestration, edge Nginx, environment contracts, monitoring, log aggregation, database backup and restore, and deployment automation. It remains cloud-neutral and does not contain Kubernetes, Docker Swarm, Redis, Kafka, or RabbitMQ deployment dependencies.

## Docker Compose

`docker-compose.dev.yml` builds sibling repositories and publishes developer ports. `docker-compose.prod.yml` consumes prebuilt images, keeps service and database ports private, mounts persistent volumes, enables restart policies, and hardens application containers with read-only filesystems, dropped capabilities, non-root users, and `no-new-privileges`.

The topology contains:

- Nginx edge and React frontend
- Auth API and Auth PostgreSQL
- Athlete API and Athlete PostgreSQL
- Media API, outbox worker, Media PostgreSQL, and MinIO
- AI Review API, review worker, outbox worker, and AI Review PostgreSQL
- Prometheus, Grafana, Loki, Promtail
- Nginx and PostgreSQL exporters

## Service Networking

The edge proxy is the only public application entrypoint. Public, internal service, data, and monitoring networks separate traffic. Every service owns its own PostgreSQL database and credentials. Cross-service calls use internal DNS names, JWTs, or directional service tokens.

## Environment Management

`.env.example` documents the complete deployment contract. Real `.env` files are ignored. Required secrets include database passwords, JWT signing secret, directional internal tokens, MinIO credentials, Grafana credentials, and the AI provider key. Production image variables should use immutable release tags or digests.

## Deployment

`scripts/deploy.sh` validates configuration, creates pre-deployment database backups when the stack is running, pulls images, reconciles Compose, and waits for health. API entrypoints execute idempotent Alembic upgrades; workers disable migrations and wait for API readiness.

Production TLS files are mounted from `nginx/tls`. Self-signed generation exists only for local smoke tests.

## CI/CD

Application repositories lint, type-check, test, build, and publish their own images. Infra CI validates shell, Compose, Prometheus, alerts, and Grafana JSON. The deployment workflow is manual, environment-protected, and runs the deployment script over SSH on a generic Docker host.

## Monitoring

FastAPI services expose liveness, readiness, and Prometheus endpoints. Nginx and PostgreSQL use exporters. Prometheus alerts cover service/database availability, 5xx rate, and latency. Grafana receives provisioned Prometheus and Loki data sources and a CoachOS operations dashboard.

## Logging

Nginx propagates or creates `X-Request-ID`. Backend JSON logs include the request ID, service, logger, severity, timestamp, and structured request fields. Promtail discovers Docker containers through the read-only Docker socket and sends parsed logs to Loki.

## Backup And Recovery

Backup scripts create custom-format PostgreSQL dumps with SHA-256 sidecars for each database. Restore is explicitly guarded by `ALLOW_DESTRUCTIVE_RESTORE=true`, stops dependent services, verifies the checksum, restores with `--clean --if-exists`, and restarts services.

Backups must be copied to encrypted off-host storage. PostgreSQL dumps do not contain MinIO videos; object storage requires its own backup or replication policy.

## Future Improvements

- Automated certificate issuance and rotation
- PostgreSQL replication and point-in-time recovery
- MinIO replication or scheduled object backup
- Image vulnerability gates and signed artifacts
- Distributed tracing
- Multi-host deployment only when availability requirements justify added orchestration complexity
