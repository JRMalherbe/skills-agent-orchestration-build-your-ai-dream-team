# Project Pulse — Implementation Plan

## 1. Summary

Project Pulse is a small static dashboard that helps Mona's contributors see, at a glance, which projects are active, who owns them, their current status, recent activity, and priority/risk. The deliverable is a polished single-page dashboard rendered from `app/index.html`, styled by `app/styles.css`, hydrated from `app/project-data.json`, and launchable from VS Code via a "Run Project Pulse Dashboard" configuration in `.vscode/launch.json`. The goal is a first view that clearly reads as a *dashboard* (cards, badges, hierarchy) — not a bare HTML page or a directory listing.

## 2. Ordered Implementation Steps

1. **Contract phase — agree on shared interfaces (no files written).**
   - Freeze the JSON schema for `projects[]` (fields, allowed `status` values, allowed `priority` values).
   - Freeze the deterministic DOM/CSS hook names both Coder and Designer will target.
2. **Author sample data — `app/project-data.json`.** (Coder)
3. **Build dashboard markup and rendering logic — `app/index.html`.** (Coder)
4. **Style the dashboard — `app/styles.css`.** (Designer, in parallel with step 3 once step 1 is locked)
5. **Add VS Code launch configuration — `.vscode/launch.json`.** (Coder, after steps 2–4 exist)
6. **Integration verification.** Orchestrator opens the dashboard via the launch config and walks the Validation checklist (section 9).

## 3. File Assignments

| File | Owner | Contents (scope) |
|---|---|---|
| `app/project-data.json` | **Coder** | Strict JSON. Top-level object with a `projects` array. Each item has `name`, `owner`, `status`, `recentActivity`, `priority`. Include enough entries (≈4–6) to exercise multiple statuses and priorities. |
| `app/index.html` | **Coder** | Semantic HTML skeleton, `<link>` to `styles.css`, script to `fetch('./project-data.json')` and render project cards into the dashboard container using the agreed CSS hook classes. No inline styles beyond what's absolutely unavoidable. |
| `app/styles.css` | **Designer** | All visual design: layout grid, card styling, badges, typography, spacing scale, color tokens, focus states, responsive breakpoints. Targets only the agreed hook classes from step 1. |
| `.vscode/launch.json` | **Coder** | Strict JSON (no comments/trailing commas). Configuration named "Run Project Pulse Dashboard". `cwd` = `${workspaceFolder}/app`. Opens `index.html` so learners see the dashboard, not a directory listing. |

Any additional supporting files (e.g. a small `app/main.js` if Coder chooses to split JS out of `index.html`) are **Coder-owned** and must be declared before step 3 begins so Designer's scope stays purely `styles.css`.

## 4. Designer Responsibilities

- **Layout:** A dashboard shell with a header/title area and a responsive grid of project cards. Grid should reflow from multi-column (wide) to single-column (narrow).
- **Project cards:** Rounded corners, subtle shadow/border, clear internal hierarchy — project name prominent, owner secondary, recent activity de-emphasized, status + priority visually distinct.
- **Status badges:** A dedicated badge treatment (pill shape, background + text color) with distinct visual states for each allowed status value agreed in step 1 (e.g. active / at-risk / blocked / shipped / on-hold). Include a neutral fallback style for unknown values.
- **Priority treatment:** Visually distinguish priority levels (e.g. high / medium / low) via an accent — color swatch, left-border, or dedicated pill. Must be perceivable without relying on color alone (icon, label, or shape as well).
- **Typography:** A single sans-serif stack, clear type scale for h1 / card title / body / meta. Line-height for readability.
- **Spacing:** Consistent spacing scale (e.g. 4/8/16/24). Comfortable padding inside cards and between grid items.
- **Responsive behavior:** Works from ~320px up. No horizontal scroll. Cards remain legible at narrow widths.
- **Accessibility:**
  - WCAG AA contrast for text and badges.
  - Visible keyboard focus styles on any interactive/focusable element.
  - Do not encode meaning in color alone.
  - Respect `prefers-reduced-motion` if any transitions are added.
