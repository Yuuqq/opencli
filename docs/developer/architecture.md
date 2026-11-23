# Architecture

OpenCLI is built on a **Dual-Engine Architecture** that supports both declarative YAML pipelines and programmatic TypeScript adapters.

## High-Level Architecture

```
Adapter discovery --> Registry --> Commander.js
                                        |
                                  executeCommand
                                        |
                         YAML pipeline or TypeScript handler
                                        |
    Runtime access                      +-- No browser: runs locally
    (as required)                       +-- Browser Bridge (default)
                                        |     CLI --HTTP--> daemon --WebSocket--> extension
                                        +-- Direct CDP (OPENCLI_CDP_ENDPOINT set)
                                              CLI --> Chrome or Electron debugging target
```

Commander formats the result after the adapter returns. Registered external CLIs are passed through to their own binaries instead.

## Core Modules

### Registry (`src/registry.ts`)
Central command registry. All adapters register their commands via the `cli()` function with metadata: site, name, description, domain, strategy, args, columns.

### Discovery (`src/discovery.ts`)
CLI discovery and manifest loading. Discovers commands from YAML and TypeScript adapter files, parses YAML pipelines, and registers them into the central registry.

### Execution (`src/execution.ts`)
Command execution: argument validation, lazy loading of adapter modules, and executing the appropriate handler function.

### Commander Adapter (`src/commanderAdapter.ts`)
Bridges registry commands to Commander.js subcommands. It parses positional arguments and options, calls the execution layer, and formats results and errors.

### Browser (`src/runtime.ts`, `src/browser/`)
The execution layer checks whether a command needs a browser. For browser sessions, the runtime chooses the direct CDP bridge if `OPENCLI_CDP_ENDPOINT` is set, or the local [Browser Bridge](../guide/browser-bridge.md) otherwise.

### Pipeline (`src/pipeline/`)
The YAML pipeline engine. Processes declarative steps:
- **fetch** — HTTP requests with cookie/header strategies
- **map** — Data transformation with template expressions
- **limit** — Result truncation
- **filter** — Conditional filtering
- **download** — Media download support

### Output (`src/output.ts`)
Unified output formatting: `table`, `json`, `yaml`, `md`, `csv`.

## Authentication Strategies

OpenCLI recognizes five adapter strategies:

| Strategy | How It Works | When to Use |
|----------|-------------|-------------|
| `public` | Direct HTTP fetch, no auth | Public APIs (HackerNews, BBC) |
| `cookie` | Reuse Chrome cookies via Browser Bridge | Logged-in sites (Bilibili, Zhihu) |
| `header` | Custom auth headers | API-key based services |
| `intercept` | Network request interception | GraphQL/XHR capture (Twitter) |
| `ui` | DOM interaction via accessibility snapshot | Desktop apps, write operations |

## Directory Structure

```
src/
├── main.ts              # Entry point
├── cli.ts               # Commander.js CLI setup + built-in commands
├── commanderAdapter.ts  # Registry → Commander bridge
├── discovery.ts         # CLI discovery, manifest loading, YAML parsing
├── execution.ts         # Arg validation, command execution
├── registry.ts          # Command registry
├── serialization.ts     # Command serialization helpers
├── runtime.ts           # Browser session & timeout management
├── browser/             # Browser Bridge and direct CDP clients
├── daemon.ts            # Loopback HTTP and extension WebSocket server
├── output.ts            # Output formatting
├── doctor.ts            # Diagnostic tool
├── pipeline/            # YAML pipeline engine
│   ├── executor.ts
│   ├── template.ts
│   └── steps/
│       ├── fetch.ts
│       ├── browser.ts
│       ├── transform.ts # Includes map, limit, and filter
│       └── download.ts
└── clis/                # Site adapters
    ├── twitter/
    ├── reddit/
    ├── bilibili/
    ├── cursor/
    └── ...
```
