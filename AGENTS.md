# Agent instructions — 24H by Webcup

This repository is a playbook for AI coding agents working on a **24H by Webcup** hackathon project.

When you work on a Webcup project, load and follow:

1. `skills/webcup/SKILL.md` — main playbook: contest rules that matter, jury grid, phase plan, feature triage, Definition of Done, development standards, security baseline.
2. `skills/webcup/references/architecture-baseline.md` — during preparation (J-7) and at launch (H0).
3. `skills/webcup/references/security-checklist.md` — before writing auth, endpoints, forms, or queries, and before the final freeze.
4. `skills/webcup/references/delivery-checklist.md` — from H+20, or when asked about deliverables.

Human-oriented summaries of the rules: `ESSENTIALS.en.md`, `ESSENTIALS.fr.md`.

Core rules, always:
- Keep the deployed URL working at every moment; deploy early and after each finished feature.
- All authorization, validation, and business logic run server-side. Deny by default.
- Finish a feature to the Definition of Done before starting the next.
- Never expose secrets, stack traces, or other users' data. AI keys stay on the server.
- Keep `FEATURES.md` updated: it becomes the feature recap deliverable.
