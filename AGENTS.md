# AGENTS.md

Scope: `DesktopCommanderMCP/`

## Purpose

Desktop Commander is a TypeScript MCP server for local filesystem, process, terminal, search, document, configuration, and UI-preview workflows.

This repository may also receive ChatGPT Apps compatibility work. Keep those changes generally useful for anyone running Desktop Commander with ChatGPT or another MCP Apps-compatible host. Do not add machine-specific workflow, tunnel profiles, local paths, private rollout notes, credentials, or workspace-only assumptions to this public repo.

Use this file for broadly applicable repo instructions. Do not create `AGENTS.override.md` for local customization; put local/private operating notes outside this repository.

## Repository Map

| Path | Purpose |
| --- | --- |
| `src/index.ts` | CLI/stdio entrypoint. |
| `src/server.ts` | MCP server capabilities, resource handlers, tool descriptors, and tool-call dispatch. |
| `src/custom-stdio.ts` | stdio transport wrapper and console-output filtering. |
| `src/tools/schemas.ts` | Zod schemas for tool inputs. |
| `src/handlers/` | Tool handler adapters for filesystem, search, terminal/process, edit, and history behavior. |
| `src/tools/` | Tool implementations and domain helpers. |
| `src/ui/` | MCP UI resource contracts, loaders, file preview, config editor, shared widget utilities, and CSS. |
| `src/remote-device/` | Remote Desktop Commander device bridge. |
| `scripts/` | Build, validation, packaging, release, and log utility scripts. |
| `test/` | Node-based regression tests and integration coverage. |
| `plugins/`, `rules/`, `skills/` | Packaging and client-specific distribution assets. |

## Standard Commands

Run from the repository root:

```bash
npm install
npm run build
npm run validate:tools
npm test
```

Use narrower commands when appropriate:

```bash
npm run test:integration
npm run inspector
npm run build:mcpb
```

Do not run release scripts unless explicitly asked:

```bash
npm run release
npm run release:minor
npm run release:major
```

## Change Rules

- Keep existing Claude Desktop and generic MCP behavior working unless the task explicitly targets a breaking migration.
- Before changing an exported tool, schema, UI resource, or transport behavior, inspect callers and related tests.
- Keep tool names, descriptions, schemas, annotations, and handlers synchronized.
- Run `npm run validate:tools` after changing tool descriptors or schemas.
- Run `npm run build` after TypeScript, UI runtime, packaging, or entrypoint changes.
- Run `npm test` for behavior changes that affect tool handlers, filesystem/process/search behavior, UI runtime logic, or security boundaries.
- Do not commit generated logs, local config, built `dist/` output, runtime profiles, tunnel IDs, API keys, OAuth secrets, or private host paths.

## ChatGPT Apps Compatibility

When adding or adapting ChatGPT Apps support:

- Preserve the existing stdio MCP server path. Stdio is valid for ChatGPT when a persistent broker such as `tunnel-client` owns the child MCP process.
- Do not require an HTTP rewrite solely to make local/private ChatGPT testing possible. Add HTTP only when the server needs independent local addressing, deployment, or lifecycle separation.
- Prefer MCP Apps standard metadata and bridge behavior:
  - `_meta.ui.resourceUri` for UI resource linkage
  - `ui/*` bridge methods and notifications where available
  - `outputSchema` for tools returning `structuredContent`
- Keep `openai/outputTemplate` only as ChatGPT compatibility where useful.
- Prefer data/render separation:
  - data tools return model-readable `content` and reusable `structuredContent`
  - render tools attach UI resources and focus on presentation
- Mark side effects accurately with annotations such as `readOnlyHint`, `destructiveHint`, and `openWorldHint`.
- Refresh ChatGPT connector metadata after changing tool names, descriptions, schemas, server instructions, or UI resources.

## Security And Privacy

Desktop Commander has high-impact host capabilities. Treat these as sensitive:

- filesystem reads and writes
- process execution and interactive terminal sessions
- document parsing or generation
- config changes
- remote URL fetches
- telemetry, history, logs, and UI-rendered content

For any new or changed tool:

- validate all model-provided inputs server-side
- enforce path and command constraints consistently
- avoid returning secrets, tokens, credentials, or unnecessary PII in `structuredContent`
- require confirmation semantics for irreversible writes or external actions
- keep prompt-injection risks in mind when rendering or acting on file/document/web content

If stronger isolation is required, prefer Docker or another explicit sandboxing/deployment boundary rather than relying on prompt-level guardrails.
