# Agent team

To build Mona's Project Pulse dashboard, I use a four-agent custom team defined under `.github/agents/`, orchestrated with GitHub Copilot CLI running in a Codespace.

## Orchestrator

- **Model:** Claude Opus 4.7 (copilot)
- **Responsibility:** Coordinates the Planner, Coder, and Designer agents. Breaks the request into phases based on the Planner's plan, assigns non-overlapping file scopes, runs independent tasks in parallel and dependent/overlapping tasks sequentially, and reports the integrated outcome. Does not implement work itself.
- **Definition:** `.github/agents/orchestrator.agent.md`

## Planner

- **Model:** Claude Opus 4.7 (copilot)
- **Responsibility:** Researches the repository and relevant docs/dependencies, then produces an implementation plan with ordered steps, file assignments, dependencies, parallelizable vs. sequential work, edge cases, and validation expectations. Does not write code.
- **Definition:** `.github/agents/planner.agent.md`

## Coder

- **Model:** GPT-5.5 (copilot)
- **Responsibility:** Implements the code within the file scope assigned by the Orchestrator — dashboard logic, structure, and, when assigned, support files like `.vscode/launch.json` so the app is easy to run and preview. Validates changes before reporting completion.
- **Definition:** `.github/agents/coder.agent.md`

## Designer

- **Model:** Gemini 3.1 Pro (copilot)
- **Responsibility:** Owns UI/UX, accessibility, information architecture, and visual design for the dashboard — project cards, status badges, priority treatment, spacing, and responsive layout — within the scope assigned by the Orchestrator.
- **Definition:** `.github/agents/designer.agent.md`

All four agents avoid staging, committing, or pushing changes; git operations remain under my control via Copilot CLI prompts. I orchestrate this team using GitHub Copilot CLI in a Codespace.
