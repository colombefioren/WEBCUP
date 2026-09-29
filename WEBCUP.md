# WEBCUP — Skeleton, stack & launch kit (full Next.js)

Team reference for the J-7 week and for H0. It covers what to build before the subject is revealed, how to deploy it on HODi, which features the subject is likely to ask for, and a ready-to-paste prompt for any LLM once the subject is out.

Companion files: `ESSENTIALS.fr.md` / `ESSENTIALS.en.md` (rules, dates, deliverables), `skills/webcup/` (coding-agent playbook during the contest).

> Everything about HODi below comes from HODi's public help pages. **Verify it on the real Webcup account at J-7**: the Webcup access mode may differ from the standard offers.

---

## 1. Principles behind the skeleton

The jury tests a **live URL** days later, with the accounts you give them, without any pitch. It scores implemented features, technical quality (structure, robustness, data/API use, security), design/UX, and overall coherence.

So the skeleton must give you, before H0:

1. **A deployed, secure app on HODi** — auth, roles, admin, and a working deploy pipeline — so the 24 hours go to the subject.
2. **Generic building blocks** that most subjects need (CRUD module, dashboards, uploads, notifications, search, stats, AI assistant), easy to rename to the subject's entities.
3. **Security and robustness by default**, since cybersecurity is on the grid and the vulnerability families are announced in advance.
4. **Nothing half-done visible.** Every prepared block sits behind a feature flag and is only shown once it serves the subject.

---

## 2. HODi — what to check on day J-7

What HODi's help documentation says (standard offers):

| Topic | HODi (per public docs) | What it means for us |
|---|---|---|
| Panel | cPanel | Env vars, DBs, cron, SSL, mail are set there |
| Node.js | "Setup Node.js App" (Passenger), Passenger runtime on higher tiers, SSH | Next.js runs as a Passenger Node app |
| Git deploy | **Hodifly**: GitHub/GitLab, auto deploy, **PR preview deployments**, rollback | Main deploy path; previews let us test before merging |
| Databases | MySQL and **PostgreSQL** | Use PostgreSQL |
| SSH | Yes, port **22974** (not 22) | For logs, migrations, debugging |
| Docker | Not mentioned | Don't plan on Docker |
| Webcup | "Specific access mode" for 24h by Webcup participants | Details come with the account — check them first |

**J-7 verification checklist (do this first, before choosing anything else):**

