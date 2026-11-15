# Rikkei DevOps - Session 05 / Exercise 02

QuickBite multi-service deployment with Docker Compose and the external
`quickbite-net` network.

## Project path mapping

The assignment path `/quickbite-project/` maps to this repository root:

`/home/white_ember/DevOps-Lab/md05_ss05_bai_02/`

## Services

| Compose service | Image | Host:container |
|---|---|---|
| `quickbite-user` | `quickbite-user-service:latest` | `8081:8081` |
| `quickbite-restaurant` | `quickbite-restaurant-service:latest` | `8082:8082` |
| `quickbite-order` | `quickbite-order-service:latest` | `8083:8083` |
| `quickbite-notification` | `quickbite-notification-service:latest` | `8084:8084` |

Compose project name: `rikkei-ss05-quickbite`.

All four services attach to the pre-existing external bridge network
`quickbite-net`. Compose declares it as `external: true`; it never creates or
deletes that network.

## Database

The databases are hosted by the external PostgreSQL container
`quickbite-db-net` (`postgres:15-alpine`). It reuses the existing Session 4
`quickbite-db` data volume and is attached to `quickbite-net` with the alias
`quickbite-db`. The stopped legacy `quickbite-db` container is left untouched.

The new DB container intentionally publishes no host port because the NixOS
`postgresql.service` already owns host port 5432. The services reach the
database only through `quickbite-net` at `quickbite-db:5432`.

Databases and owners:

| Database | Owner |
|---|---|
| `quickbite_user_db` | `quickbite_user` |
| `quickbite_restaurant_db` | `quickbite_restaurant` |
| `quickbite_order_db` | `quickbite_order` |
| `quickbite_notification_db` | `quickbite_notification` |

All four services use the standard Spring Boot datasource environment
variables (`SPRING_DATASOURCE_URL`, `SPRING_DATASOURCE_USERNAME`,
`SPRING_DATASOURCE_PASSWORD`). `quickbite-order` also has explicit Feign URLs
for the other three Compose services.

## Local setup

1. Create the untracked environment file and replace the password
   placeholders with local lab passwords:

   ```bash
   cp .env.example .env
   chmod 600 .env
   ```

2. Verify the external network exists:

   ```bash
   docker network inspect quickbite-net
   ```

3. Validate and start:

   ```bash
   docker compose config -q
   docker compose up -d
   docker compose ps
   ```

4. Health checks:

   ```bash
   curl -fsS http://localhost:8081/actuator/health
   curl -fsS http://localhost:8082/actuator/health
   curl -fsS http://localhost:8083/actuator/health
   curl -fsS http://localhost:8084/actuator/health
   ```

5. Stop only this Compose project:

   ```bash
   docker compose down
   ```

   The external network and the external database container are not removed.

## Evidence

`docker compose ps` evidence:

`homework/session_05/exercise_02/compose_up.png`

- PNG dimensions: `2506x693`
- SHA-256: `a1c3fdc055bb7c1ef3a2cd78dd328eb837cd774f6976a1b0e943f176be907ba8`

The local assignment path `/homework` is root-owned and not writable without
elevated privileges, so the screenshot is committed at the documented
repository-relative equivalent above.

## Credentials

`.env` is ignored by Git. The repository contains only placeholders in
`.env.example`; no real database password is committed.
