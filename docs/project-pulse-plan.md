# Project Pulse implementation plan

## 1. Summary

Project Pulse is a small static dashboard. It shows Mona's team projects as cards with each project's owner, status, recent activity, priority and a short summary. It loads its data from a JSON file. A VS Code launch configuration named **Run Project Pulse Dashboard** starts a local server in `app/` and opens `index.html`, so you see the dashboard instead of a directory listing.

What I found in the repository:

- `app/` is currently empty. `.vscode/` contains only `tasks.json`, which must not be changed.
- The brief (`.github/project-pulse-brief.md`) asks for a top-level `projects` array. Each project must have `name`, `owner`, `status`, `recentActivity` and `priority`. It also asks for a short contributor-friendly summary.
- The Step 3 grader (`.github/workflows/3-step.yml`) looks for exact phrases. The work must keep these:
  - `app/index.html` must contain: `Project Pulse`, `styles.css`, `project-data.json`, `project-card`, `status`, `recentActivity`, `priority`.
  - `app/styles.css` must contain: `.dashboard`, `.project-card`, `border-radius`, `box-shadow`.
  - `app/project-data.json` must pass `python3 -m json.tool` and contain `projects`, `name`, `owner`, `status`, `recentActivity`, `priority`.
  - `.vscode/launch.json` must pass `python3 -m json.tool` and contain `Run Project Pulse Dashboard` and `index.html`.
- The Step 3 prompt fixes these values: the command `python3 -m http.server 5500`, serving from the app directory, and a `serverReadyAction` that opens `http://localhost:%s/index.html`.
- No extra files are assigned, so all JavaScript stays inline in `app/index.html`. Do not create `app.js`.
- The Orchestrator (not the Planner) saves this plan to `docs/project-pulse-plan.md`. No agent stages, commits or pushes; the learner controls git.

## 2. File assignments

