# 24H by Webcup — Essentials for developers

Condensed from the official rules ("24h by Webcup – The ultimate web development sprint"). Re-read the official page before the event: numbers and details can change.

## The format in one paragraph

In 24 hours, each team builds a **functional web application** (a real product/service, not a showcase site). The subject is revealed at launch with a set of **mandatory base features**. During the 24 hours, **new feature requests drop at regular intervals** — too many to finish on purpose. For each one, the team chooses: do it now, later, or skip it. At the end, the team delivers an online app. The jury evaluates it **in the following days, without an oral presentation**, by testing it directly.

## Key dates to remember

| When | What happens | What you must do |
|---|---|---|
| **J-7** | Servers / hosting open (technical partner HODi) | Install stack, configure server, test deployment. The subject is still secret: prepare a generic foundation. |
| **J0 (H0)** | Official launch: concept, theme, base features, rules | Read everything, plan the data model, deploy the skeleton within 2 hours. |
| **H0 → H24** | New features announced at regular intervals | Triage each one (now / later / skip), keep the online app working. |
| **H+24** | Development ends — hard deadline | Everything delivered and online. Nothing after this counts. |
| **J+1 → J+X** | Jury evaluates asynchronously | Nothing to do: your URL, accounts, recap, and video speak for you. Keep the app online. |
| **J+X** | Results and prize ceremony | Podium + special mentions. |

## Deliverables (minimum, at closing)

1. **Application URL** — usable version, online.
2. **Jury access** — accounts that unlock every part of the app, including authenticated features (one per role).
3. **Recap of implemented features** — helps the jury find what you built. It does not replace testing.
4. **Short demo video** — overview of the app, its logic, and main features.

Only what is actually delivered by the deadline is evaluated.

## How you are evaluated

| Dimension | What the jury looks at |
|---|---|
| **Implemented features** | Present, working, really integrated in the app. Relevance and finish count more than quantity. |
| **Technical quality** | Coherent structure, development logic, overall robustness, integration quality, good use of data/APIs, application security. |
| **Design & UX** | Clear, coherent, usable interface: readability, ergonomics, navigation, graphic quality, user experience. |
| **Overall coherence** | Balance between ambition and execution, functional logic, overall quality, choices that serve the app. |

A shared grid is used for all teams. The jury may include technical, functional, design/UX, and **cybersecurity** profiles.

## Prizes

- General ranking: **1st, 2nd, 3rd prize** (most complete, coherent, finished projects).
- Possible special distinctions (mentions, certificates, badges): graphic quality, UX, technical quality, a particularly successful feature, originality or overall coherence. You can win one without reaching the podium.

## Cybersecurity & robustness

Not a hacking contest, but apps are tested against simple malicious behavior and common design mistakes. **The vulnerability families will be announced in advance.** Expect:

- authentication
- access control
- user input validation
- endpoint protection
- unintended data exposure
- simple brute force and role mismanagement

Some features announced during the 24h may themselves be security features.

## Allowed technologies & AI

- Any language, framework, library, versioning/deployment/automation tool, and external service, **as long as it is compatible with the provided environment**. The organization may announce technical constraints beforehand.
- **AI tools are allowed.** But using AI alone will not win: choices, structure, integration, and coherence decide.

### AI inside your app: OpenRouter (free models)

| Limit | Value | Condition |
|---|---|---|
| Requests / minute | 20 | Always |
| Requests / day | 50 | New account |
| Requests / day | 1000 | After a first one-time credit on the account |

- Free models end with `:free`. Tokens cost 0; the real limit is the **number of requests**.
- Context windows of free models are large (256K to ~1M tokens) — not a real constraint for normal features.
- The free catalog changes without notice: **keep a backup model**.
- Multiple accounts do **not** increase the quota.
- Every call counts, including tests. On an error, wait a few seconds and retry.
- **Never put the key in public code** (GitHub, screenshots). Call it from the server only.
- Check the latest limits on openrouter.ai/docs before the event.

## DO

