---
name: joltio-artifacts
description: Create and iterate Joltio dashboards, reports, and data apps from local Svelte folders backed exclusively by governed Joltio functions. Use when a user asks to build, edit, preview, or publish a Joltio artifact.
---

# Joltio Artifacts

Treat the project folder as the source of truth. Never author `artifact.json`; Joltio derives its manifest and `joltio.lock` from the folder.

## Workflow

0. Agree on a brief with the user before writing any UI: who reads the dashboard, what they decide with it, and the one question it answers. Save it as `BRIEF.md` in the project folder, following @references/brief.md. When `BRIEF.md` already exists, read it first and build against it.
1. Run `joltio init <folder> --name "<name>"` when `src/App.svelte` does not exist.
2. Inspect the available governed data with the MCP tool `artifacts_functions_list`, then use `artifacts_functions_run` to verify columns and representative rows before designing the UI. Map each number in the brief to a function following "Choosing a function" below; create the functions that are missing before writing UI.
3. Edit `src/App.svelte`, local components, TypeScript, CSS, and `src/layout.json` directly. Import data through static `joltio:function/<slug>` imports.
4. Run `joltio dev <folder>` while iterating. This compiles and reloads locally on `127.0.0.1`; it does not synchronize a remote draft. Use `--no-open` only when another process owns the browser.
5. Run `joltio preview --remote <folder>` when an explicit remote parity check or shareable draft is needed.
6. Run `joltio publish <folder> --visibility individual|group|common` only when the rendered result and real data have been verified. Publication uploads permitted source and the server validates and compiles it again.

Joltio does not provide or consume model tokens. Use the customer's current Codex or Claude session for code generation and Joltio CLI/MCP for deterministic platform operations.

## Choosing a function

Pick a function for what its output contains, not for words its name shares with the question. A function about "solar generation" does not answer "solar captured price" because both say solar.

A function fits a number of the brief only when all of these hold, checked on its output schema and on rows returned by `artifacts_functions_run`:

- **Measure.** A column already holds that measure in the brief's unit: a captured price is a generation-weighted price in EUR/MWh, not a price column next to a generation column.
- **Period.** It covers the brief's period, or takes it as a parameter. A function fixed to the last seven days does not answer "this month".
- **Grain and scope.** Its time grain, geography, market and technology match the question, with times in Europe/Madrid.

When no function fits, create one instead of combining the closest ones:

0. Check first that no function in the workspace already returns that measure, including unverified ones in `artifacts_functions_list`; reuse it rather than creating a near-duplicate.
1. Explore the source with `data_describe` and `data_query` until one read-only query returns the measure for the brief's period. Take the period (and any unit, technology or market choice) as parameters with defaults.
2. Create it with `artifacts_functions_create` (or `joltio artifacts functions create --input-file function.json`), with a description that says what the output measures, its unit, period and source, plus the output schema and caveats.
3. Run it with representative parameters, check the rows against an independent `data_query` calculation, and import it.
4. Keep it: a function created for a dashboard is likely to be asked for again, so never delete it once the dashboard works. Write `verification.json` next to `BRIEF.md` with the cases you checked and a `note` naming the independent query, and tell the user it is a new, unverified function ready for an owner or administrator to certify (below). Do not certify it yourself; an agent checking its own numbers is not an independent review.
5. When the dashboard is published to a group or the workspace, give the function the same visibility with `artifacts_visibility_update`, so the readers can run it and the next dashboard finds it.

The function stays in the workspace that created it. Never copy it to another workspace or propose it for the global curated catalog: its query reveals what that customer analyses, and other workspaces may be its competitors.

Never derive a number of the brief inside the UI by joining or weighting rows from two functions: their periods, grains and time zones rarely match, and the result is not governed. Formatting, totals of one function's rows and unit conversion are fine. If you cannot create the function (no permission, no source table on the plan), report it as a gap in `BRIEF.md` and leave that number out; do not fill it with the nearest function. This applies even when the user asks for speed or not to be asked: say which number needed a new function.

## Workspace function verification

Create functions with `joltio artifacts functions create --input-file function.json`, execute representative parameters with `joltio artifacts functions run --input-file run.json`, and review the results against an independent source. After checking the description, output schema, chart and CSV/JSON downloads, a workspace owner or administrator can certify that exact version with `joltio artifacts functions verify --input-file verification.json`.

The verification file contains `id`, `expected_version` (returned by creation or visibility publication), `cases` (an array of objects with `params` and complete `expected_rows`), and a `note` explaining the independent review. Supply actual reviewed rows for the current workspace. Include defaults, a non-default parameter and an empty result where applicable. The server reruns every case, checks exact values, primitive output types and the declared chart columns, and requires at least one non-empty result. Verification records the reviewer and exact definition in the workspace audit. A changed version, name or body invalidates that workspace certification; creating or publishing alone never certifies a function. This does not add the function to the global curated catalog.

## Boundaries

- Use only relative local imports, `joltio:runtime`, and `joltio:function/<slug>`.
- Do not use `fetch`, WebSockets, workers, WASM, dynamic imports, `eval`, secrets, `.env`, arbitrary packages, or direct database access.
- Keep data access inside governed functions. If the needed function is absent, create it as described in "Choosing a function" instead of embedding SQL or an endpoint in the artifact.
- Preserve the existing design when editing unless the user asks for a redesign.
- Use managed layout blocks only for regions intended for drag/resize. Keep freeform areas code-managed.
- Show only the markets and programs of Joltio Settlements, with their names, and give every chart encoding (color, stroke, marker, fill, band) a direct label or a legend. Details in @references/brief.md.

Read @references/patterns.md when choosing an interaction or visualization pattern.
