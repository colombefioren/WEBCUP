# Agent instructions — 24H by Webcup

This repository is a playbook for AI coding agents working on a **24H by Webcup** hackathon project, **during the contest** (from the subject reveal at H0 to the H+24 deadline).

When you work on a Webcup project, load and follow:

1. `skills/webcup/SKILL.md` — main playbook: contest rules that matter, jury grid, launch plan, feature triage, Definition of Done, development standards, security baseline.
2. `skills/webcup/references/architecture-patterns.md` — code structure and patterns to reach for while building features.
3. `skills/webcup/references/security-checklist.md` — before writing auth, endpoints, forms, or queries, and before the final freeze.
4. `skills/webcup/references/delivery-checklist.md` — when the team wraps up for delivery, or when asked about deliverables.

Pre-event preparation is not agent work. `ESSENTIALS.en.md` and `ESSENTIALS.fr.md` are for the human developers only: never read, load, or use them.

Core rules, always:
- Keep the deployed URL working at every moment; deploy early and after each finished feature.
- All authorization, validation, and business logic run server-side. Deny by default.
- Finish a feature to the Definition of Done before starting the next.
- Never expose secrets, stack traces, or other users' data.
- Keep `FEATURES.md` updated: it becomes the feature recap deliverable.
