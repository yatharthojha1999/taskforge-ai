# TaskForge AI

A multi-tenant task management platform built with independently deployable microservices, with AI-powered capabilities planned as the platform evolves.

## Repositories

## Microservices

| Service | Responsibility | Repository |
|---|---|---|
| User Service | Tenant management, user profiles, and user APIs | [taskforge-user-service](https://github.com/yatharthojha1999/taskforge-user-service) |
| Auth Service | Credentials, login, JWTs, and refresh-token sessions | Planned |

Each microservice is maintained in its own GitHub repository with independent code, dependencies, migrations, and deployment configuration.

## Architecture

Each microservice has its own GitHub repository, application code, dependencies, configuration, database migrations, and README.

## Technology Stack

- Node.js and Express
- PostgreSQL
- Kafka and Redis *(planned)*
- Python and FastAPI for future AI capabilities
- Docker and Kubernetes *(planned)*

## Development Status

The User Service is under development. Other services and infrastructure components will be added incrementally.

See each service's repository README for its setup instructions and implemented functionality.