# Backend Structure Document

## 1. Backend Architecture

This project’s backend lives in a dedicated `apps/server` service within a pnpm monorepo. It uses Express.js with TypeScript to organize code into clear modules and follows an adapter-style pattern for extending functionality. Key points:

- Service‐oriented layout: one service for web (`apps/web`), one for API logic (`apps/server`), plus shared packages (`packages/*`).
- Design patterns:
  - Adapter pattern in `packages/cli-manager` to support multiple CLI tools.
  - Modular controllers, services, and middleware in Express.
- Scalability:
  - Each service runs in its own container, so you can scale the API independently of the frontend.
  - Database connections managed by Prisma’s connection pool.
- Maintainability:
  - A monorepo with shared packages (`ui`, `cli-manager`, `terminal`, etc.) avoids duplicated code.
  - TypeScript throughout ensures catch‐errors-early and consistent interfaces.
- Performance:
  - WebSockets for real‐time log streaming minimize polling overhead.
  - Redis-based rate limiting protects heavy endpoints.

## 2. Database Management

The backend uses two data stores to handle different needs:

- **PostgreSQL (SQL)**
  - Managed by Prisma ORM in `apps/server` for type-safe queries and schema migrations.
  - Stores core data: user accounts, sessions, CLI jobs, AI requests.
  - Migrations are defined in a central `schema.prisma` file and applied via `prisma migrate`.

- **Redis (NoSQL)**
  - Used for rate limiting on endpoints like `/api/ai/generate`.
  - Key‐value store for short‐lived rate‐limit counters.

Data practices:
- Regular backups of PostgreSQL (daily snapshots).
- Use of environment‐specific databases for dev, staging, and prod.
- Connection strings and credentials stored securely in environment variables or a secrets manager.

## 3. Database Schema

Below is a human‐readable summary of the PostgreSQL schema, followed by the SQL definitions.

Users and sessions:
- **users**: holds user identity and password data.
- **sessions**: tracks active user sessions or tokens.

CLI jobs and AI requests:
- **cli_jobs**: records each command execution request and its state.
- **ai_requests**: logs AI generation requests and responses.

Database schema (PostgreSQL):
```sql
-- Users table
CREATE TABLE users (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  email TEXT UNIQUE NOT NULL,
  password_hash TEXT NOT NULL,
  created_at TIMESTAMPTZ DEFAULT now(),
  updated_at TIMESTAMPTZ DEFAULT now()
);

-- Sessions table
CREATE TABLE sessions (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID REFERENCES users(id) ON DELETE CASCADE,
  token TEXT NOT NULL,
  expires_at TIMESTAMPTZ NOT NULL,
  created_at TIMESTAMPTZ DEFAULT now()
);

-- CLI Jobs table
CREATE TABLE cli_jobs (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID REFERENCES users(id) ON DELETE SET NULL,
  command TEXT NOT NULL,
  status TEXT NOT NULL CHECK (status IN ('pending','running','completed','failed','stopped')),
  exit_code INT,
  created_at TIMESTAMPTZ DEFAULT now(),
  started_at TIMESTAMPTZ,
  completed_at TIMESTAMPTZ
);

-- AI Requests table
CREATE TABLE ai_requests (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID REFERENCES users(id) ON DELETE SET NULL,
  prompt TEXT NOT NULL,
  result JSONB,
  created_at TIMESTAMPTZ DEFAULT now()
);
```  

## 4. API Design and Endpoints

The backend exposes RESTful endpoints and a WebSocket for real-time features:

- **Authentication** (if moved to server)
  - POST `/auth/sign-up` – create a new user.
  - POST `/auth/sign-in` – issue a session token.

- **CLI Management**
  - POST `/cli/run` – start a CLI job. Expects `{ command: string }`.
  - POST `/cli/stop/:id` – stop a running job by ID.
  - WebSocket `/cli/logs/:id` – stream stdout/stderr messages for job ID.

- **AI Generation**
  - POST `/api/ai/generate` – send a prompt, receive a generated result.
    • Protected by API key or session token.
    • Rate limiting enforced via Redis.

- **Health and Utility**
  - GET `/healthz` – simple up/healthy check.
  - GET `/metrics` – expose Prometheus metrics.

Each endpoint validates input with middleware, checks auth tokens, and returns clear JSON responses. Errors use an `{ error: string }` format.

## 5. Hosting Solutions

We recommend container-based deployment using a cloud provider:

- **Compute**
  - AWS ECS (Fargate) or Google Cloud Run to host Docker containers without managing servers.
- **Database**
  - AWS RDS for PostgreSQL or managed Cloud SQL.
- **Cache**
  - AWS ElastiCache (Redis) or managed Memorystore.

Benefits:
- **Reliability**: Managed services offer automatic failover and backups.
- **Scalability**: Fargate and serverless databases scale with demand.
- **Cost-effectiveness**: Pay‐as‐you‐go with no idle server costs.

## 6. Infrastructure Components

Key components working together:

- **Load Balancer**
  - AWS Application Load Balancer routes HTTP(S) to multiple `apps/server` instances.
- **CDN**
  - Vercel (for `apps/web`) or CloudFront caches static assets close to users.
- **Caching/Rate Limit**
  - Redis handles rate‐limit counters and can cache frequent queries if needed.
- **Docker Compose (Dev)**
  - Defines `web`, `server`, `postgres`, `redis`, and any sandbox‐manager service for local testing.

Together, these ensure fast responses, even under load, and maintain a consistent developer experience across environments.

## 7. Security Measures

Multiple layers of protection safeguard data and services:

- **Authentication & Authorization**
  - JWT or session tokens for API access.
  - Role‐based checks for sensitive endpoints.
- **Encryption**
  - TLS (HTTPS) in transit.
  - At-rest encryption for RDS and Redis.
- **Input Sanitization**
  - All CLI commands and AI prompts are validated and escaped to prevent injection.
- **Sandboxed Execution**
  - CLI commands run in isolated Docker containers with strict CPU and memory limits.
- **Rate Limiting**
  - Prevent abuse of `/api/ai/generate` via Redis counters.
- **Secrets Management**
  - Environment variables stored in a secrets manager (AWS Secrets Manager, Vault).

## 8. Monitoring and Maintenance

To keep the backend healthy and performant:

- **Logging**
  - Structured application logs sent to a centralized service (e.g., CloudWatch Logs, ELK).
- **Metrics**
  - Prometheus collects metrics (request rates, latencies, error counts).
  - Grafana dashboards visualize trends.
- **Error Tracking**
  - Sentry or a similar tool captures uncaught exceptions and alerts the team.
- **CI/CD**
  - GitHub Actions runs linting, TypeScript checks, unit tests (Jest), and end-to-end tests (Playwright).
  - Deploys `apps/web` to Vercel and `apps/server` containers to ECS automatically on merge.
- **Maintenance**
  - Scheduled database backups and periodic dependency updates.
  - Automated canary releases for safe rollouts.

## 9. Conclusion and Overall Backend Summary

This backend setup delivers a clear separation of concerns, with an Express.js API service handling business logic, a type‐safe PostgreSQL database via Prisma, and Redis for caching and rate‐limiting. Containerization and cloud hosting ensure reliable, scalable deployments, while shared packages in a pnpm monorepo drive maintainability and code reuse. Real-time features powered by WebSockets and sandboxed Docker execution enable interactive CLI and AI experiences. Together, these components align with the project’s goal of a flexible, performant developer platform without sacrificing security or developer ergonomics.