- **Required CSS hooks (deterministic, must match Coder's DOM):**
  - `.dashboard` — outer dashboard region
  - `.dashboard__header` — title/intro block
  - `.project-grid` — cards container
  - `.project-card` — a single project
  - `.project-card__title`, `.project-card__owner`, `.project-card__activity` — content slots
  - `.status-badge` + status modifier classes (e.g. `.status-badge--active`, `.status-badge--at-risk`, `.status-badge--blocked`, `.status-badge--shipped`, `.status-badge--on-hold`, `.status-badge--unknown`)
  - `.priority` + priority modifier classes (e.g. `.priority--high`, `.priority--medium`, `.priority--low`, `.priority--unknown`)
  - `.empty-state` and `.error-state` for the edge cases below
- **Out of scope for Designer:** HTML structure changes, data shape, launch config.

## 5. Coder Responsibilities

- **HTML structure (`app/index.html`):**
  - `<!doctype html>`, `<html lang="…">`, `<meta charset>`, `<meta name="viewport">`, page `<title>` (e.g. "Project Pulse").
  - Link `styles.css`.
  - Semantic landmarks: `<header>`, `<main>`, and a container using `.dashboard`.
  - A `.project-grid` element that will be populated at runtime.
  - Reserved `.empty-state` and `.error-state` regions (hidden by default) that the render logic can reveal.
- **Data loading:**
  - `fetch('./project-data.json')` on load.
  - Parse and validate: guard for non-object root, missing `projects`, non-array `projects`.
  - On network/parse failure, show `.error-state` with a human-readable message; log to console.
- **Rendering logic:**
  - Iterate `projects` and build DOM nodes using the exact class hooks in section 4.
  - Map `status` values to `.status-badge--<slug>` and unknown values to `.status-badge--unknown`.
  - Map `priority` values to `.priority--<slug>` with same unknown fallback.
  - Escape/textContent all data (do not inject raw HTML) to avoid XSS from JSON content.
  - If `projects` is empty, show `.empty-state`.
- **Deterministic DOM/class hooks** exactly matching section 4. No renaming without re-agreement.
- **`.vscode/launch.json` requirements:**
  - Strict JSON, no comments, no trailing commas.
  - One configuration named `"Run Project Pulse Dashboard"`.
  - `cwd` = `${workspaceFolder}/app`.
  - Opens `index.html` (not a directory) so the learner sees the dashboard immediately.
  - Deterministic name, working directory, and target file.
- **Out of scope for Coder:** visual design decisions and any edits to `styles.css`.

## 6. Dependencies Between Steps

- Step 1 (contract) **blocks** steps 2, 3, and 4. The JSON shape must exist before rendering; the class hook names must exist before both Coder's DOM and Designer's CSS.
- Step 2 (`project-data.json`) **blocks** step 3's runtime rendering being verifiable (but not step 3's authoring, since the shape is fixed in step 1).
- Step 3 (`index.html` DOM) and step 4 (`styles.css`) share the class hooks agreed in step 1; once locked, they are independent.
- Step 5 (`launch.json`) **depends on** `app/index.html` existing so the launch target is real.
- Step 6 (integration) **depends on** steps 2–5.

## 7. Parallel vs. Sequential Work

**Sequential (must not parallelize):**
- Step 1 before everything else — shared contract prevents drift between DOM classes (Coder) and CSS selectors (Designer), and prevents Coder rendering fields the JSON doesn't define.
- Step 5 after steps 2–4 — a launch config that points to a non-existent `index.html` is a broken deliverable.
- Step 6 last — integration check.

**Parallel (safe, non-overlapping file scopes):**
- After step 1 locks:
  - **Coder** can work on `app/project-data.json` + `app/index.html` simultaneously.
  - **Designer** can work on `app/styles.css` simultaneously.
  - File scopes do not overlap; only the shared hook names couple them, and those are frozen in step 1.

