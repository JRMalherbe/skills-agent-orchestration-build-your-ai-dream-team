# Project Pulse — Final Handoff

## Overview

Project Pulse is a polished single-page static dashboard that gives Mona's contributors an at-a-glance view of active projects — showing each project's name, owner, status, recent activity, and priority as responsive, accessible cards.

## Team

This deliverable was produced by a four-agent custom team defined under `.github/agents/` and coordinated through GitHub Copilot CLI in a Codespace:

- **Orchestrator** — Broke the request into phases, assigned non-overlapping file scopes, ran independent Designer and Coder work in parallel, and verified integration.
- **Planner** — Produced `docs/project-pulse-plan.md`: ordered steps, file assignments, the shared JSON schema + CSS hook contract, edge cases, and validation expectations.
- **Designer** — Owned the visual system: layout tokens, responsive grid, card styling, status badges, priority treatment, accessibility (WCAG AA, `:focus-visible`, `prefers-reduced-motion`).
- **Coder** — Implemented the markup, data, rendering logic, and launch configuration within the scope assigned by the Orchestrator.

## Deliverables

| File | Owner | Purpose |
|---|---|---|
| `app/index.html` | Coder | Semantic dashboard shell titled "Project Pulse". Links `styles.css`, fetches `project-data.json`, renders `.project-card` articles at runtime with XSS-safe `textContent`. Handles empty and error states. |
| `app/styles.css` | Designer | Design system for the dashboard — tokens, `.dashboard` shell, responsive `.project-grid`, `.project-card` with border-radius and box-shadow, status badges, priority treatment (color + shape + label), edge states. |
| `app/project-data.json` | Coder | Strict JSON with a top-level `projects` array. Each entry provides `name`, `owner`, `status`, `recentActivity`, and `priority`. Five sample projects exercise every allowed status and multiple priorities. |
| `.vscode/launch.json` | Coder | Strict JSON launch configuration named "Run Project Pulse Dashboard" that serves the `app/` directory via `python3 -m http.server 5500` and opens `http://localhost:5500/index.html` through `serverReadyAction` — landing the learner on the rendered dashboard, not a directory listing. |

## Shared contract

The Planner froze a shared contract before implementation so Designer and Coder could work in parallel without drift:

- **JSON schema:** top-level `projects` array; each item has `name`, `owner`, `status`, `recentActivity`, `priority`.
- **Allowed values:** status ∈ {`active`, `at-risk`, `blocked`, `shipped`, `on-hold`}; priority ∈ {`high`, `medium`, `low`}; unknown values fall back to an `--unknown` modifier.
- **DOM/CSS hooks:** `.dashboard`, `.dashboard__header`, `.project-grid`, `.project-card`, `.project-card__title`, `.project-card__owner`, `.project-card__activity`, `.status-badge` + state modifiers, `.priority` + level modifiers, `.empty-state`, `.error-state`.

## Validation

Verified against the plan's validation checklist:

1. **Title & references** — `app/index.html` uses the exact `<title>Project Pulse</title>` and references both `styles.css` and `project-data.json`.
2. **Rendered cards** — Cards use the `project-card` class and display each project's status (as a badge), recent activity, and priority (with a visible "Priority: …" label so meaning is not conveyed by color alone).
3. **Design system** — `app/styles.css` includes explicit `.dashboard` and `.project-card` selectors, uses `border-radius`, `box-shadow`, and an `auto-fit` responsive grid that collapses to a single column at ≤640px.
4. **Accessibility** — WCAG AA contrast on badges and text, visible `:focus-visible` outline, `prefers-reduced-motion` honored, priority conveyed via color + shape + text label.
5. **JSON strictness** — `app/project-data.json` and `.vscode/launch.json` both parse cleanly with `python3 -c "import json; json.load(...)"` and contain no comments or trailing commas.
6. **Data coverage** — Five projects cover every allowed status (`active`, `at-risk`, `blocked`, `shipped`, `on-hold`) and multiple priorities.
7. **Launch behavior** — The "Run Project Pulse Dashboard" configuration in `.vscode/launch.json` runs `python3 -m http.server 5500` with `cwd = ${workspaceFolder}/app` and uses `serverReadyAction` to open `http://localhost:%s/index.html` — the dashboard frontend, not a directory listing.
8. **Edge paths** — Empty `projects` reveals `.empty-state`; fetch/parse failure reveals `.error-state` and logs to the console.
9. **XSS safety** — All project data is injected via `textContent`; no `innerHTML` with raw JSON.
10. **Contract alignment** — Coder's status slugs (`active`, `at-risk`, `blocked`, `shipped`, `on-hold`) and priority slugs (`high`, `medium`, `low`) match Designer's modifier classes exactly.

## Handoff

The Project Pulse dashboard is complete and ready for Mona's team. To preview it:

1. Open the repository in VS Code (or a Codespace).
2. Open the **Run and Debug** panel.
3. Select **Run Project Pulse Dashboard** and start it.
4. The configured `serverReadyAction` opens the dashboard at `http://localhost:5500/index.html` automatically.

To evolve the dashboard, add or edit entries in `app/project-data.json` (respecting the shared contract above). Visual tweaks belong in `app/styles.css`; DOM structure and rendering logic live in `app/index.html`. The launch configuration in `.vscode/launch.json` needs no further changes for day-to-day use.
