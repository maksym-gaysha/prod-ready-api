# Production-Ready REST API — Reference Architecture

A showcase backend API demonstrating production-grade engineering practices using **Node.js (ES Modules)**, **Express 5**, **Neon Serverless PostgreSQL**, and **Drizzle ORM**. Engineered with enterprise security best practices using **Arcjet**, role-based access control (RBAC), structured logging with **Winston**, automated testing with **Jest**, and automated CI/CD workflows with **GitHub Actions** and **Docker**.

---

## Table of Contents

- [Features](#features)
- [Tech Stack](#tech-stack)
- [Project Architecture](#project-architecture)
- [Prerequisites](#prerequisites)
- [Environment Variables](#environment-variables)
- [Quick Start](#quick-start)
- [Database Management](#database-management)
- [API Reference](#api-reference)
  - [System & Health](#system--health)
  - [Authentication](#authentication)
  - [Users](#users)
- [Security & Rate Limiting](#security--rate-limiting)
- [Testing & Quality](#testing--quality)
- [Docker & Containerization](#docker--containerization)
- [CI/CD Workflows](#cicd-workflows)

---

## Features

- **Robust Architecture**: Modular MVC structure using native ECMAScript Modules (ESM) and subpath imports (`#config/*`, `#controllers/*`, `#routes/*`, etc.).
- **Serverless PostgreSQL**: Scalable database powered by [Neon](https://neon.tech/) with [Drizzle ORM](https://orm.drizzle.team/) for type-safe queries and schema migrations.
- **Advanced Security & Protection**:
  - **Arcjet Integration**: Shield attack protection, bot detection, and role-based sliding window rate limiting (Guest / User / Admin).
  - **Helmet & CORS**: Essential HTTP security headers and configurable CORS policies.
  - **Secure Authentication**: JWT-based session handling stored in secure HTTP-only cookies with bcrypt password hashing.
  - **Role-Based Access Control (RBAC)**: Endpoint-level permission guards for `admin` and `user` roles.
- **Input Validation**: Strict schema validation using **Zod**.
- **Observability & Logging**: Centralized logging with **Winston** with rotation support and HTTP request logging via **Morgan**.
- **Automated Testing**: Integration and unit testing with **Jest** and **Supertest**.
- **Docker Support**: Multi-stage production and development Dockerfiles and Compose configurations.
- **CI/CD Pipelines**: Automated GitHub Actions for linting/formatting, test execution with ephemeral Neon database branches, and container image builds/pushes.

---

## Tech Stack

| Layer | Technology |
|---|---|
| **Runtime & Framework** | Node.js (v20+), Express 5 |
| **Database & ORM** | Neon PostgreSQL Serverless, Drizzle ORM, Drizzle Kit |
| **Security & Guard** | Arcjet (`@arcjet/node`), Helmet, bcrypt, jsonwebtoken |
| **Validation** | Zod |
| **Logging** | Winston, Morgan |
| **Testing** | Jest, Supertest |
| **Tooling & Code Style** | ESLint, Prettier |
| **DevOps & Containers** | Docker, Docker Compose, GitHub Actions |

---

## Project Architecture

```
prod-ready-api/
├── .github/
│   └── workflows/
│       ├── docker-build-and-push.yml   # Docker build and registry push
│       ├── lint-and-format.yml         # ESLint and Prettier checks
│       └── tests.yml                   # CI test suite with Neon ephemeral DB
├── drizzle/                            # Generated database migrations
├── scripts/
│   ├── dev.sh                          # Development container startup script
│   └── prod.sh                         # Production container startup script
├── src/
│   ├── config/                         # Configuration (Arcjet, DB, Logger)
│   ├── controllers/                    # Route controllers (Auth, Users)
│   ├── middleware/                     # Express middlewares (Auth, Security, Error)
│   ├── models/                         # Drizzle schema definitions
│   ├── routes/                         # Express API route declarations
│   ├── services/                       # Business logic and database operations
│   ├── utils/                          # Shared helper utilities (JWT, Cookies)
│   ├── validations/                    # Zod validation schemas
│   ├── app.js                          # Express application setup
│   ├── index.js                        # Application entry point
│   └── server.js                       # HTTP server bootstrap
├── tests/                              # Integration & unit test suites
├── docker-compose.dev.yml              # Local Docker compose configuration
├── docker-compose.prod.yml             # Production Docker compose configuration
├── Dockerfile                          # Multi-stage Dockerfile
├── drizzle.config.js                   # Drizzle ORM configuration
├── eslint.config.js                    # ESLint configuration
└── package.json
```

---

## Prerequisites

- **Node.js**: `v20.x` or higher
- **npm**: `v10.x` or higher
- **PostgreSQL Database** (e.g., [Neon Serverless Postgres](https://neon.tech))
- **Arcjet Account & API Key**: [arcjet.com](https://arcjet.com/) (for rate limiting and bot detection)
- **Docker** *(optional, for containerized execution)*

---

## Environment Variables

Create a `.env` file in the root directory by copying `.env-example`:

```bash
cp .env-example .env
```

Populate the required environment variables:

```env
# Server Configuration
PORT=3000
NODE_ENV=development
LOG_LEVEL=info

# Database Configuration
DATABASE_URL=postgresql://<user>:<password>@<host>/<database>?sslmode=require

# Arcjet Security
ARCJET_KEY=ajkey_your_arcjet_api_key

# Neon API (Used in CI/CD ephemeral branch testing)
NEON_API_KEY=your_neon_api_key
NEON_PROJECT_ID=your_neon_project_id
PARENT_BRANCH_ID=your_neon_parent_branch_id

# JWT Authentication
JWT_SECRET=your_super_secret_jwt_key
JWT_EXPIERES_IN=1d
```

---

## Quick Start

### 1. Install Dependencies

```bash
npm install
```

### 2. Run Database Migrations

Generate and apply the Drizzle ORM migrations to your database:

```bash
npm run db:generate
npm run db:migrate
```

### 3. Start the Development Server

Starts the server with Node's built-in file watcher:

```bash
npm run dev
```

The API will be accessible at: `http://localhost:3000`

### 4. Start in Production Mode

```bash
npm start
```

---

## Database Management

This project uses [Drizzle ORM](https://orm.drizzle.team/) with Drizzle Kit for type-safe schema modeling and migrations.

| Command | Description |
|---|---|
| `npm run db:generate` | Generate SQL migration files based on schema changes in `src/models/` |
| `npm run db:migrate` | Apply pending migrations to the target database |
| `npm run db:studio` | Launch Drizzle Studio web GUI to browse and edit database records |

---

## API Reference

### System & Health

| Method | Endpoint | Access | Description |
|---|---|---|---|
| `GET` | `/health` | Public | Returns service health, uptime, and current timestamp. |
| `GET` | `/` | Public | Returns server status message. |
| `GET` | `/api` | Public | API root status check. |

---

### Authentication

Base path: `/api/auth`

#### 1. Sign Up
- **Endpoint**: `POST /api/auth/sign-up`
- **Access**: Public
- **Request Body**:
  ```json
  {
    "name": "John Doe",
    "email": "john@example.com",
    "password": "SecurePassword123!",
    "role": "user"
  }
  ```
- **Response** `(201 Created)`: Returns created user details (excluding password) and sets an HTTP-only JWT cookie.

#### 2. Sign In
- **Endpoint**: `POST /api/auth/sign-in`
- **Access**: Public
- **Request Body**:
  ```json
  {
    "email": "john@example.com",
    "password": "SecurePassword123!"
  }
  ```
- **Response** `(200 OK)`: Sets the `token` cookie containing the JWT payload and returns user info.

#### 3. Sign Out
- **Endpoint**: `POST /api/auth/sign-out`
- **Access**: Public
- **Response** `(200 OK)`: Clears the authentication cookie.

---

### Users

Base path: `/api/users`

#### 1. List All Users
- **Endpoint**: `GET /api/users`
- **Access**: Authenticated (Requires `admin` role)
- **Response** `(200 OK)`: Array of user objects.

#### 2. Get User by ID
- **Endpoint**: `GET /api/users/:id`
- **Access**: Authenticated
- **Response** `(200 OK)`: User profile object.

#### 3. Update User by ID
- **Endpoint**: `PUT /api/users/:id`
- **Access**: Authenticated (User can update own profile; `admin` can update any profile)
- **Request Body**:
  ```json
  {
    "name": "Jane Doe",
    "email": "jane@example.com"
  }
  ```
- **Response** `(200 OK)`: Updated user profile.

#### 4. Delete User by ID
- **Endpoint**: `DELETE /api/users/:id`
- **Access**: Authenticated (Requires `admin` role)
- **Response** `(200 OK)`: Confirmation message.

---

## Security & Rate Limiting

The API is fortified with **Arcjet** middleware that dynamically enforces security rules per client role:

- **Shield Protection**: Detects and mitigates common web attacks (SQLi, XSS, suspicious payloads).
- **Bot Detection**: Blocks automated crawlers and malicious bots.
- **Role-Based Sliding Window Rate Limiting**:
  - `admin`: 20 requests / minute
  - `user`: 10 requests / minute
  - `guest`: 100 requests / minute (or customized threshold)

---

## Testing & Quality

### Run Unit and Integration Tests
```bash
npm test
```

### Code Linting & Formatting
```bash
# Check lint errors
npm run lint

# Automatically fix lint errors
npm run lint:fix

# Format code with Prettier
npm run format

# Check formatting compliance
npm run format:check
```

---

## Docker & Containerization

### Development Container
Starts the app inside a container with hot-reloading:
```bash
npm run dev:docker
# Or directly with compose:
docker compose -f docker-compose.dev.yml up --build
```

### Production Container
Builds and runs the optimized production image:
```bash
npm run prod:docker
# Or directly with compose:
docker compose -f docker-compose.prod.yml up -d --build
```

---

## CI/CD Workflows

GitHub Actions workflows are preconfigured under `.github/workflows/`:

1. **Lint and Format (`lint-and-format.yml`)**: Runs on pull requests and pushes to `main` to verify code quality with ESLint and Prettier.
2. **Automated Testing with Neon DB (`tests.yml`)**: Creates an ephemeral Neon PostgreSQL database branch on the fly, runs database migrations and the Jest test suite, and cleans up the branch upon completion.
3. **Docker Build & Push (`docker-build-and-push.yml`)**: Automatically builds, tags, and pushes multi-platform Docker container images to DockerHub or GitHub Container Registry on releases/pushes.
