# Parsnip

## \*\*\* NOTICE \*\*\*
Parsnip is currently in a beta release state and is intended for early adopters. Users may experience issues, bugs and missing features. Those experiencing difficulties with Parsnip should create a GitHub issue.

## Overview
<<<<<<< HEAD
Parsnip assists in writing protocol parsers for the open-source network security monitoring tool [Zeek](https://github.com/zeek/zeek.git). It was designed for Industrial Control Systems (ICS) protocols but applies to any protocol.

The ecosystem has four components:
1. An Angular GUI that renders a protocol's packet structure and lets an analyst author it click-by-click.
2. An intermediate language (IL) — a set of JSON files with typed keywords — that the GUI produces and the compiler consumes.
3. A Flask/SQLAlchemy backend that stores IL entities in Postgres and emits a zipped parser package (Spicy, Zeek, and event files).
4. A compiler leveraged by the backend to produce the Zeek/Spicy parser
=======
Parsnip is a program developed to assist in the parsing of protocols using the open source network security monitoring tool [Zeek](https://github.com/zeek/zeek.git). Parsnip is specifically designed to be applied towards developing Industrial Control Systems (ICS) protocol parsers but can be applied to any protocol.

The Parsnip ecosystem consists of three parts:
1. A GUI interface designed provide a visual representation of a protocol's packet structure
2. JSON files in an intermediate language (IL) that is fed into the backend. This intermediate language is made up of a set of JSON structures using keywords for each key-value pair to indicate it's type. 
3. A backend that performs processing on the parsnip IL files and outputs the spicy, zeek and event files necessary for a parser
>>>>>>> main

## Project structure

<<<<<<< HEAD
The superproject tracks three git submodules plus shared top-level orchestration:
=======
* backend: folder containing the code for the backend; used for processing parsnip IL files to create zeek parsers
* docs: folder containing project documentation
* frontend: folder containing the code for the web frontend; used for creating parsnip IL files
* LICENSE.txt: code license file
* NOTICE.txt: code notice file
* README.md: this file
>>>>>>> main

| path | purpose |
| --- | --- |
| `parsnip-backend/` | Flask/SQLAlchemy API. Stores IL entities, emits IL bundles, proxies compile requests. Branch `v2_integration`. |
| `parsnip-frontend/` | Angular GUI for authoring IL. Branch `v2_integration`. |
| `parsnip-compiler/` | Python package that turns an IL bundle into a Zeek/Spicy parser package. Ships a CLI (`main.py`) and an HTTP microservice (`compiler_service.py`) that also runs `spicyz`. Branch `compiler-validation-invariants`. |
| `docker-compose.yml` | Wires db + backend + frontend + long-running compiler microservice into one stack. |
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

Root-level workflows build and run all four images from this repo:

- Full stack (db + backend + frontend + compiler microservice): `docker compose up --build`
- If there are issues with frontend container, try running `docker compose down -v` before re-running the above command. Please not note that this may wipe all existing parsers.

The UI's **Compile** tab posts a `.pil` upload to `/api/parsers/compile`, which the backend proxies to the compiler microservice. The service runs the parsnip compiler and then `spicyz`, returning the generated source tree + a Zeek-loadable `.hlto` alongside the captured toolchain logs.

Standalone submodule workflows still work per-repo:

- Frontend: `cd parsnip-frontend && docker compose up --build`
- Backend: `cd parsnip-backend && docker compose up --build`
- Compiler CLI: `cd parsnip-compiler && docker compose run --rm compiler /input /output` (writes to `./input` → `./output` inside that repo).

### Services and ports

| service | container | port | notes |
| --- | --- | --- | --- |
| `db` | `parsnip-db` | `5432` | postgres:16, named volume `postgres_data`. |
| `backend` | `parsnip-backend` | `5000` | Flask API. Depends on `db` healthcheck. |
| `frontend` | `parsnip-frontend` | `4200 → 80` | Angular bundle served by nginx; `/api` is rewritten to the backend. |
| `compiler-service` | `parsnip-compiler-service` | `5001` | HTTP microservice wrapping the compiler + `spicyz`. Image is based on `zeek/zeek:8.0.7`. |

All services share the `parsnip-network` bridge.

### Rebuilds

The backend and frontend bake source at build time (`COPY . .`). `docker compose restart` runs stale code; `docker compose up -d` without `--build` can reuse cached layers. When a change needs to be implemented, rebuild and verify the served artifact:

## Known limitations

* Package creation may have file-permission issues. If a package does not install, check that files under `testing/scripts` are executable.
* Frontend coverage is still catching up to the IL: the `minus` keyword is partially supported.
* Self-recursive types with multiple switches are not fully handled.
* Zeek btests in generated packages cover availability only.
