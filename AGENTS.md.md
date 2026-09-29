# AGENTS.md

Instructions for any AI coding agent working on this project. Read this file fully before making changes.

## Project overview

Taskboard is a single-page to-do app. It runs entirely in the browser with no backend, no build step and no dependencies.

**Features (all must keep working):**
- Create tasks (title, notes, priority, due date, repeat)
- View tasks (list, expandable notes, priority colour, overdue highlight)
- Edit and update tasks (edit dialog, done checkbox)
- Delete tasks (single delete with confirmation, clear completed)
- Notes on every task
- Extra features: recurring tasks (daily, weekly, monthly), search, filters (All/Active/Done), sorting, progress bar, JSON export/import

## Project structure

```
/
├── AGENTS.md        # this file
└── index.html   # entire app: HTML, CSS and JavaScript in one file
```

Do not split the app into multiple files, add a framework, or add a build tool unless the user explicitly asks.

## How to run

1. Open `index.html` directly in a browser, or
2. Serve the folder locally: `python3 -m http.server 8000`, then visit `http://localhost:8000/index.html`

## Coding rules

- Use plain HTML, CSS and vanilla JavaScript only. No external scripts, fonts, CDNs or network requests. The app must work offline.
- Render user-entered text with `textContent`. Never put user data into `innerHTML`, `eval` or inline event attributes.
- Keep all logic in small, single-purpose functions with clear names. Prefer editing existing functions over adding duplicates.
- Use `const` and `let`, never `var`. Use strict equality (`===`).
- Dates are stored as `YYYY-MM-DD` strings in local time. Do not use `toISOString()` for "today".
- Wrap every `localStorage` read and write in `try/catch`. The app must still load if storage is empty, blocked or corrupted.
- Colours come from CSS variables on `:root`. Support light and dark mode through `prefers-color-scheme`.
- Keep the layout responsive. It must be usable at 360px width without horizontal scrolling.
- Accessibility floor: every control has a text label or `aria-label`, keyboard focus is visible, and `prefers-reduced-motion` is respected.
- UI text uses sentence case and active verbs ("Save task", not "Submit"). Keep an action's name the same everywhere it appears.

## Data model and storage

Tasks are saved in `localStorage` under the key `taskboard.v1` as a JSON array:

```json
{
  "id": 1700000000000.123,
  "title": "string, required, max 140 chars",
  "notes": "string, may be empty",
  "priority": "low | medium | high",
  "due": "YYYY-MM-DD or empty string",
  "repeat": "daily | weekly | monthly or empty string",
  "done": false,
  "spawned": false,
  "created": 1700000000000
}
```

Rules:
- Never rename or remove existing fields without a migration. Old saved data and old backup files must keep loading.
- Adding a field is fine, but it needs a safe default when missing.
- If the shape changes in a breaking way, bump the storage key (for example `taskboard.v2`) and migrate from the old key.

## Recurring task behaviour

- Completing a recurring task creates one new copy with the next due date. The completed task stays in the Done list.
- `spawned` prevents duplicate copies when a task is unticked and ticked again.
- If the task has no due date, the next date counts from today.
- If the next date would be in the past, keep advancing until it is today or later.
- Monthly repeats clamp to the last day of shorter months (31 Jan becomes 28 or 29 Feb).

## Validation and testing requirements

**Endpoint rule (mandatory):** Write tests for every endpoint you create, and always validate that these endpoints are working before reporting the task as done. Tests must cover the success case, invalid input, and missing or unauthorised access. Run the tests and report the results. Today the app has no backend and therefore no endpoints, so this rule applies as soon as any API, serverless function or server route is added.

There is no automated test suite for the front end. Before saying a task is finished, run the app and manually verify each item below. Report the results honestly: say what you tested, what passed, what failed, and what you could not test.

**Core features**
- [ ] Create a task with title only, and another with every field filled
- [ ] An empty or whitespace-only title is rejected
- [ ] Edit a task and confirm the changes show in the list
- [ ] Tick and untick a task, and confirm the progress bar and counter update
- [ ] Delete one task (cancel the confirmation once, then confirm it)
- [ ] Notes expand and show line breaks correctly

**Extra features**
- [ ] Search finds text in titles and in notes
- [ ] Each filter tab shows the right tasks
- [ ] Each sort option orders tasks correctly
- [ ] A task with a past due date shows as overdue, and stops when it is done
- [ ] Ticking a daily, weekly and monthly task creates the correct next due date, once only
- [ ] Export downloads a valid JSON file, and importing it restores the same tasks
- [ ] Importing a non-JSON or invalid file shows an error and does not wipe existing tasks

**Robustness**
- [ ] Refresh the page: all tasks persist
- [ ] Titles or notes containing `<script>` or HTML render as plain text
- [ ] Very long titles wrap and do not break the layout
- [ ] Works at 360px width and in dark mode
- [ ] No errors in the browser console during any of the above

## Security rules

- Never commit secrets: no API keys, tokens, passwords or `.env` files. If a secret is needed, read it from an environment variable and document the variable name only.
- Treat all input as untrusted, including imported backup files. Validate type and shape before using it.
- Do not add third-party scripts, trackers or analytics.
- Do not log personal or task content to the console in production code.
- If an endpoint is ever added: validate all input on the server, return correct HTTP status codes, and never expose stack traces to the client.

## Git and commit conventions

- Small, focused commits, each doing one thing.
- Commit messages are short and in the imperative: `Add weekly repeat option`, `Fix overdue date check`.
- Never commit generated files, large binaries or editor settings.
- Do not rewrite history or force-push unless the user asks.

## Deployment

- The app is static and is hosted on Vercel or Netlify straight from the `main` branch. There is no build command and no output directory.
- `index.html` must stay at the repository root so the live URL opens the app.
- After any change to `main`, re-test the core features on the live URL, not only locally.

## Boundaries

- Do not delete or overwrite user data, backups or the `taskboard.v1` storage key without a migration.
- Do not remove tests, checklist items or safety checks to make a change pass.
- Ask before adding a dependency, a backend, an account system or a database.

## Definition of done

A task is done only when all of these are true:
1. The requested change works.
2. The testing checklist items affected by the change pass, and the core create, edit, delete and persistence checks still pass.
3. There are no console errors.
4. Any new endpoint has tests that pass.
5. This file is updated if conventions, the data model or the feature list changed.
6. The reply states what was changed, what was tested, and what is still unverified.

## Workflow expectations

- Make the smallest change that solves the problem. Do not rewrite working code or restyle the app unless asked.
- Do not remove or change an existing feature to make a new one easier.
- After any change, re-run the checklist items affected by it, plus a quick check that create, edit, delete and persistence still work.
- If a requirement is unclear, ask one short question before building.
- If you find a bug outside the current task, mention it in your reply instead of silently fixing it.
- Update this file when you change conventions, the data model or the feature list.
