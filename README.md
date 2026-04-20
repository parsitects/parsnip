# Parsnip

## \*\*\* NOTICE \*\*\*
Parsnip is currently in a beta release state and is intended for early adopters. Users may experience issues, bugs and missing features. Those experiencing difficulties with Parsnip should create a GitHub issue.

## Overview
Parsnip assists in writing protocol parsers for the open-source network security monitoring tool [Zeek](https://github.com/zeek/zeek.git). It targets Industrial Control Systems (ICS) protocols but applies to any protocol.

The ecosystem has three components:
1. An Angular GUI that renders a protocol's packet structure and lets an analyst author it click-by-click.
2. An intermediate language (IL) — a set of JSON files with typed keywords — that the GUI produces and the compiler consumes.
3. A Flask/SQLAlchemy backend that stores IL entities in Postgres and emits a zipped parser package (Spicy, Zeek, and event files).

## Project structure

The superproject tracks three git submodules plus shared top-level orchestration:

| path | purpose |
| --- | --- |
| `parsnip-backend/` | Flask/SQLAlchemy API. Stores IL entities, emits IL bundles and compiled parser packages. Branch `v2_integration`. |
| `parsnip-frontend/` | Angular GUI for authoring IL. Branch `v2_integration`. |
| `parsnip-compiler/` | Python CLI that turns an IL bundle into generated parser sources. Branch `compiler-validation-invariants`. |
| `docker-compose.yml` | Wires db + backend + frontend into one stack; compiler is a profiled one-shot service. |
| `tooling/` | Cross-boundary helpers that span frontend + backend + canonical baseline. See `tooling/README.md`. |
| `compiler-input/`, `compiler-output/` | Volume mount points used when running the compiler service from the repo root. |
| `LICENSE.txt`, `NOTICE.txt` | License and notice. |

Each submodule has its own README with module-specific setup.

## Prerequisites

Clone with submodules, or initialize them after cloning:

```bash
git clone --recurse-submodules <url>
# or, in an existing clone:
git submodule update --init --recursive
```

The root `docker compose` commands build from the submodule trees and have nothing to build without them.

## Docker workflows

Root-level workflows build and run all three images from this repo:

- UI + API stack: `docker compose up --build`
- Include the compiler service profile: `docker compose --profile tools up --build`
- Run the compiler on demand against `compiler-input/` and `compiler-output/`: `docker compose run --rm compiler /input /output`

Standalone submodule workflows still work per-repo:

- Frontend: `cd parsnip-frontend && docker compose up --build`
- Backend: `cd parsnip-backend && docker compose up --build`
- Compiler: `cd parsnip-compiler && docker compose run --rm compiler /input /output`

### Services and ports

| service | container | port | notes |
| --- | --- | --- | --- |
| `db` | `parsnip-db` | `5432` | postgres:16, named volume `postgres_data`. |
| `backend` | `parsnip-backend` | `5000` | Flask API. Depends on `db` healthcheck. |
| `frontend` | `parsnip-frontend` | `4200 → 80` | Angular bundle served by nginx; `/api` is rewritten to the backend. |
| `compiler` | `parsnip-compiler` | — | CLI-only, under profile `tools`. Mounts `./compiler-input` and `./compiler-output`. |

All services share the `parsnip-network` bridge.

### Rebuilds

The backend and frontend bake source at build time (`COPY . .`). `docker compose restart` runs stale code; `docker compose up -d` without `--build` can reuse cached layers. When a change needs to land, rebuild and verify the served artifact:

```bash
docker compose build --no-cache frontend && docker compose up -d --no-deps frontend
```

## Cross-boundary tooling

Shared helpers that reach across submodule boundaries live under `tooling/`:

- `tooling/il_parity_check.py` — semantic parity gate. Calls `GET /parsers/{name}/generate_il` on a running backend, extracts the zip, and diffs it against the canonical baseline at `../icsnpp-hart-ip/parsnip_files/intermediate_language/` via `parsnip-backend/tooling/compare_il.py`. Dual-mode (CLI or pytest). See `tooling/README.md` for env-var overrides.

Per-submodule tooling (backend emission utilities, frontend build helpers, compiler runners) lives inside each submodule's own `tooling/` directory.

## Parity oracle

The v2 parity bar is **seed-path == UI-path**, both parsnip-v2 output. A parser authored entirely through the UI must produce byte-identical IL, compiled output, and runtime logs to the same parser authored by `parsnip-backend/tooling/seed_from_protocol.py`.

The shipped `icsnpp-hart-ip/{analyzer,scripts}/` tree is hand-polished post-processing on top of raw compiler output and is **not** the byte-level parity target. Its IL at `icsnpp-hart-ip/parsnip_files/intermediate_language/` is valid as a semantic reference for per-entity checks and is what `tooling/il_parity_check.py` consumes.

## AI-only scratch

`.docs/` and `.tmp/` are gitignored scratch areas used by Claude/Codex agents for phase directives, VERDICT write-ups, and ephemeral helper scripts. They do not persist past the current effort. Treat anything in them as load-bearing only for the active phase.

## Known limitations

* Package creation may have file-permission issues. If a package does not install, check that files under `testing/scripts` are executable.
* Choice actions can currently only point to objects, not other types such as integers.
* Frontend coverage is still catching up to the IL: AND/OR conditionals, layer-2 parsing, and the `minus` keyword are partial.
* Self-recursive types with multiple switches are not fully handled.
* Zeek btests in generated packages cover availability only.
