# AGENTS.md

Guidance for coding agents (Claude Code loads it via `CLAUDE.md`) working in this repository. This root file stays minimal on purpose: orientation, commands, layout, and an index of pointers. Every rule lives behind its pointer — subsystem invariants in `docs/agents/*.md`, coding standards and review rules in [`CODING_STANDARDS.md`](CODING_STANDARDS.md) — so before changing an area, read the file its pointer names.

## Project overview

K-Dense BYOK is a local AI research-assistant app ("Kady") that brings the user's own model credentials (API keys or supported subscriptions). It runs natively on macOS, Linux, and Windows. It is one repo with **two** runtime services started together by the cross-platform launcher `start.mjs` (wrapped by `./start.sh` on macOS/Linux and `start.cmd` on Windows):

| Service | Port | Code |
|---|---|---|
| Frontend (Next.js 16 / React 19) | 3000 | `web/` |
| Backend (TypeScript + Pi coding-agent SDK) | 8000 | `server/` |

The backend embeds the **Pi SDK** and runs a **single flat agent**: built-in tools plus a `subagent`/`bg_wait` delegation pair, an `interview` clarifying-questions tool, web tools, live PDF annotations, durable Modal compute, `generate_image`, and MCP tools. There is no orchestrator/expert split and no model proxy — models go directly to **OpenRouter**, a connected Pi OAuth provider, **NVIDIA NIM** (API key), or a local **Ollama** / OpenAI-compatible server; everything runs locally, and user data lives in `projects/`. The full tool inventory and the turn-by-turn flow: [`docs/agents/agent-runtime.md`](docs/agents/agent-runtime.md).

## Commands

Backend (`cd server` first; Node ≥ 22.19 recommended):

```bash
npm install                 # install deps
npm run dev                 # tsx watch on port 8000
npm run start               # run backend (tsx)
npm run prep                # ensure default project + seed scientific skills
npm run typecheck           # tsc --noEmit
npm test                    # vitest
```

Frontend (`cd web` first):

```bash
npm install
npm run dev                 # Next.js dev server (port 3000)
npm run build               # production build
npm run test                # vitest
```

Full app (both services):

```bash
./start.sh                  # installs deps, seeds skills, starts backend + frontend
start.cmd                   # same, on Windows (both wrap `node start.mjs`)
```

## Project / sandbox layout

```
projects/
├── index.json                        # registry
└── <projectId>/
    ├── project.json                  # metadata (ProjectMeta)
    └── sandbox/                       # Pi agent cwd; files visible to all tabs
        ├── user_data/                # uploads
        ├── .pi/skills/               # per-project skills (Pi-discovered)
        ├── .pi/skills-archived/      # unchanged skills retired upstream
        ├── .pi/sessions/             # Pi JSONL session files (one per tab)
        ├── .kady/skills-sync.json     # upstream hashes + pending local conflicts
        ├── .kady/runs/<sessionId>/costs.jsonl   # cost ledger
        ├── .kady/provenance/<sessionId>/steps.jsonl  # observed step provenance
        └── .kady/modal/jobs/<jobId>/ # durable Modal state, events, and logs
```

## Releases

- `server/package.json` `version` is the single source of truth for the app version. The web build reads it at build time (`web/next.config.ts` injects `NEXT_PUBLIC_APP_VERSION`); `web/package.json` deliberately has no `version` field.
- Releasing = bump `server/package.json` version and push/merge to `main`. The `Release` workflow (`.github/workflows/release.yml`) runs on every push to `main`, and if the tag `v<version>` doesn't exist yet it creates it plus a GitHub release with auto-generated notes. No manual tagging.

## Testing notes

- Backend tests: `cd server && npm test` (vitest, in `server/test/`). `KADY_PROJECTS_ROOT` is pointed at a temp dir via `vitest.config.ts`.
- Frontend tests: `cd web && npm test` (vitest); `npx tsc --noEmit` for the frontend typecheck.

## Rules index

Each entry names the work that requires its file; the target holds that area's invariants and failure modes.

- **Model or provider change** — adding or changing a provider, model ref, API key, OAuth login, billing class, Fusion preset, local or custom model server, thinking level, or one-shot model call, or diagnosing a provider refusal → [`docs/agents/models-and-providers.md`](docs/agents/models-and-providers.md)
- **Settings or env change** — touching env loading, the Pi agent directory, credentials routes, or any Settings panel → [`docs/agents/configuration.md`](docs/agents/configuration.md)
- **Run-path change** — touching the run pipeline, session registry, SSE events, image attachments, system runs, the extension lifecycle, the `interview` tool, or compaction → [`docs/agents/agent-runtime.md`](docs/agents/agent-runtime.md)
- **Cost or budget change** — touching the cost ledger, tool-usage split, spend caps, or budget reservations → [`docs/agents/cost-and-budgets.md`](docs/agents/cost-and-budgets.md)
- **Provenance change** — touching step recording, the sandbox scanner, subagent harvest, environment snapshots, or lineage lookup → [`docs/agents/provenance.md`](docs/agents/provenance.md)
- **Skills or templates change** — touching skill fetch/sync/install, prompt templates, or slash-command expansion → [`docs/agents/skills.md`](docs/agents/skills.md)
- **Sandbox or preview change** — touching sandbox file APIs, annotation sidecars, LaTeX/SyncTeX, scientific preview helpers, or Office preview/editing → [`docs/agents/sandbox-and-previews.md`](docs/agents/sandbox-and-previews.md)
- **Delegation change** — touching `subagent`/`bg_wait`, specialist personas or tool overrides, workflow scripts, verification gates, schedules/missions, per-agent memory, or the supervisor/watchdog → [`docs/agents/subagents.md`](docs/agents/subagents.md)
- **MCP change** — touching MCP server config, codemode or tool exposure, the lead's tool denylist, or MCP OAuth/Paperclip/Radius → [`docs/agents/mcp.md`](docs/agents/mcp.md)
- **Modal change** — touching the Modal tool family, job manager, instance catalog, batches, or restart recovery → [`docs/agents/modal-compute.md`](docs/agents/modal-compute.md)
- **Notebook or evidence change** — touching the lab notebook, evidence links, frozen plans, robustness workflows, research memory, evidence packages, or What-next proposals → [`docs/agents/notebook.md`](docs/agents/notebook.md)
- **Security boundary change** — touching `request-guard`/`auth`, the access token, browser URL builders, the raw-data guard, or the bash classifier → [`docs/agents/security.md`](docs/agents/security.md)
- **Any non-trivial change or review** — toolchain rules, Windows portability, the pinned harness, and duplicates that must stay in sync → [`CODING_STANDARDS.md`](CODING_STANDARDS.md)

User-facing feature guides live in `docs/` and are indexed by `docs/README.md`; everything under `docs/agents/` is agent-facing.