| File | Owner agent | Purpose |
| --- | --- | --- |
| `app/index.html` | **Coder** (follows the Designer's markup and class guidance) | Page structure: `<title>Project Pulse</title>`, `<h1>`, a `.dashboard` wrapper, and a link to `styles.css`. An inline `<script>` loads `project-data.json` with `fetch` and builds one `.project-card` per project. It also shows loading, empty and error states. |
| `app/styles.css` | **Designer** | All visual design: layout of `.dashboard`, the card grid, `.project-card` (with `border-radius` and `box-shadow`), status badges, priority treatment, responsive breakpoints, focus styles and reduced-motion handling. |
| `app/project-data.json` | **Coder** | Fixed sample data in strict JSON: `{ "projects": [ ... ] }`, covering every status and priority value at least once. |
| `.vscode/launch.json` | **Coder** | Strict JSON with no comments. Defines **Run Project Pulse Dashboard**, which runs `python3 -m http.server 5500` with `cwd` set to `${workspaceFolder}/app` and opens `http://localhost:%s/index.html`. |

Files no one may touch: `.vscode/tasks.json`, `.devcontainer/**`, `.github/**`, `scripts/**`, `docs/agent-team.md`.

## 3. Designer responsibilities

The Designer owns `app/styles.css` only, and gives the Coder guidance for `app/index.html`.

1. **Agree the class names first (Phase 0, below).** Confirm or adjust the class names in section 6. The Coder will build the HTML against this list, so it must not change after Phase 1 starts.
2. **Make it look like a real dashboard, not a bare page:**
   - A header band with the "Project Pulse" title and a short subtitle.
   - A summary strip with project counts by status.
   - A card grid.
   - Clear typography, a neutral background, and cards that stand out from it using `border-radius`, `box-shadow`, contrast and generous padding.
3. **Status badges:** pill-shaped `.status-badge` elements, with modifiers `--on-track`, `--at-risk`, `--blocked`, `--completed` and `--unknown`. The badge text always shows the status, so meaning never depends on colour alone. Text must meet WCAG AA contrast (at least 4.5:1).
4. **Priority treatment:**
   - High-priority cards get a coloured left border via `.project-card[data-priority="high"]` and a bold "High priority" label.
   - Medium and low priority get progressively quieter styling.
   - `--unknown` gets a neutral style.
5. **Responsive layout:**
   - Grid using `repeat(auto-fill, minmax(280px, 1fr))`.
   - One column below about 600px.
   - No horizontal scrolling at 320px wide.
   - Long names and activity text wrap (`overflow-wrap: anywhere`).
6. **Accessibility styling:**
   - Visible `:focus-visible` outlines.
   - `@media (prefers-reduced-motion: reduce)` turns off hover transitions.
   - Text sizes in `rem`.
   - Optionally, a `@media (prefers-color-scheme: dark)` theme that still meets contrast requirements.
7. **State styling:** `.dashboard-message` covers loading and empty states; `.dashboard-message--error` covers the error state.
8. **Report back:** design decisions, tradeoffs, and checks the Orchestrator should run. The Designer has no `execute` tool, so it relies on the Orchestrator and Coder for any command-line checks.

## 4. Coder responsibilities

The Coder owns `app/index.html`, `app/project-data.json` and `.vscode/launch.json`.

1. **`app/project-data.json`**
   - Strict JSON (no comments, no trailing commas) with a top-level `"projects"` array of 6 projects.
   - Required fields per project: `name`, `owner`, `status`, `recentActivity`, `priority`. Recommended: `id` (a short unique name such as `"docs-refresh"`) and `summary` (one sentence).
   - Allowed `status` values: `"on-track"`, `"at-risk"`, `"blocked"`, `"completed"`.
   - Allowed `priority` values: `"high"`, `"medium"`, `"low"`.
   - `recentActivity` is a short readable string, for example `"Merged accessibility fixes for the card grid"`.
   - Cover every status and every priority at least once.
2. **`app/index.html`**
   - `<!DOCTYPE html>`, `<html lang="en">`, `<meta charset="utf-8">`, a viewport meta tag, `<title>Project Pulse</title>`, `<link rel="stylesheet" href="styles.css">`.
   - Semantic structure: `<header>` containing an `<h1>Project Pulse</h1>`, then `<main class="dashboard">`.
   - Each project card is an `<article class="project-card">` with an `<h2>`.
   - The inline script uses the literal path `project-data.json` (`fetch("project-data.json")`) and reads `data.projects`.
   - Each card shows the project's name, owner, status, recentActivity, priority and optional summary. The visible labels "Status", "Recent activity" and "Priority" must also appear in the markup.
   - Build cards with `document.createElement` and `textContent`, not `innerHTML`, so text from the data file is never run as HTML.
   - Before using them as CSS modifiers, clean the `status` and `priority` values: trim spaces, make lowercase, and turn spaces or underscores into hyphens. Map anything outside the allowed lists to `unknown`, but still display the original text.
   - Sort cards in a fixed order: priority high → medium → low → unknown, then by `name` alphabetically.
   - Calculate the counts for the summary strip from the data.
   - Show loading, empty and error states in a region marked `role="status"` with `aria-live="polite"` (details in section 8).
3. **`.vscode/launch.json`** — strict JSON with no comments. Recommended shape:
   - `"version": "0.2.0"`
   - One entry in `"configurations"` with:
     - `"name": "Run Project Pulse Dashboard"`
     - `"type": "node"` and `"request": "launch"` (VS Code's built-in JavaScript debugger, so no extra extension is needed)
     - `"runtimeExecutable": "python3"`
     - `"runtimeArgs": ["-u", "-m", "http.server", "5500"]`. The `-u` flag stops Python holding back its "Serving HTTP" message, which the launch config waits for before opening the browser.
     - `"cwd": "${workspaceFolder}/app"`
     - `"console": "integratedTerminal"`
     - `"serverReadyAction"`:
       - `"action": "openExternally"`
       - `"pattern": "Serving HTTP on .* port ([0-9]+)"`
       - `"uriFormat": "http://localhost:%s/index.html"`
4. Run the validation checks in section 9 and report what changed, what was checked, and any remaining risk.

## 5. Ordered implementation steps

| Step | Phase | Agent | Files | Description |
| --- | --- | --- | --- | --- |
| 0 | Phase 0: agree the shared contract | Orchestrator, Designer to confirm | none (this plan) | Lock the data shape and class names from section 6. Changes after this point need a new sequential round. |
| 1a | Phase 1 (parallel) | Designer | `app/styles.css` | Write all styles against the class names, including every badge and priority modifier and every state class. |
| 1b | Phase 1 (parallel) | Coder | `app/project-data.json` | Write the 6 sample projects covering all statuses and priorities. |
| 1c | Phase 1 (parallel) | Coder | `.vscode/launch.json` | Write the launch config from section 4. It does not depend on any other file's content. |
| 1d | Phase 1 (after 1b, same Coder) | Coder | `app/index.html` | Write the markup and inline rendering script using the class names and the data shape. |
| 2 | Phase 2: check the pieces fit | Orchestrator | read only | Confirm every class the HTML outputs has a matching CSS rule, and that the JSON field names match what the script reads. |
| 3 | Phase 3: fixes (one at a time) | Designer and/or Coder | only the file each owns | Fix any mismatches found in step 2. Never two agents on the same file at once. |
| 4 | Phase 4: validation | Coder runs commands; Orchestrator reviews | read only | Run section 9 and report results to the learner. |

## 6. Dependencies between steps/files

**Data shape (`app/project-data.json` → `app/index.html`)**

```json
{
  "projects": [
    {
      "id": "string",
      "name": "string",
      "owner": "string",
      "status": "on-track | at-risk | blocked | completed",
      "recentActivity": "string",
      "priority": "high | medium | low",
      "summary": "string (optional)"
    }
  ]
}
```

The script in `app/index.html` reads exactly these keys. If a key is renamed in one file, the other must change in the same phase.

**Class names (`app/index.html` → `app/styles.css`)**

| Class or attribute | Used on |
| --- | --- |
| `.dashboard` | the `<main>` wrapper |
| `.dashboard-header`, `.dashboard-subtitle` | the header area |
| `.dashboard-summary`, `.summary-item` | the counts strip |
| `.project-grid` | the `<ul>`/`<section>` containing the cards |
| `.project-card` | each `<article>`, with `data-status="<cleaned status>"` and `data-priority="<cleaned priority>"` |
| `.project-card__header`, `.project-name`, `.project-owner`, `.project-summary`, `.project-activity`, `.project-meta` | parts of a card |
| `.status-badge` + `.status-badge--on-track`, `--at-risk`, `--blocked`, `--completed`, `--unknown` | status badges |
| `.priority-badge` + `.priority-badge--high`, `--medium`, `--low`, `--unknown` | priority labels |
| `.dashboard-message` + `.dashboard-message--error` | loading, empty and error states |

**Other dependencies**

- `app/index.html` refers to `styles.css` and `project-data.json` by relative path, so all three files must stay together in `app/`.
- `.vscode/launch.json` depends on `app/index.html` existing at the server root. The server root comes from `"cwd": "${workspaceFolder}/app"` and the URL `http://localhost:%s/index.html`. Change the directory or file name and the launch config breaks.
- The JSON can only be loaded over HTTP, so how the dashboard is run depends on the launch config (see section 8).
- Step 2 depends on 1a–1d. Step 4 depends on step 3.

## 7. Parallel work decisions

**What runs in parallel**

- **Designer (`app/styles.css`) alongside Coder (`app/project-data.json`, `app/index.html`, `.vscode/launch.json`).**
  - Why: the two agents own different files, and the only thing they share (the class names and data shape) is fixed in Phase 0 before either starts. CSS only reads class names, not data, so the Designer doesn't need the Coder's output.
- **`.vscode/launch.json` alongside everything else.**
  - Why: it only needs the agreed constants (`app/`, `index.html`, port 5500), not the content of any other file.

**What runs sequentially**

- **Phase 0 before Phase 1.** Both agents depend on the class names; if they are not fixed first, the HTML and CSS drift apart.
- **`app/project-data.json` before `app/index.html` (same Coder).** The rendering code and the sample data should be written against the same field names. This costs nothing because one agent does both.
- **Phase 2 checks and Phase 3 fixes after Phase 1.** Mismatches only show up once both sides exist.
- **Fixes in Phase 3 run one at a time per file.** For example, if the Designer suggests HTML changes, the Coder applies them to `app/index.html` in a separate step. Two agents never edit the same file at once.
- **Validation (Phase 4) runs last** because it checks the combined result.

## 8. Edge cases

1. **Opening `index.html` directly from disk (`file://`).** Browsers block `fetch` of local files. Show a `.dashboard-message--error` that tells the user to start **Run Project Pulse Dashboard** (or `python3 -m http.server 5500` in `app/`) and open `http://localhost:5500/index.html`. Do not fail silently, and do not show a blank page.
2. **Fetch fails or the server returns an error (404/500).** Check `response.ok`. Throw an error that includes the HTTP status, catch it, and show the error message, logging details with `console.error`.
3. **Badly formed JSON.** `response.json()` throws an error; show the same error state with a "could not read project data" message.
4. **`projects` missing or not a list.** Treat this as a data error and show a clear message rather than crashing.
5. **Empty `projects` list.** Show an empty state such as "No projects to show yet."; the summary counts show 0.
6. **Unknown or oddly written status or priority** (for example `"At Risk"`, `"on_track"`, `"paused"`). Clean the value first. If it is still not in the allowed list, use the `--unknown` modifier and show the original text.
7. **Missing required fields on a project.** Show a fallback such as "Unassigned" for owner or "No recent activity" for recentActivity, instead of showing `undefined`. Skip a project only if it has no `name`.
8. **Long text or special characters in names or activity.** CSS wraps the text; `textContent` means characters like `<script>` are shown as plain text, never run.
9. **Port 5500 already in use** (for example by a previous run or the Live Server extension). Python fails with "Address already in use". Stop the earlier server first. The port stays fixed at 5500 so the setup is predictable.
10. **Browser opens before the server is ready, or never opens.** Python normally holds back its "Serving HTTP" message, so the browser-open trigger never fires. The `-u` flag fixes this. If the pattern still doesn't match, open `http://localhost:5500/index.html` manually.
11. **Codespaces port forwarding.** The `localhost` URL is forwarded automatically when opened externally. If the browser shows a directory listing, the `cwd` or `uriFormat` is wrong.
12. **Browser serving an old copy of the JSON.** Optionally use `fetch("project-data.json", { cache: "no-store" })` so edits show up straight away.
13. **No JavaScript.** Add a `<noscript>` message explaining that the dashboard needs JavaScript.

## 9. Validation expectations

**JSON files are strict and valid**

- `python3 -m json.tool app/project-data.json` succeeds.
- `python3 -m json.tool .vscode/launch.json` succeeds. Since this parser rejects comments and trailing commas, success also proves launch.json is strict JSON.
- Running `grep -nE '//|/\*' .vscode/launch.json` finds nothing except the `//` inside URLs.

**Data shape**

- A short Python check confirms that `projects` is a non-empty list.
- Every project has `name`, `owner`, `status`, `recentActivity` and `priority`.
- Every status value is one of the four allowed values and every priority value is one of the three.

**launch.json contents**

- `name` is `"Run Project Pulse Dashboard"`.
- `cwd` is exactly `"${workspaceFolder}/app"`.
- The command arguments include `http.server` and `5500`.
- `serverReadyAction.uriFormat` is exactly `"http://localhost:%s/index.html"`.

**Phrases the grader looks for**

- `app/index.html` contains `Project Pulse`, `styles.css`, `project-data.json`, `project-card`, `status`, `recentActivity`, `priority`.
- `app/styles.css` contains `.dashboard`, `.project-card`, `border-radius`, `box-shadow`.
- `.vscode/launch.json` contains `index.html`.

**Page renders cards**

- From `app/`, start `python3 -m http.server 5500` in the background.
- `curl -s http://localhost:5500/index.html` returns HTML containing `Project Pulse`, not "Directory listing for".
- `curl -s http://localhost:5500/project-data.json` returns the JSON.
- Running **Run Project Pulse Dashboard** in VS Code opens the dashboard with 6 visible cards, each showing a status badge and a priority label. Stop the server afterwards.

**Class names match**

- Every class the script outputs has at least one rule in `styles.css`, including all `--unknown` modifiers and the message classes.

**Responsive**

- Use browser DevTools at about 320px, 768px and 1280px wide.
- One column on narrow screens, several columns on wide screens.
- No horizontal scrolling, and text wraps cleanly.

**Accessibility basics**

- `lang="en"`, one `<h1>`, an `<h2>` per card, a `<main>` landmark, and the `aria-live` message region.
- Status and priority are readable as text, not only as colour.
- Text contrast is at least 4.5:1 (check in DevTools or Lighthouse).
- Visible focus outlines, and reduced-motion is respected.

**State checks**

- Temporarily point the page at a missing file, or open it via `file://`, and confirm the error message appears.
- Temporarily use an empty `projects` list and confirm the empty-state message. Undo both changes afterwards.

**Do not use `scripts/validate-exercise.sh` as a pass/fail gate.** It checks the template itself, and it deliberately fails once `app/*` and `.vscode/launch.json` are tracked in git.

## 10. Open questions

1. **Launch type.** I recommend `"type": "node"` with `runtimeExecutable: "python3"` because the debugger it needs is built into VS Code; the devcontainer doesn't install the Python extension. The alternative, `"type": "debugpy"` with `"module": "http.server"`, is cleaner but needs the Python extension. Which should the Coder use? I haven't run the recommended setup in this Codespace, so it should be checked once by pressing play.
2. **Fallback when opened via `file://`.** Should the page include a built-in copy of the data so it still shows something? That would duplicate the data and could drift out of sync. I recommend showing a clear error and steering users to the launch config.
3. **Final status and priority values.** Confirm the four statuses (`on-track`, `at-risk`, `blocked`, `completed`) and three priorities (`high`, `medium`, `low`). The brief says "priority or risk level". Should risk be its own field, or covered by `at-risk` plus priority?
4. **`recentActivity` format.** Should it stay a plain string, or become something structured like `{ "description", "date" }` for sorting or "updated X days ago" text? A plain string keeps the grader's checks simple.
5. **Dark mode.** Should the Designer add a `prefers-color-scheme: dark` theme, or ship light only?
6. **Summary strip contents.** Counts by status only, or also a count of high-priority projects?