- [ ] Log into the Webcup cPanel. Note the plan limits: RAM, CPU, processes (LVE limits), disk, inodes.
- [ ] "Setup Node.js App": which **Node versions** are available? (Next.js needs a recent LTS.)
- [ ] Can `next build` run on the server without being killed (RAM)? If not → build elsewhere (Hodifly build step or GitHub Actions) and deploy the built output.
- [ ] Hodifly: connect the GitHub repo, confirm Next.js auto-detection, test a deploy, test a **PR preview**, test a **rollback**.
- [ ] Create the PostgreSQL DB + user in cPanel. Check the Postgres version and that it's reachable from the app (and from your machine after allow-listing your IP, for migrations).
- [ ] SSH on port 22974 works; find where app logs go.
- [ ] Domain / subdomain for the app, HTTPS certificate (AutoSSL / Let's Encrypt), HTTP→HTTPS redirect.
- [ ] Cron jobs available in cPanel (needed for scheduled tasks).
- [ ] Email: create an SMTP mailbox in cPanel (password reset, notifications). Test sending.
- [ ] Are WebSockets supported through Passenger? If unsure, plan on SSE/polling.
- [ ] Is outbound HTTPS allowed (AI provider, external APIs)?
- [ ] How many Passenger processes run? (In-memory cache/rate-limit won't be shared across processes → use the DB.)
- [ ] Ask the organizers: technical constraints, the announced vulnerability families, deliverable format, and whether the jury gets our repo or only the URL.

---

## 3. Recommended stack (full Next.js)

Pick versions that are current at J-7 and **pin them**. Don't upgrade during the contest.

| Layer | Choice | Why |
|---|---|---|
| Framework | **Next.js (App Router) + TypeScript strict** | One codebase for UI, API, SSR |
| Server logic | Server Actions for forms/mutations, Route Handlers (`app/api/*`) for public/JSON endpoints, cron, webhooks | Clear boundaries, built-in origin checks on Server Actions |
| Database | **PostgreSQL** (HODi) | Relations, constraints, transactions, full-text search |
| ORM | **Drizzle** + `pg` (pure JS, light on shared hosting). Prisma is fine **only if** it runs on HODi at J-7 | Fewer native-binary surprises on cPanel |
| Auth | **Better Auth** (email/password, DB sessions, rate limiting, admin plugin for roles/ban) — or Auth.js if the team knows it | Roles, bans and sessions without writing it all |
| Validation | **Zod** — one schema shared by form and server | Same rules client + server |
| UI | Tailwind CSS + **shadcn/ui**, lucide icons, sonner toasts, react-hook-form, TanStack Table (admin tables), Recharts (charts) | Fast, accessible, consistent |
| i18n | UI in **French** by default (jury is francophone); `next-intl` only if EN is needed | Clarity for the jury |
| AI | **Vercel AI SDK** + OpenRouter provider, server-only, streaming, tool calling | Assistant/agent with the app's own data |
| Email | Nodemailer via cPanel SMTP | Reset password, notifications |
| Files | Local disk **outside** `public/`, served by an authenticated route (or S3-compatible if available) | Access control on files |
| Jobs | cPanel cron → `POST /api/cron/<job>` with a secret header; `jobs` table | No Redis/queue needed |
| Rate limit & cache | **DB-backed** (Postgres tables) + Next.js caching (`unstable_cache` / `use cache`, `revalidateTag`) | Works across Passenger processes |
| Logs | pino (JSON) with request id | Debug on the server over SSH |
| Tests | Vitest (services, authz) + Playwright smoke tests (login, main flow per role) | Catch regressions before deploy |
| CI | GitHub Actions: lint + typecheck + build + tests on every PR | Main stays deployable |
| Tooling | pnpm, ESLint, Prettier, Husky pre-commit (lint-staged) | Consistent code |

Build: `output: "standalone"` in `next.config` so the deploy is a self-contained Node server Passenger can start.

---

## 4. Project structure (separation of concerns)

One responsibility per file; no file near 1000 lines (aim ~300). Business rules live in services, never in pages or components.

```
src/
  app/
    (public)/            landing, about, legal pages, login/register
    (app)/               authenticated area: dashboard, resources, profile, notifications
    (admin)/admin/       admin area: users, roles, audit log, stats, settings, feature flags
    api/
      health/route.ts
      cron/[job]/route.ts
      ai/chat/route.ts
      files/[id]/route.ts
    error.tsx  not-found.tsx  loading.tsx  layout.tsx
  modules/<entity>/      one folder per domain entity
    <entity>.schema.ts       Zod input schemas
    <entity>.service.ts      business rules + authorization calls
    <entity>.repository.ts   Drizzle queries only
    <entity>.actions.ts      Server Actions (thin: validate → service → revalidate)
    <entity>.dto.ts          response shapes (what may leave the server)
    components/              UI for this entity
  lib/
    auth/        session helpers, requireUser(), requireRole()
    authz/       permission matrix + can(user, action, resource)
    db/          client, schema, migrations
    security/    rate limit, headers, CSP nonce, sanitize
    ai/          provider client, tools, fallback, prompt templates
    mail/  files/  jobs/  audit/  flags/  logger/  cache/
  components/ui/  shared design-system components
scripts/
  seed.ts         jury accounts per role + realistic demo data
FEATURES.md       live feature status → becomes the jury recap
```

---

## 5. The skeleton — what to build before H0

Priority order: build top to bottom, deploy after each block.

### 5.1 Foundation (must have)

- [ ] Next.js app deployed on HODi via Hodifly, HTTPS, env vars in cPanel, `/api/health` returns DB status.
- [ ] PostgreSQL + Drizzle migrations + `seed.ts`.
- [ ] Security headers (CSP with nonce, HSTS, nosniff, frame-ancestors, referrer-policy), `X-Powered-By` off.
- [ ] Central error handling: `error.tsx`, `not-found.tsx`, `global-error.tsx`, JSON error contract for API routes `{ error: { code, message, fields? } }`.
- [ ] Structured logs with request id.
- [ ] CI on GitHub (lint, typecheck, build, tests).

### 5.2 Accounts & auth

- [ ] Register, login, logout, session in DB (httpOnly, secure, sameSite).
- [ ] Forgot / reset password by email (single-use expiring token), change password.
- [ ] Email verification (feature-flagged — enable if the subject needs it).
- [ ] Profile page: name, avatar upload, preferences.
- [ ] Delete my account + export my data (RGPD — cheap, and shows maturity).
- [ ] Login rate limiting + lockout/backoff, generic error messages (no account enumeration).
- [ ] Optional, flagged: 2FA (TOTP) — a classic "security" feature drop.

### 5.3 Roles & permissions

- [ ] Roles: `admin`, `user`, plus one spare role (`staff` / `moderator` / `author` / `manager`) to rename to the subject.
- [ ] Central permission matrix `can(user, action, resource)` used by every service. UI hides what the server already forbids — never the other way round.
- [ ] Ownership helper `assertOwner(resource, user)` for every read/update/delete (IDOR protection).
- [ ] Role changes only by admin, logged in the audit log. Users can never set their own role (schema strips `role`).

### 5.4 Dashboards

- [ ] **User dashboard**: KPI cards, recent activity, shortcuts, empty states that explain what to do first.
- [ ] **Admin dashboard**: user count/growth, activity over time, top items, recent sign-ups, errors/health — charts from real aggregation queries.
- [ ] Spare-role dashboard (e.g. author/manager): "my content" stats.

### 5.5 Admin back-office

- [ ] Users table: search, filters (role, status), sort, pagination, change role, ban/unban.
- [ ] **Audit log** viewer: who did what, when, on which resource (logins, role changes, deletions, admin actions).
- [ ] **Feature flags** page: turn prepared blocks on/off without redeploying (lets you hide unfinished work and react to drops fast).
- [ ] Content moderation queue (flagged items) — generic, rename to subject.
- [ ] Site settings (name, contact, maintenance mode).

### 5.6 Generic resource module (the most valuable block)

One complete example entity (`Item`) to duplicate and rename in minutes when the subject arrives:

- [ ] List with search, filters, sort, pagination (cursor or offset + max limit).
- [ ] Detail page, create/edit forms (react-hook-form + shared Zod schema), delete with confirm, soft delete.
- [ ] Status workflow (`draft → published → archived`) with role-based transitions.
- [ ] Owner field + authz on every action; author/owner shown on the item.
- [ ] Image/file attachments.
- [ ] Comments, likes/ratings, favorites/bookmarks (each behind a flag).
- [ ] CSV export (and import if cheap).
- [ ] Optimistic UI + toasts + loading skeletons.

### 5.7 Cross-cutting blocks

- [ ] **Uploads**: MIME + extension allowlist, size limit, random names, stored outside `public/`, served through an authz route, image resize.
- [ ] **Notifications**: in-app (bell + unread count + list) and email for key events; user preferences.
- [ ] **Search**: Postgres full-text across main entities, debounced input.
- [ ] **Stats / data exploitation**: aggregation queries + cache + Recharts; this directly answers "bonne exploitation des données".
- [ ] **Scheduled jobs**: cron route + `jobs` table (reminders, digests, cleanup).
- [ ] **Public API** (flagged): read-only JSON endpoints with API keys + rate limit — a common "technical integration" drop.
- [ ] **Webhooks / external API client** template with timeout, retry (idempotent only), cache, fallback.
- [ ] **PDF generation** (invoice/ticket/report) — flagged, only if cheap in the stack.
- [ ] **Calendar/booking** primitives (date picker, availability, conflict check) — flagged.
- [ ] **Map** (Leaflet + OpenStreetMap) — flagged, for location-based subjects.
- [ ] **Real-time**: SSE or polling helper (live counters, notifications).
- [ ] Dark mode, accessible components (labels, focus, contrast, keyboard).

### 5.8 AI assistant / agent (strong differentiator if done right)

- [ ] Chat panel (streaming) available to logged-in users.
- [ ] **Tool calling scoped to the user's permissions**: the agent calls the same services as the UI (search items, summarize, create draft, get my stats) — never raw DB access, never beyond the user's role.
- [ ] System prompt server-side; user input delimited; model output rendered as text (escaped), never executed, never used to authorize actions.
- [ ] Rate limit per user, timeout, fallback model list, response cache, friendly message when the provider is down or the quota is hit. The core app must work without AI.
- [ ] Other AI blocks to reuse on the subject: summarize, classify/tag, generate a description from fields, semantic-ish search, moderation of user content.
- [ ] Keep the AI key only on the server; mock the AI client in development to save quota (see ESSENTIALS for OpenRouter limits).

### 5.9 Public pages & polish

- [ ] Landing page that explains the product in one screen (jury has no pitch).
- [ ] Legal: mentions légales, privacy (RGPD), cookies.
- [ ] SEO: metadata, Open Graph image, `sitemap.xml`, `robots.txt`.
- [ ] 404 / 500 pages in the app's style.
- [ ] Onboarding hints / first-run guide.

### 5.10 Seed & jury access

- [ ] `seed.ts` creates one jury account per role with strong unique passwords + realistic, theme-neutral demo data (easy to re-theme at H0).
- [ ] A reset script to restore demo data if a juror breaks it.

### 5.11 Tests you want green before H0

- [ ] Auth flow (register, login, reset).
- [ ] Authz: user A cannot read/edit user B's item (IDOR test); user cannot reach admin routes.
- [ ] Validation rejects bad input; rate limit returns 429.
- [ ] Playwright smoke: login per role → main dashboard renders.

---

## 6. Plausible feature drops → what we reuse

| Likely drop | Ready block |
|---|---|
| New role / permissions change | Roles + permission matrix |
| Admin must manage X | Admin back-office + resource module |
| Dashboard / statistics | Dashboards + stats block |
| Search / filters | Search + resource list |
| Export (CSV/PDF) | Export + PDF blocks |
| Notifications / reminders | Notifications + cron jobs |
| Upload images/documents | Uploads block |
| Comments / ratings / favorites | Resource module options |
| Chatbot / AI feature | AI assistant block |
| Public API / integration | Public API + external client template |
| Booking / calendar | Calendar primitives |
| Map / geolocation | Map block |
| Real-time updates | SSE/polling helper |
| **2FA / stronger passwords / lockout** | Auth block (security drops) |
| **Audit trail / logs** | Audit log |
| **Rate limiting / brute-force protection** | Rate-limit lib |
| **RGPD: export/delete my data** | Account block |
| Moderation / reporting | Moderation queue |
| Multi-language | next-intl (only if asked) |
| Accessibility / dark mode | UI kit |
| Payment | Stripe **test mode** only, flagged — skip unless central to the subject |

---

## 7. Deploying Next.js on HODi (to validate at J-7)

1. `next.config`: `output: "standalone"`.
2. cPanel → **Setup Node.js App**: pick the Node version, application root, startup file (the standalone `server.js`), set env vars (`DATABASE_URL`, auth secret, SMTP, AI key, `NODE_ENV=production`).
3. **Hodifly**: connect GitHub, deploy `main` automatically, use PR previews to test risky changes, keep rollback ready.
4. If the server can't build (RAM limits): build in GitHub Actions and deploy the built standalone output instead.
5. Run migrations over SSH (port 22974) or in the deploy step — never by hand at 4 a.m. without a backup.
6. Copy `public/` and `.next/static` next to the standalone server (standalone doesn't include them by default).
7. cPanel cron → `curl -X POST -H "x-cron-secret: …" https://<app>/api/cron/<job>`.
8. Take a DB backup before every migration and at the deadline.
9. Smoke test after every deploy: `/api/health`, login, one main flow.

---

## 8. Team split (4 people, adapt)

- **Back-end & security**: DB, auth, authz, services, rate limit, audit, security checklist.
- **Front-end & UX**: design system, pages, dashboards, states, responsive, accessibility.
- **Integration & DevOps**: HODi, Hodifly, CI, env, cron, mail, AI block, monitoring.
- **Product & delivery**: reads drops, triage (now/later/skip), `FEATURES.md`, seed data, jury accounts, demo video.

---

## 9. H0 procedure (once the subject is revealed)

1. Read the whole subject. List entities, roles, base features, security needs.
2. Rename the spare role and the `Item` module to the subject's entities; add entity modules by duplication.
3. Update schema + migrations + seed with themed demo data.
4. Turn on only the flags the subject needs. Hide everything else.
5. Fill `FEATURES.md` with base features + acceptance criteria.
6. Deploy the themed skeleton before building new features.
7. Paste the prompt below into your LLM together with the subject.

---

## 10. LLM prompt for H0 and every feature drop

Copy, fill the `{…}` parts, paste into any LLM or coding agent.

````text
You are the senior full-stack engineer of our team at "24H by Webcup", a 24-hour web application hackathon in the Indian Ocean region.

## How the contest works
- The subject (an app concept, a theme, and mandatory base features) is revealed at H0. During the 24 hours, new feature requests are announced at regular intervals — deliberately more than anyone can finish. For each one we decide: do it now, later, or skip it.
- At the deadline we deliver: the live app URL, jury accounts for every role, a recap of implemented features, and a short demo video. Only what is delivered and online counts.
- The jury evaluates days later, asynchronously, by testing the live URL. No pitch. The jury may include technical, functional, design/UX, and cybersecurity profiles.
- Scoring: (1) implemented features — present, working, integrated, finished; relevance beats quantity; (2) technical quality — coherent structure, development logic, robustness, integration quality, good use of data/APIs, application security; (3) design & UX — clear, coherent, usable; (4) overall coherence — ambition balanced with execution, choices that serve the product. A pile of unintegrated features is penalized.
- Security testing covers: authentication, access control, input validation, endpoint protection, unintended data exposure, simple brute force, role mismanagement.

## Our stack and skeleton (already built and deployed)
- Next.js App Router + TypeScript strict, Server Actions + Route Handlers, PostgreSQL + Drizzle, Better Auth (DB sessions, roles, rate limiting), Zod, Tailwind + shadcn/ui, Recharts, Vercel AI SDK (server-only), Nodemailer, pino. Deployed on HODi (cPanel, Passenger Node app, Hodifly git deploy).
- Structure: src/modules/<entity>/{schema, service, repository, actions, dto, components}. Business rules in services only. Authorization via can(user, action, resource) and assertOwner(). One responsibility per file; no file over ~300 lines (hard limit 1000).
- Ready blocks: auth (reset, verify, 2FA flag), roles {admin, user, {SPARE_ROLE}}, admin back-office (users, roles, bans, audit log, feature flags, moderation), user/admin dashboards, generic resource module (CRUD, search, filters, pagination, status workflow, attachments, comments/ratings/favorites, CSV export), uploads, notifications (in-app + email), full-text search, stats, cron jobs, public API with keys, AI assistant with permission-scoped tools, SSE helper, legal pages, seed with jury accounts.

## The subject
{PASTE THE FULL SUBJECT HERE}

## What I need now
{ONE OF: "Plan the H0 adaptation" | "Implement feature: …" | "Triage this new feature drop: …" | "Review readiness for delivery"}

## Rules for your answer
1. Reuse the existing blocks and conventions; do not introduce a new library unless necessary, and say why.
2. For a new feature drop, first give a triage: value for the jury, cost, risk to working features, fit with the app's story → recommend now / later / skip with one reason.
3. For implementation: list the files to create/change (respecting the module structure), then the code. Include the Zod schema, the service with authorization (role + ownership), the Server Action or Route Handler, the UI with loading/empty/error/success states, and the DB migration if needed.
4. Security by default: authenticated unless explicitly public, validated input, explicit output DTO (never return password hashes, tokens, or other users' private data), rate limit on sensitive actions, no secrets client-side, user content escaped, AI output treated as untrusted.
5. Keep the app deployable: no breaking changes to shared code without saying so; migrations must be safe on existing data.
6. End with: how to test it on the live app (steps per role), and the line to add to FEATURES.md.
7. Answer in {French/English}. The app UI is in French.
````

---

## 11. Sources

- HODi help — web hosting category (cPanel, Node.js, Hodifly, databases, SSH): https://help.hodi.host/fr/category/hebergement-web-1tfo26c/
- Webcup × HODi partnership: https://www.webcup.fr/actualites/a-la-webcup-on-heberge-plus-green-avec-hodi-host-29893.html
- Official rules: "24h by Webcup – The ultimate web development sprint" (Notion page shared by the organizers).
