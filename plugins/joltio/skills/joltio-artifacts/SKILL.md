---
name: joltio-artifacts
description: Create and iterate Joltio dashboards, reports, and data apps from local Svelte folders backed exclusively by governed Joltio functions. Use when a user asks to build, edit, preview, or publish a Joltio artifact.
---

# Joltio Artifacts

Treat the project folder as the source of truth. Never author `artifact.json`; Joltio derives its manifest and `joltio.lock` from the folder.

## Workflow

0. Agree on a brief with the user before writing any UI: who reads the dashboard, what they decide with it, and the one question it answers. Save it as `BRIEF.md` in the project folder, following @references/brief.md. When `BRIEF.md` already exists, read it first and build against it.
1. Run `joltio init <folder> --name "<name>"` when `src/App.svelte` does not exist.
2. Inspect the available governed data with the MCP tool `artifacts_functions_list`, then use `artifacts_functions_run` to verify columns and representative rows before designing the UI.
3. Edit `src/App.svelte`, local components, TypeScript, CSS, and `src/layout.json` directly. Import data through static `joltio:function/<slug>` imports.
4. Run `joltio dev <folder>` while iterating. This compiles and reloads locally on `127.0.0.1`; it does not synchronize a remote draft. Use `--no-open` only when another process owns the browser.
5. Run `joltio preview --remote <folder>` when an explicit remote parity check or shareable draft is needed.
6. Run `joltio publish <folder> --visibility individual|group|common` only when the rendered result and real data have been verified. Publication uploads permitted source and the server validates and compiles it again.

Joltio does not provide or consume model tokens. Use the customer's current Codex or Claude session for code generation and Joltio CLI/MCP for deterministic platform operations.

## Workspace function verification

Create functions with `joltio artifacts functions create --input-file function.json`, execute representative parameters with `joltio artifacts functions run --input-file run.json`, and review the results against an independent source. After checking the description, output schema, chart and CSV/JSON downloads, a workspace owner or administrator can certify that exact version with `joltio artifacts functions verify --input-file verification.json`.

The verification file contains `id`, `expected_version` (returned by creation or visibility publication), `cases` (an array of objects with `params` and complete `expected_rows`), and a `note` explaining the independent review. Supply actual reviewed rows for the current workspace. Include defaults, a non-default parameter and an empty result where applicable. The server reruns every case, checks exact values, primitive output types and the declared chart columns, and requires at least one non-empty result. Verification records the reviewer and exact definition in the workspace audit. A changed version, name or body invalidates that workspace certification; creating or publishing alone never certifies a function. This does not add the function to the global curated catalog.

## Boundaries

- Use only relative local imports, `joltio:runtime`, and `joltio:function/<slug>`.
- Do not use `fetch`, WebSockets, workers, WASM, dynamic imports, `eval`, secrets, `.env`, arbitrary packages, or direct database access.
- Keep data access inside governed functions. If the needed function is absent, tell the user what governed function must be added instead of embedding SQL or an endpoint in the artifact.
- Preserve the existing design when editing unless the user asks for a redesign.
- Use managed layout blocks only for regions intended for drag/resize. Keep freeform areas code-managed.
- Show only the markets and programs of Joltio Settlements, with their names, and give every chart encoding (color, stroke, marker, fill, band) a direct label or a legend. Details in @references/brief.md.

Read @references/patterns.md when choosing an interaction or visualization pattern.
