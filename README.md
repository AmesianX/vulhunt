# VulHunt Community Edition

VulHunt is a vulnerability hunting framework developed by Binarly's Research
team. It is designed to help security researchers and practitioners identify
vulnerabilities in software binaries and UEFI firmware. VulHunt is built on top
of Binarly's Binary Analysis and Inspection System (BIAS), which provides a
powerful and flexible environment for analysing and understanding binaries.
VulHunt integrates with the capabilities of the Binarly Transparency Platform
(BTP) to enable large-scale vulnerability management, hunting, and triage
capabilities.

VulHunt Community Edition is a free and open-source version of the VulHunt
engine within the BTP, designed to facilitate community-developed rulepacks and
integrations.

## Building (with cargo-make)

### Prerequisites

```bash
cargo install cargo-make
```

### Building

```bash
cargo make --profile <development|release> build
```

With support for Binary Ninja:

```bash
cargo make --profile <development|release> build --features=bndb
```

### Installation

```bash
cargo make --profile <development|release> install
```

With support for Binary Ninja:

```bash
cargo make --profile <development|release> install --features=bndb
```

## Building (without cargo-make)

### Prerequisites

```bash
git submodule update --init
```

Install LuaJIT with requisite patches:

```bash
git clone https://github.com/LuaJIT/LuaJIT.git -b v2.1
cd LuaJIT
git apply /path/to/vulhunt-ce/patches/luajit-vulhunt.patch
```

For macOS:

```bash
export MACOSX_DEPLOYMENT_TARGET=$(sw_vers -productVersion)
```

For macOS and Linux:

```bash
make BUILDMODE='static'
export LUA_LIB=/path/to/LuaJIT/src/
export LUA_LIB_NAME=luajit
export LUA_LINK=static
```

For Windows:

```bash
cd src
msvcbuild.bat BUILDMODE='static'
set LUA_LIB=C:\path\to\LuaJIT\src\
set LUA_LIB_NAME=lua51
set LUA_LINK=static
```

### Building

```bash
cargo build --release
```

With support for Binary Ninja:

```bash
cargo build --release --features=bndb
```

### Packaging

Prerequisites:

```bash
cargo install cargo-make
```

Build packages for the current platform:

```bash
cargo make prepare-package --features=...
```

## Usage

### Scanning binaries

```bash
vulhunt-ce scan <INPUT> -o <OUTPUT> -d <BIAS_DATA> -r <RULES> [OPTIONS]
```

Options:

- `<INPUT>`: Path to the binary, BA2 archive, or BNDB file to scan
- `-o, --output <OUTPUT>`: Path to write output JSON
- `-d, --data <BIAS_DATA>`: Directory containing auxiliary data (processor specifications, etc.). Can also be set via `BIAS_DATA` environment variable
- `-r, --rules <RULES>`: Directory containing VulHunt rules. Can also be set via `BIAS_VULHUNT_RULES` environment variable
- `-m, --modules <MODULES>`: Directory containing VulHunt modules (optional). Can also be set via `BIAS_VULHUNT_MODULES` environment variable
- `--loader <LOADER>`: Configure the loader to use (default: `component`). Available loaders:
  - `component`: Scan single binary files
  - `ba2`: Scan BA2 (Binarly Archive 2) archives containing multiple components
  - `bndb`: Scan Binary Ninja databases (requires `--features=bndb` at build time)
- `--pretty`: Format output for human consumption and render issues to stdout
- `--stream`: Format output as a stream of JSONL messages
- `--compress`: Compress output JSONL stream with Zstandard

Example:

```bash
vulhunt-ce scan lib.so -o results.json -d /path/to/bias-data -r /path/to/rules --pretty
vulhunt-ce scan firmware.ba2 --loader ba2 -o results.json -d /path/to/bias-data -r /path/to/rules --pretty
vulhunt-ce scan project.bndb --loader bndb -o results.json -d /path/to/bias-data -r /path/to/rules --pretty
```

### Starting the MCP server

VulHunt can run as an MCP (Model Context Protocol) server for integration with AI assistants. By default, it starts a streaming HTTP server with SSE (Server-Sent Events) transport at `http://127.0.0.1:8080`:
```bash
vulhunt-ce mcp -d <BIAS_DATA> [OPTIONS]
```

Options:

- `-d, --data <BIAS_DATA>`: Directory containing auxiliary data (required). Can also be set via `BIAS_DATA` environment variable
- `-m, --modules <MODULES>`: Directory containing VulHunt modules (optional). Can also be set via `BIAS_VULHUNT_MODULES` environment variable
- `--stdio`: Use stdio transport instead of HTTP
- `--host <HOST>`: Host address to bind (default: `127.0.0.1`)
- `--port <PORT>`: Port to listen on (default: `8080`)

### BA2 archive utilities

List components in a BA2 archive:

```bash
vulhunt-ce ba2 list-components <INPUT>
```

Extract a component from a BA2 archive:

```bash
vulhunt-ce ba2 extract-component <INPUT> -o <OUTPUT> --component-id <UUID>
```

Options:

- `<INPUT>`: Path to the BA2 archive
- `-o, --output <OUTPUT>`: Output path for the extracted component
- `--component-id <UUID>`: UUID of the component to extract

### BTP integration

Interact with the Binarly Transparency Platform (BTP). All commands require authentication:

Common options:

- `-u, --username <USERNAME>`: BTP username (or `BTP_USERNAME` env var)
- `-p, --password <PASSWORD>`: BTP password (or `BTP_PASSWORD` env var)
- `-s, --instance-slug <SLUG>`: Instance slug, e.g., `your-org.prod` (or `BTP_INSTANCE_SLUG` env var)

Available commands:

```bash
vulhunt-ce btp push-rules <INPUTS> -r <REPOSITORY> [-t <TAG>] [--name <NAME>] [--platform <posix|uefi>] [--modules <DIR>] [--deploy-to-product <ULID> | --deploy-to-org <ULID>]
vulhunt-ce btp list-products
vulhunt-ce btp create-product --name <NAME> [--description]
vulhunt-ce btp upload <FILE> --product-id <ULID> --name <NAME> --version <VERSION> [--scan]
vulhunt-ce btp list-images --product-id <ULID>
vulhunt-ce btp list-scans --product-id <ULID> --image-id <ULID>
vulhunt-ce btp create-scan --product-id <ULID> --image-id <ULID>
vulhunt-ce btp get-scan --product-id <ULID> --image-id <ULID> --scan-id <ULID>
vulhunt-ce btp get-findings --product-id <ULID> --image-id <ULID>
vulhunt-ce btp download-ba2 --product-id <ULID> --image-id <ULID> [--scan-id <ULID>] [-o <OUTPUT>]
```

## License

This project is licensed under the GNU General Public License v3.0 - see the [LICENSE](LICENSE) file for details.

Copyright (c) 2026 Binarly Inc. and VulHunt developers.
