# Webcup Architecture Baseline (build during J-7)

Goal: arrive at H0 with a deployed, secure, subject-agnostic skeleton so the 24h go to features, not setup. The subject is unknown before H0, so everything here must be generic.

## Stack selection rules

- Pick what the team already masters. Speed of the team beats theoretical best stack.
- Must run on the provided server (HODi). Verify runtime versions, DB availability, reverse proxy, HTTPS, ports, and disk/RAM limits on day J-7.
- Prefer one full-stack framework with server-side rendering or a clear API layer (e.g. Next.js/Nuxt/SvelteKit + ORM, Laravel, Django, Rails, NestJS + SPA). Fewer moving parts = fewer failures at 4 a.m.
- Relational DB (PostgreSQL or MySQL) by default: relations, constraints, and transactions help both coherence and security.

## Target folder structure (adapt to framework)

```
src/
  modules/<entity>/        # routes/controller, service, repository, schema, dto
  middleware/              # auth, role, validate, rateLimit, errorHandler, requestId
  lib/                     # db, cache, logger, ai (OpenRouter proxy), mailer
  ui/components/           # design-system components with loading/empty/error states
  ui/pages/
prisma|migrations/
scripts/seed.ts            # jury accounts per role + demo data
FEATURES.md                # live feature status = final recap deliverable
```

## Pre-built before H0 (checklist)

**Deploy**
- [ ] Repo, `main` protected by habit, `.gitignore` with `.env`.
- [ ] One-command deploy to the provided server (script or CI). Tested twice.
- [ ] HTTPS, domain, HTTP→HTTPS redirect.
- [ ] Production env vars set on server; debug off.
- [ ] Process manager / container restart on crash.

**Backend core**
- [ ] DB connection, migrations, seed script.
- [ ] Auth module: register, login, logout, me; argon2id/bcrypt; secure cookie session; session rotation.
- [ ] Roles: `user`, `admin` (+ placeholder for a third role); role guard middleware.
- [ ] Ownership helper: `assertOwner(resource, user)` reused by every module.
- [ ] Validation middleware with a schema library.
- [ ] Central error handler + JSON error contract `{ error: { code, message, fields? } }`.
- [ ] Rate limiter (login, register, reset, AI, generic API).
- [ ] Security headers (e.g. helmet or framework equivalent), CORS allowlist, CSRF.
- [ ] Structured logger with request id; `/health` returns DB status.
- [ ] Pagination helper (cursor or offset with max limit).
- [ ] Cache helper (in-memory or Redis) with `getOrSet(key, ttl, fn)` and `invalidate(prefix)`.
- [ ] AI proxy (`lib/ai`): OpenRouter call server-side, timeout, fallback model list, response cache, 429 handling, dev mock.

**Frontend core**
- [ ] Layout, navigation, auth pages, protected route handling.
- [ ] Design tokens (colors, type scale, spacing), dark/light if cheap.
- [ ] Components: button, input with error, select, table/list with pagination, card, modal, toast, skeleton loader, empty state, error boundary.
- [ ] 404 and 500 pages.
- [ ] Responsive check at 390px.

**Quality**
- [ ] Lint + typecheck + minimal tests run in one command.
- [ ] Dependency audit clean.

## Patterns to reach for during the 24h

| Need | Pattern |
|---|---|
| Hot read endpoint (dashboard, listings) | Cache-aside with TTL, invalidate on write |
| Lists | Pagination + index on sort/filter columns |
| Duplicate-prone writes (orders, votes, submissions) | Unique constraint and/or idempotency key |
| Multi-step write | DB transaction |
| Slow work (emails, AI, exports) | Background job or async with status polling; never block the request long |
| External API / AI | Server-side call, timeout, retry with backoff (idempotent only), cache, fallback |
| Counters (likes/views) | Atomic increment, unique constraint per user |
| Search | DB full-text or indexed `ILIKE` with limit; debounce in UI |
| Stats / data exploitation | Aggregation queries + cache; charts with clear labels |
| Real-time | Polling first; WebSocket/SSE only if the feature requires it |

## Scalability story (visible to a technical jury)

Keep the app server **stateless** (sessions in DB/Redis, uploads in storage), use indexes, pagination, caching, and compression. Mention these choices in `FEATURES.md` / README with a small architecture diagram: it shows intent even if traffic is small.
