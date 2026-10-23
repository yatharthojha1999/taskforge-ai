TaskForge AI
TaskForge AI is a multi-service task-management application under active
development. The current backend includes a User Service and an Auth
Service, with PostgreSQL used for persistence. The frontend is a
React/Vite application.
Project status: The Dockerized User Service and Auth Service have
been started successfully, and registration, login, and logout APIs
have been tested. Other functionality may still be in development;
check each service's README and source code for the current
implementation.

Repository layout
taskforge-ai/
├── taskForge-ai-users/       # User and tenant management API
├── taskforge-auth-service/   # Authentication and session API
├── taskforge-frontend/       # React/Vite frontend
├── compose.yaml              # Local Docker Compose stack
└── .env                      # Local secrets/configuration (do not commit)
Architecture
- Frontend: React/Vite, typically served locally at
  http://localhost:5173.
- User Service: Node.js and Express, exposed on host port 3000.
- Auth Service: Node.js and Express, exposed on host port 3001.
- PostgreSQL: One PostgreSQL container exposed on host port
  5433; the services use separate databases.
Inside the Compose network, services communicate using Compose service
names rather than localhost:
- User Service → Auth Service: http://auth-service:3001
- Auth Service → User Service: http://user-service:3000
Within a container, localhost refers to that same container. The
application configuration reads AUTH_SERVICE_URL and
USER_SERVICE_URL from the environment and keeps localhost defaults for
local, non-Docker development.
Prerequisites
- Docker Desktop with Docker Compose
- Node.js and npm if running the frontend or a backend service
  directly on the host
- A local .env file at the project root
Configuration
Create a root .env file and set the PostgreSQL password:
POSTGRES_PASSWORD=replace-with-a-strong-local-development-password
Do not commit .env, passwords, private keys, access tokens, or refresh
tokens. If a development password has been exposed, rotate it and update
the local configuration.

The Compose file configures these database names:

  Service        Database
  User Service   taskforge_users
  Auth Service   taskforge_auth

The PostgreSQL image's POSTGRES_DB: taskforge creates the initial
database only when the data directory is initialized. The two service
databases are separate databases and must exist before their migration
jobs run. With the current setup, create them manually if they do not
already exist:
docker compose exec postgres psql -U taskforge -d postgres -c "CREATE DATABASE taskforge_users;"
docker compose exec postgres psql -U taskforge -d postgres -c "CREATE DATABASE taskforge_auth;"
If PostgreSQL reports that a database already exists, verify it rather
than trying to recreate it. These commands do not delete the existing
taskforge database.
Start the backend stack
From the project root:
docker compose up -d postgres
docker compose ps
If the two service databases do not exist, create them using the
commands above. Then run migrations separately:
docker compose --profile migrations run --build --rm user-migrate
docker compose --profile migrations run --build --rm auth-migrate
Start or rebuild the services:
docker compose up -d --build user-service auth-service
docker compose ps
View logs:
docker compose logs --tail=80 user-service auth-service
docker compose logs -f user-service auth-service
Inspect the effective service-to-service URLs:
docker compose exec user-service printenv AUTH_SERVICE_URL
docker compose exec auth-service printenv USER_SERVICE_URL
Expected values:
http://auth-service:3001
http://user-service:3000
Database tables
After successful migrations, the expected tables are:
- taskforge_users: tenants, users, pgmigrations
- taskforge_auth: user_credentials, auth_sessions,
  pgmigrations
Verify them with:
docker compose exec postgres psql -U taskforge -d taskforge_users -c "\dt"
docker compose exec postgres psql -U taskforge -d taskforge_auth -c "\dt"
Avoid docker compose down -v unless you intentionally want to delete
the Compose-managed persistent volume and its data.
Run the frontend
The frontend can run locally while the backend runs in Docker:
cd .\taskforge-frontend
npm install
npm run dev
Open the Vite URL printed by the command (commonly
http://localhost:5173). Browser-side API requests should use
host-accessible URLs such as http://localhost:3000 and
http://localhost:3001, unless a Vite proxy or frontend environment
configuration is set up.
API overview
The following routes are known to be implemented in the current project.
Confirm request schemas and response examples in the
route/controller/validator code before relying on them.
  Service           Method            Endpoint                                                 Purpose
  User Service      POST            /v1/tenants/register                                   Register a tenant
                                                                                               and its initial
                                                                                               user

  User Service      GET             /internal/v1/tenants/:linkname/users/by-email/:email   Internal user
                                                                                               lookup used by
                                                                                               Auth Service

  Auth Service      POST            /internal/v1/credentials                               Internal
                                                                                               credential
                                                                                               creation used by
                                                                                               User Service

  Auth Service      POST            /auth/v1/login                                         Authenticate a
                                                                                               user

  Auth Service      POST            /auth/v1/refresh                                       Refresh/rotate a
                                                                                               session token

  Auth Service      POST            /auth/v1/logout                                        Log out/revoke a
                                                                                           refresh session
The User Service registration flow calls the Auth Service to create
credentials. A user is kept in a pending state until credential creation
succeeds, according to the implemented registration flow. Registration
and login depend on the two services being able to reach each other.
Troubleshooting
A service cannot reach another service
Inside Docker, use http://auth-service:3001 and
http://user-service:3000, not localhost for cross-container calls.
Verify the environment variables and ensure the app's config reads them.
A database does not exist
Create taskforge_users and taskforge_auth in the Compose PostgreSQL
server, then run each service's migration job. Do not delete the volume
as a troubleshooting shortcut.
A container is restarting
docker compose ps
docker compose logs --tail=100 user-service
docker compose logs --tail=100 auth-service
Resolve the first current error in the logs before repeatedly
rebuilding.
Development notes
- Each backend service owns its database and migration history.
- Keep secrets and private JWT key material out of version control.
- Update this README and the relevant service README whenever an
  endpoint, migration, environment variable, setup command, or test
  workflow changes.
- Document implemented functionality separately from planned
  functionality.