**Justification:** The Orchestrator's delegation rule is "parallel only when file scopes do not overlap and there are no data dependencies." Freezing the schema + class hooks up front removes the data dependency, leaving fully disjoint file ownership.

## 8. Edge Cases

- **Empty `projects` array:** show `.empty-state` with a friendly message; do not render an empty grid.
- **Missing/invalid fields on a project:** render the card with a graceful fallback (e.g. "Unknown owner", "No recent activity") rather than throwing.
- **Unknown `status` or `priority` values:** apply the `--unknown` modifier class; do not crash styling.
- **Long titles / long owner / long activity strings:** must wrap or truncate cleanly; no horizontal overflow of cards or grid.
- **Many projects (e.g. 50+):** grid must remain performant and scrollable; no fixed heights that clip content.
- **JSON fetch failure** (file missing, served with wrong MIME, parse error): show `.error-state`, log to console, do not leave a blank page.
- **Small viewports (≈320px):** single-column layout, tap targets remain comfortable, badges remain legible.
- **Accessibility:** AA contrast on badges and priority accents; visible keyboard focus; semantic landmarks; status/priority meaning conveyed by text as well as color.
- **XSS safety:** all data injected via `textContent` / safe DOM APIs, never `innerHTML` with raw JSON strings.
- **Launch config edge cases:** `cwd` typo, opening the folder instead of `index.html`, or including JSON comments would all break the "no directory listing" requirement — Coder must guard against these.

## 9. Validation Expectations

Success is verified when *all* of the following are true:

1. Opening `app/index.html` in a browser shows a clearly styled dashboard (header + grid of cards), not a bare HTML page.
2. The rendered project cards match the entries in `app/project-data.json` (name, owner, status, recent activity, priority).
3. Each card shows a visible **status badge** with a state-specific visual treatment.
4. Each card shows a visible **priority** treatment that's distinguishable without relying on color alone.
5. Layout is responsive: resizing to ~320px produces a single-column, non-clipped layout; wider viewports produce a multi-column grid.
6. The VS Code "Run Project Pulse Dashboard" launch configuration runs and opens `index.html` from the `app/` folder (not a directory listing).
7. `.vscode/launch.json` is strict JSON (parses with `JSON.parse` / `jq`) and contains no comments.
8. `app/project-data.json` is strict JSON, has a top-level `projects` array, and every entry contains the five required fields.
9. Browser devtools console shows **no errors** during initial load.
10. Empty-state and error-state paths can be triggered manually (e.g. by temporarily emptying `projects` or renaming the JSON file) and render gracefully.
11. Keyboard `Tab` produces a visible focus indicator on any focusable element; badge/priority text meets AA contrast.

## 10. Open Questions

1. **Allowed `status` values** — the brief doesn't enumerate them. Proposed set: `active`, `at-risk`, `blocked`, `shipped`, `on-hold`. Confirm before step 1 locks.
2. **Allowed `priority` values** — proposed `high`, `medium`, `low`. Confirm.
3. **Launch mechanism** — two reasonable interpretations of `.vscode/launch.json`: (a) a browser-debug launch that opens the `file://…/app/index.html` URL directly, or (b) a launch that starts a static server with `cwd = ${workspaceFolder}/app` and opens `index.html`. Both satisfy "cwd is `${workspaceFolder}/app` and opens `index.html`." Orchestrator/learner should confirm which is preferred; Coder should pick a deterministic, comment-free JSON configuration either way.
4. **Optional `app/main.js` split** — is a separate JS file acceptable, or should rendering logic stay inline in `index.html`? Affects file assignments in section 3.
5. **Number/realism of sample projects** — is 4–6 fine, or does Mona want a specific set of project names?
6. **Brand/visual direction** — any existing color palette or brand tokens Designer should honor? If none, Designer will pick a neutral, accessible palette.
