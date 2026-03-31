# Claude Code CLI (Source Overview)

This directory contains the core source for the **Claude Code** terminal application (`claude`) and related runtimes (interactive TUI, headless print mode, MCP server mode, remote/bridge flows, and SDK-oriented streaming I/O).

## What this project does

At a high level, this codebase implements an AI coding assistant CLI with:

- Interactive terminal UI (Ink/React based)
- Non-interactive/headless mode (`-p` / `--print`) for automation and pipes
- A command system (`/help`, `/review`, `/mcp`, `/plugin`, etc.)
- A tool execution system (shell, file read/edit/write, web fetch/search, MCP tools, task tools, etc.)
- Plugin and skill loading (bundled, local, and plugin-provided)
- MCP client + MCP server support
- Remote-control and bridge/session workflows
- Permission and policy enforcement for tool execution

## Important note about this folder

This `src/` directory does **not** currently include top-level build metadata (`package.json`, lockfile, tsconfig, etc.), so it should be treated as a **source snapshot/subtree** rather than a standalone build root.

If you want to run or build the project, use the full repository root that contains the package manager and build config files.

## Entry points

- `entrypoints/cli.tsx`: bootstraps CLI fast paths and launches main runtime
- `main.tsx`: command-line parsing, session setup, command dispatch, runtime orchestration
- `entrypoints/init.ts`: startup initialization (config, env, telemetry, policy/settings load, cleanup registration)
- `entrypoints/mcp.ts`: starts the MCP server implementation
- `entrypoints/sdk/*`: SDK schemas/types used by SDK consumers/builders

## High-level architecture

- `commands/`: slash/local commands and command handlers
- `tools/`: model-invocable tools and tool-specific logic/UI/prompts
- `services/`: integrations and subsystems (API, MCP, plugins, analytics, limits, tips, etc.)
- `bridge/`, `remote/`, `server/`: remote control/session bridge and server/session plumbing
- `components/`, `ink/`, `hooks/`, `screens/`: terminal UI and interaction layer
- `skills/`: bundled and loaded skill logic
- `tasks/`: background/agent task abstractions
- `state/`, `context/`: app/session state containers
- `utils/`: shared infra (config, permissions, auth, telemetry, git, shell, session storage, etc.)

## CLI behavior highlights

The default command starts an interactive session:

```bash
claude
```

Headless mode is supported:

```bash
claude -p "Summarize the current repository"
claude -p --output-format json "Generate release notes"
claude -p --output-format stream-json
```

From the command definitions in `main.tsx`, notable capabilities include:

- Auth management (`auth login|status|logout`)
- MCP management (`mcp serve|list|get|add-json|remove|...`)
- Plugin/marketplace management (`plugin ...`, `plugin marketplace ...`)
- Session lifecycle (`--continue`, `--resume`, `--session-id`, `--name`)
- Tool control (`--tools`, `--allowed-tools`, `--disallowed-tools`)
- Permission controls (`--permission-mode`, `--dangerously-skip-permissions`)
- Remote/bridge flows (`remote-control`, `ssh`, server/open commands)

## Commands and tools model

- Built-in commands are assembled in `commands.ts` and filtered by availability/auth/context.
- Commands can come from multiple sources: built-in, skills, plugins, workflows, bundled skills, and MCP.
- Tools are registered in `tools.ts`, feature-gated, and filtered by permission rules before exposure.
- Tool use is mediated by validation + permission context, with policy-aware filtering.

## Security and trust model

The runtime includes multiple protective layers:

- Workspace trust gating for operations that should not run in untrusted directories
- Permission modes and explicit tool permission prompts
- Allow/deny tool lists from CLI/config
- Enterprise policy filtering (including MCP server policy checks)
- Safe/unsafe config-env application split during startup

## Development notes

- Build/runtime logic uses Bun-specific features (`bun:bundle` feature gating and macros).
- Many optional capabilities are controlled by feature flags and environment toggles.
- Code is organized so expensive or optional modules are loaded lazily to optimize startup.

## Recommended next step

If your goal is to run or modify this project end-to-end, open the full repo root (with `package.json`) and add a companion root README section linking to this source architecture guide.
