# Parsnip

## \*\*\* NOTICE \*\*\*
Parsnip is currently in a beta release state and is intended for early adopters. Users may experience issues, bugs and missing features. Those experiencing difficulties with Parsnip should create a GitHub issue.

## Overview
Parsnip is a program developed to assist in the parsing of protocols using the open source network security monitoring tool [Zeek](https://github.com/zeek/zeek.git). Parsnip is specifically designed to be applied towards developing Industrial Control Systems (ICS) protocol parsers but can be applied to any protocol.

The Parsnip ecosystem consists of three parts:
1. A GUI interface designed provide a visual representation of a protocol's packet structure
2. JSON files in an intermediate language (IL) that is fed into the backend. This intermediate language is made up of a set of JSON structures using keywords for each key-value pair to indicate it's type. 
3. A backend that performs processing on the parsnip IL files and outputs the spicy, zeek and event files necessary for a parser

## Project Structure

Parsnip is organized as a superproject with three git submodules:

* `parsnip-backend/`: Flask/SQLAlchemy API that processes Parsnip IL into Spicy/Zeek parser packages
* `parsnip-frontend/`: Angular GUI for authoring Parsnip IL
* `parsnip-compiler/`: Python CLI that compiles IL bundles into generated parser sources
* `LICENSE.txt`: code license file
* `NOTICE.txt`: code notice file
* `README.md`: this file

Each submodule has its own README with module-specific details.

## Prerequisites

Clone with submodules, or initialize them after cloning:

```bash
git clone --recurse-submodules <url>
# or, in an existing clone:
git submodule update --init --recursive
```

All Docker workflows below assume the submodules are checked out. Without them, the root `docker compose` commands have nothing to build.

## Docker Workflows

The repository supports both module-level and root-level Docker Compose workflows.

Standalone submodule workflows:
- Frontend: `cd parsnip-frontend && docker compose up --build`
- Backend: `cd parsnip-backend && docker compose up --build`
- Compiler: `cd parsnip-compiler && docker compose run --rm compiler /input /output`

Root-level workflows:
- UI/API stack: `docker compose up --build`
- Include compiler service profile: `docker compose --profile tools up --build`
- Run compiler on demand: `docker compose run --rm compiler /input /output`

## Known Limitations
* Package creation may have some file permission issues. If a package does not install, check that the files in the testing/scripts directory are executable.
* Choices actions can currently only point to objects, not other types such as integers.
* Frontend functionality is limited. Specifically AND and OR conditionals, layer 2 parsing and the minus keyword need to be implemented in the intermediate language.
* Self-recursive types with multiple switches are not properly handled. This will be fixed in a later release.
* Zeek btest in the created package is limited to a basic availability test. Functionality for expanding btests has not been added.
