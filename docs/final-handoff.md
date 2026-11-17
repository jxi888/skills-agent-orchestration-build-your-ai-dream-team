# Project Pulse final handoff

Project Pulse is a one-page dashboard for Mona's team. It shows each project's owner, status, recent activity and priority. A team of custom agents built it, coordinated through GitHub Copilot CLI. The agent definitions are in [`docs/agent-team.md`](agent-team.md).

## Agent team and roles

| Agent | Role in this build |
| --- | --- |
| **Orchestrator** | Split the work into phases. First, it locked a shared contract for class names and data fields. Next, Designer and Coder worked in parallel on separate files. Then it ran an integration check and sent a targeted fix. It did not write code itself. |
| **Planner** | Researched the repo and wrote [`docs/project-pulse-plan.md`](project-pulse-plan.md). The plan covers file assignments, edge cases, validation expectations and open questions. |
| **Designer** | Owned `app/styles.css`, including the visual and accessibility decisions: card layout, status and priority badges, focus outlines, reduced motion, forced colors and responsive grid. Later, it fixed a gap found in the integration check by adding the `summary-item--*` accent rules. |
| **Coder** | Owned `app/index.html`, `app/project-data.json` and `.vscode/launch.json`, and saved the plan to `docs/`. |

## Deliverables

- `app/index.html`: The dashboard page (`<title>Project Pulse</title>`). Its inline script loads `project-data.json` and renders a status summary strip and one `project-card` per project. Each card shows the status, `recentActivity` and a priority badge. The page also shows loading, empty and error messages in an `aria-live` region.
- `app/styles.css`: The dashboard styles. It includes the `.dashboard` layout, `.project-card` cards with `border-radius` and `box-shadow`, and status, priority and summary modifiers (including `--unknown`). It also covers visible focus, `prefers-reduced-motion`, `forced-colors` and a `@media` rule for small screens.
- `app/project-data.json`: Sample data with six projects. Each has `id`, `name`, `owner`, `status`, `recentActivity`, `priority` and `summary`. Together they cover all four statuses and all three priorities.
- `.vscode/launch.json`: Strict-JSON launch configuration **"Run Project Pulse Dashboard"**. It serves `app/` on port 5500 and opens `index.html`.

## How to run

1. Open the **Run and Debug** view in VS Code.
2. Select **Run Project Pulse Dashboard**.
3. Press the green play button.

The configuration runs `python3 -m http.server 5500` with `cwd` set to `${workspaceFolder}/app`. When the server is ready, VS Code opens `http://localhost:%s/index.html` with the port filled in.

Manual fallback: from `app/`, run `python3 -m http.server 5500`, then open <http://localhost:5500/index.html>.

## Dashboard validation results

These checks were run in the Codespace on 2026-11-17.

| Check | Result |
| --- | --- |
| `python3 -m json.tool app/project-data.json` | ✅ Pass |
| `python3 -m json.tool .vscode/launch.json` (strict JSON, no comments; the only `//` is inside the URL) | ✅ Pass |
| Data shape: `projects` is a non-empty list (6 items) and every project has name, owner, status, recentActivity and priority | ✅ Pass |
| Statuses are all in {on-track, at-risk, blocked, completed}; priorities are all in {high, medium, low} | ✅ Pass |
| launch.json: name, `cwd`, `runtimeArgs` include `http.server` and `5500`, `uriFormat` is `http://localhost:%s/index.html` | ✅ Pass |
| `app/index.html` contains `Project Pulse`, `<title>Project Pulse</title>`, `styles.css`, `project-data.json`, `project-card`, `status`, `recentActivity`, `priority` | ✅ Pass |
| `app/styles.css` contains `.dashboard`, `.project-card`, `border-radius`, `box-shadow`, `@media` (responsive rule at `max-width: 600px`) | ✅ Pass |
| Class cross-check: all 37 classes the page outputs, including every `status-badge--*`, `priority-badge--*`, `summary-item--*`, `dashboard-message--error`, `meta-*` and `project-*` class, have a rule in `styles.css` | ✅ Pass (0 missing) |
| Step 3 workflow checks run locally: files exist, keyphrases present, both JSON files parse, launch file contains name and `index.html` | ✅ Pass |
| Served from `app/` on port 5500: `/index.html` returns 200, contains "Project Pulse" and no "Directory listing" | ✅ Pass |
| Served `/project-data.json` and `/styles.css` | ✅ Pass (200, JSON parses) |
| Test server stopped afterwards (port 5500 free) | ✅ Pass |

No check failed.

## Learner handoff

### What's ready

The dashboard, styles, sample data and launch configuration are all in place and passed every automated check above.

### How to run it

Run **Run Project Pulse Dashboard** from the Run and Debug view, or use the manual fallback in [How to run](#how-to-run).

### Remaining manual checks

- **Press play in VS Code.** Confirm the launch config starts the server and opens the dashboard with 6 cards. Each card should show a status badge and a priority label. The agents tested with `python3 -m http.server` directly; nobody has pressed play in the VS Code UI yet.
- **Visual and responsive check.** In DevTools, view the page at about 320px, 768px and 1280px wide. It should show one column on narrow screens, several columns on wide screens and no horizontal scrolling.
- **Contrast check.** Confirm text contrast is at least 4.5:1 using DevTools or Lighthouse.
- **Port 5500 conflicts.** Live Server and other tools often use port 5500. If the launch fails or opens the wrong page, stop the other server or change the port in `.vscode/launch.json`.

### Decisions on the plan's open questions

- **Launch type:** `"type": "node"` with `runtimeExecutable: "python3"`. This needs no Python extension.
- **Theme:** Light theme only (`color-scheme: light`, no dark-mode rules).
- **`recentActivity`:** Stays a plain string.
- **Opening via `file://`:** The page shows a clear error that points to the launch config and the manual URL. There is no built-in copy of the data.
- **Statuses and priorities:** Four statuses and three priorities, as listed in the plan. Risk is covered by `at-risk` plus priority.
- **Summary strip:** Shows a total plus a count for each status, and an "Unknown status" count only when needed.

### Git

No agent staged, committed or pushed anything. You control all git operations.
