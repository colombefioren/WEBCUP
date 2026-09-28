# WEBCUP

Agent skill + team notes for the **24H by Webcup** hackathon (24-hour web application sprint).

- `skills/webcup/` — the skill (English), used **during the contest** (H0 → H+24). Tells the coding agent how to maximize the jury score: feature triage, secure-by-default architecture, Definition of Done, security checklist, delivery checklist.
- `ESSENTIALS.en.md` / `ESSENTIALS.fr.md` — **for the human developers only**, not for coding agents: dates, deliverables, evaluation grid, do's and don'ts, the J-7 preparation checklist, and what to check before the event. None of the install methods below copy them into your project.
- `AGENTS.md` — entry point for agents that read `AGENTS.md`.

## Install

### Claude Code

```bash
claude plugin marketplace add colombefioren/WEBCUP
claude plugin install webcup@webcup -s user
```

Or copy the skill folder:

```bash
git clone https://github.com/colombefioren/WEBCUP.git
cp -r WEBCUP/skills/webcup ~/.claude/skills/webcup
```

### Any `skills` CLI-compatible agent (Cursor, Codex, and others)

```bash
npx skills add colombefioren/WEBCUP
```

### Freebuff / Codebuff and other agents

Copy only the agent files into your project (not the whole repo, so the human-only `ESSENTIALS` files stay out of the agent's workspace), then point the agent at `.webcup/AGENTS.md`:

```bash
git clone --depth 1 https://github.com/colombefioren/WEBCUP.git /tmp/webcup
mkdir -p .webcup && cp -r /tmp/webcup/skills /tmp/webcup/AGENTS.md .webcup/
rm -rf /tmp/webcup
```

Or paste `skills/webcup/SKILL.md` and `skills/webcup/references/` into the agent's knowledge/instructions location (e.g. `knowledge.md`, `.cursor/rules/`).

## Source

Based on the official rules page: *24h by Webcup – The ultimate web development sprint*. Always re-check the official page before the event.
