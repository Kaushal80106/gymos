# GymOS: agent instructions

Product spec: `docs/PRD.md` (source of truth). Read the relevant section before each task.

## Stack
- Monorepo: pnpm + Turborepo. `apps/web` (Next.js App Router, TypeScript), `apps/api` (NestJS), `apps/worker` (NestJS standalone + BullMQ), `packages/shared` (Zod schemas, types, constants), `packages/ui`.
- PostgreSQL 16 with Prisma. Redis 7 (BullMQ, rate limiting, cache, idempotency).
- Frontend: Tailwind, shadcn/ui, TanStack Query, React Hook Form + Zod.
- Local dev: docker-compose for Postgres and Redis.

## Rules
- Multi-tenant: every tenant table has `tenant_id`. Enforce isolation with Postgres row-level security AND tenant checks in the API. Set the tenant per request with `SET LOCAL app.tenant_id`.
- Money safety: payments are append-only; corrections go through reversals. Money endpoints require an `Idempotency-Key`. Use integer paise for amounts, never floats.
- Business logic (renewal dates, freezes, status derivation, ageing buckets, reminder eligibility) lives in pure functions in `packages/shared` with unit tests.
- Dates: store UTC; compute expiry and reminder windows in the branch timezone (default `Asia/Kolkata`).
- Validate all input with Zod/class-validator. Return errors as `{ code, message, details }`.
- Never hardcode secrets. Use `.env.example` and read config through a typed config module.
- WhatsApp and payment providers sit behind interfaces with a mock implementation used in dev and tests.
- Style: strict TypeScript, no `any`, small modules, conventional commits.

## Definition of done (every task)
1. `pnpm lint`, `pnpm typecheck` and `pnpm test` pass.
2. New behaviour has tests (unit for logic, integration for API with a real Postgres/Redis).
3. Migrations are included and run cleanly from scratch.
4. README or docs updated if setup or commands changed.
5. Summarise what changed, what was not done, and any assumptions.