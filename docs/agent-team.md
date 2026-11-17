# Agent team

I am using GitHub Copilot CLI in a Codespace to orchestrate these custom agents while building Mona's Project Pulse dashboard. All definitions live in `.github/agents/`.

| Agent | Target model | Responsibility | Definition |
| --- | --- | --- | --- |
| Orchestrator | Claude Opus 5.5 (copilot) | Breaks the request into phases, delegates to specialists with explicit file scopes, runs work in parallel only when scopes don't overlap, and verifies the integrated result. Does not implement. | `.github/agents/orchestrator.agent.md` |
| Planner | Claude Opus 5.5 (copilot) | Researches the repo and docs, then returns an implementation plan with ordered steps, file assignments, dependencies, edge cases, and validation expectations. Writes no code. | `.github/agents/planner.agent.md` |
| Coder | GPT-5.5 (copilot) | Implements code, fixes bugs, and adds runnable-app support such as `.vscode/launch.json` (cwd `${workspaceFolder}/app`, opens `index.html`) within its assigned files. | `.github/agents/coder.agent.md` |
| Designer | Gemini 3.7 Flash (copilot) | Handles UI/UX, accessibility, and visual design: polished dashboard styling with project cards, status badges, priority treatment, and responsive layout (`.dashboard`, `.project-card`). | `.github/agents/designer.agent.md` |

## Workflow

1. Orchestrator asks Planner for a plan.
2. Orchestrator splits the plan into phases with file ownership.
3. Coder and Designer work in parallel only when their files don't overlap.
4. Orchestrator verifies the combined result and reports back.

None of the agents stage, commit, or push; I control all git operations through Copilot CLI prompts.
