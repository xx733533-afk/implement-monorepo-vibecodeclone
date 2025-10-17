# Project Requirements Document (PRD)

## 1. Project Overview

VibeCode-Clone is a multi-service developer platform built on a monorepo structure. Its `apps/web` component provides a full-featured, authenticated web UI where developers can edit code, run CLI commands, and view real-time output. Behind the scenes, an `apps/server` service handles sandboxed command execution, AI integrations, and database operations. Shared packages like `ui`, `cli-manager`, and `terminal` ensure consistent tooling and design across the entire system.

We’re building this platform to give developers a one-stop environment for coding, experimenting with AI assistants (like GPT or Claude), and executing commands securely in Docker sandboxes. Key objectives for version one include: 1) a smooth sign-up/sign-in flow, 2) a responsive dashboard with editor and terminal components, 3) reliable back-end endpoints to spawn and manage isolated CLI processes, 4) real-time log streaming via WebSockets, and 5) a clear, maintainable monorepo setup with tests and CI/CD pipelines.

## 2. In-Scope vs. Out-of-Scope

**In-Scope** (v1 deliverables):
- Monorepo setup using pnpm workspaces.
- `apps/web`: Next.js + React + TypeScript frontend with Better Auth for user auth.
- `apps/server`: Express.js + TypeScript backend with Prisma ORM + PostgreSQL + Redis rate-limiting.
- Shared packages under `packages/`:
  - `ui` (shadcn/ui + Tailwind CSS + next-themes)
  - `cli-manager` (Docker sandboxing and CLI adapter logic)
  - `terminal` (WebSocket-powered log streaming component)
- API endpoints:
  - POST `/cli/run` to start a sandboxed process.
  - GET/WebSocket `/cli/logs/:id` for real-time output.
  - POST `/cli/stop/:id` to terminate processes.
- Docker Compose configuration for all services (Postgres, Redis, web, server).
- Automated tests: Jest (unit/integration) and Playwright (E2E).
- CI/CD via GitHub Actions and Vercel deployment for `apps/web`.

**Out-of-Scope** (planned for later phases):
- Mobile clients or native apps.
- Advanced analytics dashboards or team collaboration features.
- Third-party plugin marketplace.
- Complex sandbox orchestration beyond Docker containers (e.g., Kubernetes).
- Support for every CLI tool—initial focus on AI adapters (OpenAI, Claude) and core shell commands.

## 3. User Flow

A new developer visits the VibeCode-Clone site and lands on the sign-in page. They choose “Sign up,” enter their email and password, and submit. Behind the scenes, the request goes to `apps/server`’s auth adapter (Prisma + PostgreSQL). On success, the frontend redirects the user to the main dashboard. The dashboard features a left sidebar (navigation), a top header (user profile, theme toggle), and a central panel displaying a code editor and integrated terminal.

Within the dashboard, the developer writes or pastes code in the editor. To run a command, they switch to the terminal tab, type something like `generate-docs myFile.md`, and hit Run. The frontend sends a POST to `/cli/run`, then opens a WebSocket connection to `/cli/logs/:id`. Logs stream back in real time and appear in the terminal UI. If they click “Stop,” the frontend POSTs to `/cli/stop/:id`, and the backend safely kills the Docker process. The user can toggle between dark and light modes at any time.

## 4. Core Features

- **Authentication**: Sign up, sign in, session management via Better Auth/Prisma.
- **Dashboard Layout**: Sidebar, header, and main content area with editor and terminal.
- **Code Editor**: Integrated editor component (e.g., Monaco) for writing code.
- **Terminal & Log Streaming**: WebSocket client in `packages/terminal` for real-time stdout/stderr.
- **CLI Manager**: `packages/cli-manager` spawns Docker sandboxes, auto-installs binaries, abstracts multiple CLI adapters.
- **API Endpoints**: `/cli/run`, `/cli/logs/:id`, `/cli/stop/:id` in `apps/server`.
- **Shared UI Library**: Central `packages/ui` based on shadcn/ui and Tailwind.
- **Database Layer**: Prisma ORM with PostgreSQL, Redis for rate-limiting.
- **Monorepo Orchestration**: pnpm workspaces, shared tsconfig, ESLint/Prettier.
- **Containerization**: Docker Compose for multi-service orchestration.
- **Testing & CI/CD**: Jest, Playwright, GitHub Actions, Vercel deploy.

## 5. Tech Stack & Tools

- Frontend: Next.js (App Router), React, TypeScript, Tailwind CSS, shadcn/ui, next-themes.
- Backend: Node.js, Express.js, TypeScript, Prisma ORM, PostgreSQL, Redis.
- Monorepo: pnpm workspaces, root tsconfig with project references, shared ESLint/Prettier configs.
- Containerization: Docker, Docker Compose.
- Real-time: WebSockets (ws or built-in Next.js API routes).
- CLI Sandboxing: Dockerode or Docker CLI via `packages/cli-manager`.
- Testing: Jest (unit/integration), Playwright (E2E).
- CI/CD: GitHub Actions (pnpm filtering), Vercel for frontend.
- AI Integration (future): OpenAI GPT-4, Anthropic Claude SDKs.
- IDE Plugins (optional): Cursor, Windsurf for faster AI-driven development.

## 6. Non-Functional Requirements

- **Performance**: Backend API responses under 200ms; terminal streaming latency under 100ms.
- **Scalability**: Support 100+ concurrent sandboxed processes; horizontal scaling via Docker Compose forks or future container orchestrator.
- **Security**: HTTPS everywhere, JWT/session protection, strict Docker sandbox limits (CPU, memory, time), input sanitization.
- **Reliability**: 99.9% uptime goal, automatic process cleanup on server restart.
- **Usability**: WCAG-friendly UI, dark/light modes, responsive design.
- **Compliance**: GDPR-compatible user data handling, encryption at rest for sensitive data.

## 7. Constraints & Assumptions

- Requires Docker and Docker Compose installed in dev/production.
- Node.js >= 18, pnpm package manager.
- Availability of external AI APIs (OpenAI/Claude) and valid API keys.
- Users run modern browsers with WebSocket support.
- Redis instance mandatory for rate-limiting.
- Monorepo build times manageable with pnpm caching and selective builds.

## 8. Known Issues & Potential Pitfalls

- **Docker-in-Docker Overhead**: Contain sandboxes safely but watch performance. Mitigate with resource limits and lightweight base images.
- **WebSocket Stability**: Handle reconnections, dropped messages. Implement exponential backoff and client-side buffering.
- **ORM Migration**: Drizzle → Prisma schema drift. Keep schema definitions in sync and maintain migration scripts.
- **API Rate Limits**: External AI calls may hit quotas. Add caching, request batching, and graceful degradation.
- **Monorepo Complexity**: Large repos slow CI. Use pnpm’s `--filter` and GitHub Actions caching to speed up builds.

---
This PRD lays out the functional and non-functional scope of the VibeCode-Clone platform’s first release. Each section contains enough detail for an AI or development team to generate precise technical docs (Tech Stack, Frontend Guidelines, Backend Structure, etc.) without ambiguity.