- Use J-7 fully: stack installed, HTTPS, one-command deploy, auth + roles, DB + migrations, base UI components, tested on the real server.
- Deploy in the first 2 hours, then after every finished feature. The online app must work at all times.
- Finish one feature completely before starting another.
- Prioritize: value for the jury vs time vs risk. Security features are often cheap and valued.
- Keep a `FEATURES.md` updated all along (status + reason for skipped features) — it becomes the recap.
- Do all security checks server-side (auth, roles, ownership, validation).
- Prepare realistic demo data and jury accounts for each role.
- Freeze features around H+20; then polish, secure, test as the jury, record the video.
- Split roles in the team: back-end/security, front-end/UX, integration/deploy, product/triage/deliverables.

## DON'T

- Don't show features that don't really work (mocks, dead buttons, "coming soon").
- Don't chase quantity: a feature pile without integration is penalized.
- Don't leave the online app broken, even for "5 minutes".
- Don't protect admin pages only by hiding them in the interface.
- Don't expose secrets, stack traces, `.env`, passwords, or other users' data.
- Don't waste the OpenRouter quota on tests.
- Don't deploy risky changes in the last hour.
- Don't forget any deliverable: jury accounts, recap, video.

## J-7 preparation (servers open, subject still secret)

Goal: arrive at H0 with a deployed, secure, subject-agnostic skeleton so the 24 hours go to features, not setup. The subject is unknown, so everything built here must be generic. The coding-agent skill starts at H0: it will reuse whatever you prepare here.

### Choosing the stack

- Pick what the team already masters. Team speed beats the theoretical best stack.
- It must run on the provided server (HODi). Check runtime versions, DB availability, reverse proxy, HTTPS, ports, and disk/RAM limits on day J-7.
- Prefer one full-stack framework with server-side rendering or a clear API layer (e.g. Next.js/Nuxt/SvelteKit + ORM, Laravel, Django, Rails, NestJS + SPA). Fewer moving parts = fewer failures at 4 a.m.
- Relational DB (PostgreSQL or MySQL) by default: relations, constraints, and transactions help both coherence and security.

### Build before H0

**Deploy**
- [ ] Repo, `.gitignore` with `.env`.
- [ ] One-command deploy to the provided server (script or CI). Tested twice.
- [ ] HTTPS, domain, HTTP→HTTPS redirect.
- [ ] Production env vars set on the server; debug off.
- [ ] Process manager / container restart on crash.

**Back-end**
- [ ] DB connection, migrations, seed script (jury accounts per role + demo data).
- [ ] Auth: register, login, logout, me; argon2id or bcrypt (cost ≥ 12); secure cookie session; session rotation.
- [ ] Roles: `user`, `admin` (+ room for a third role); role guard middleware.
- [ ] Ownership helper reused by every module.
- [ ] Schema validation middleware.
- [ ] Central error handler + JSON error contract `{ error: { code, message, fields? } }`.
- [ ] Rate limiter (login, register, reset, AI, generic API).
- [ ] Security headers, CORS allowlist, CSRF protection.
- [ ] Structured logger with request id; `/health` returns DB status.
- [ ] Pagination helper and cache helper (in-memory or Redis).
- [ ] AI proxy route: server-side OpenRouter call, timeout, fallback model, cache, 429 handling, dev mock. Delete it if unused.

**Front-end**
- [ ] Layout, navigation, auth pages, protected routes.
- [ ] Design tokens (colors, type scale, spacing).
- [ ] Components: button, input with error, select, list/table with pagination, card, modal, toast, skeleton loader, empty state, error boundary.
- [ ] 404 and 500 pages.
- [ ] Responsive check at 390px.

**Quality**
- [ ] Lint + typecheck + minimal tests in one command.
- [ ] Dependency audit clean.
- [ ] Smoke test: "hello authenticated world" deployed end to end on the real server.

## To check / prepare before the event

- [ ] Official page: rules, dates, local organizer info, any technical constraints.
- [ ] Announced list of vulnerability families (published before the contest).
- [ ] Server specs (runtime versions, DB, RAM, disk, domain, HTTPS, ports).
- [ ] OpenRouter account created, key stored safely, current limits and free models checked, backup model chosen.
- [ ] Deployment tested twice from scratch.
- [ ] Team roles decided, communication channel ready, snacks and sleep plan.
- [ ] Screen recording tool ready for the demo video.
