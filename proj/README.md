# Energy Management Platform – Request-Reply Assignment

This repository implements the **Distributed Systems – Assignment 1** requirements for an Energy Management System built with a request-reply communication paradigm. The solution is composed of containerised microservices exposed through a Spring Cloud Gateway reverse proxy and consumed by a React frontend.

## Architecture Overview

- **Auth Service (`backend/auth-service`)** – Spring Boot 3 / PostgreSQL. Handles user registration & login, issues JWT tokens with role and user id claims.
- **User Service (`backend/user-service`)** – Spring Boot 3 / PostgreSQL. Provides CRUD operations over platform users (email, full name, role).
- **Device Service (`backend/device-service`)** – Spring Boot 3 / PostgreSQL. Manages devices (name, maximum consumption) and their assignment to users.
- **API Gateway (`backend/api-gateway`)** – Spring Cloud Gateway. Validates JWT tokens, enforces role-based routing and forwards authenticated traffic to the internal services.
- **Frontend (`frontend`)** – React single-page application with role-aware navigation:
  - Admins: login, CRUD on users and devices, assign/unassign devices.
  - Clients: login and view devices assigned to their account.
- **Docker (`backend/docker-compose.yml`)** – Spins up the three microservices, their dedicated PostgreSQL databases and the API gateway. The frontend runs separately with `npm start` but targets the gateway at `http://localhost:8080`.

A UML deployment diagram describing the containers and communication channels is available at `docs/deployment-diagram.puml`.

## Repository Layout

```
backend/
  auth-service/
  user-service/
  device-service/
  api-gateway/
  docker-compose.yml
frontend/
docs/
  deployment-diagram.puml
README.md
```

## Running the Platform

### 1. Backend & Gateway via Docker Compose

```powershell
cd backend
docker compose up --build
```

Exposed ports:

| Service        | Port |
|----------------|------|
| API Gateway    | 8080 |
| Auth Service   | 8083 |
| User Service   | 8081 |
| Device Service | 8082 |
| PostgreSQL DBs | 5433–5435 (mapped for inspection) |

Environment variables (JWT secret, DB credentials) can be adjusted in `backend/docker-compose.yml`.

### 2. Frontend

```powershell
cd frontend
npm install
npm start
```

The React app runs on `http://localhost:3000` and communicates with the gateway at `http://localhost:8080` (configured in `src/commons/hosts.js`).

## Manual Service Execution (without Docker)

Each Spring Boot service bundles the Maven Wrapper. From its folder run:

```powershell
.\mvnw.cmd spring-boot:run
```

Default datasource settings live in `src/main/resources/application-local.yml`/`.properties`; override them through environment variables or JVM args when needed.

## Testing

Automated tests run with:

```powershell
cd backend/auth-service   ; .\mvnw.cmd test
cd ../user-service        ; .\mvnw.cmd test
cd ../device-service      ; .\mvnw.cmd test
cd ../api-gateway         ; .\mvnw.cmd test
```

Frontend tests are available via `npm test` (Create React App tooling).

## Feature Matrix vs. Requirements

| Requirement                                       | Implementation                                               |
|---------------------------------------------------|--------------------------------------------------------------|
| Auth microservice with JWT                        | `backend/auth-service`                                       |
| User CRUD microservice                            | `backend/user-service`                                       |
| Device CRUD + assignment microservice             | `backend/device-service`                                     |
| Reverse proxy + API gateway                       | `backend/api-gateway` (Spring Cloud Gateway, JWT filter)     |
| Docker deployment                                  | `backend/docker-compose.yml`                                 |
| Admin frontend (users, devices, assignment)       | `frontend/src/components/admin/AdminDashboard.js`            |
| Client frontend (view assigned devices)           | `frontend/src/components/client/ClientDashboard.js`          |
| Role-based login UI                               | `frontend/src/components/auth/LoginPage.js` + context        |
| Documentation + deployment diagram                | `README.md`, `docs/deployment-diagram.puml`                  |

## API Quick Reference (Gateway paths)

- `POST /api/auth/login` – Obtain JWT token (`token`, `role`, `username`, `userId`).
- `POST /api/auth/register` – Create platform user credentials (primarily for bootstrap).
- `GET /api/users` – List users (ADMIN).
- `POST /api/users` – Create user profile (ADMIN).
- `PUT /api/users/{id}` / `DELETE /api/users/{id}` – Update / delete user (ADMIN).
- `GET /api/devices` – List devices (ADMIN) or devices filtered by `?userId=` for clients.
- `POST /api/devices` / `PUT /api/devices/{id}` / `DELETE /api/devices/{id}` – Device CRUD (ADMIN).
- `POST /api/devices/{id}/assign` – Body `{ "userId": <id> }`, assign device (ADMIN).
- `POST /api/devices/{id}/unassign` – Unassign device (ADMIN).

All non-auth routes require `Authorization: Bearer <token>` and roles enforced by the gateway.

## Default Credentials

Flyway creates a bootstrap admin account in `auth-service`:

| Username | Password  | Role  |
|----------|-----------|-------|
| admin    | admin123  | ADMIN |

After authenticating, use the admin dashboard to create additional clients and assign devices.

---
For any change requests or deployment adjustments, update both the docker compose file and the service-specific `application-*.yml`/`.properties` files to keep environments in sync.

