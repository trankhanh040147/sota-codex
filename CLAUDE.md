# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Purpose

**sota-codex** is a collection of binding instructions, architectural blueprints, and strict rule sets designed to align AI output with modern 2026+ engineering standards. It serves as a "Source of Truth" for forcing AI agents out of median/legacy patterns.

## Repository Structure

```
sota-codex/
├── stacks/                    # Full-stack architectural blueprints
│   ├── astro-hybrid/          # Astro 5 + React 19 guidelines (AGENTS.md)
│   └── go-cli/crush/          # Go CLI reference implementation
```

## Stack-Specific Guidelines

### Astro Hybrid Stack (`stacks/astro-hybrid/`)

See `stacks/astro-hybrid/AGENTS.md` for the complete ruleset. Key points:

- **Interactivity Tiers**: Tier 1 (Static .astro), Tier 1.5 (Server Islands with `server:defer`), Tier 2 (Vanilla TS <50 lines), Tier 3 (React Islands for complex state)
- **State**: Use Nanostores for cross-island state, not React Context
- **Mutations**: Use Astro Actions (`src/actions/`), not manual API endpoints
- **View Transitions**: Use `<ClientRouter />` (Astro 5), handle `astro:page-load` for script reinitialization
- **Environment**: Use `astro:env` for type-safe env vars, not `process.env` or `import.meta.env`

### Go CLI Stack (`stacks/go-cli/crush/`)

Reference implementation with `Makefile` and `Taskfile.yaml` for commands.

**Build/Test Commands** (from `stacks/go-cli/crush/`):
```bash
make build           # Build binary
make test            # Run tests
make lint            # Run linters (golangci-lint)
make fmt             # Format code (gofumpt)
task dev             # Run with profiling (CRUSH_PROFILE=true)
task lint:fix        # Lint and auto-fix

# Single test
go test ./internal/llm/prompt -run TestGetContextFromPaths

# Update golden files
go test ./... -update
```

**Code Style**:
- Use `gofumpt` for formatting (stricter than gofmt)
- Group imports: stdlib, external, internal
- Use `fmt.Errorf` for error wrapping, always pass `context.Context` first
- Use `testify/require`, `t.Parallel()`, `t.TempDir()` for tests
- JSON tags: snake_case, file permissions: octal (0o755)
- Comments on own lines: capital letter, end with period, wrap at 78 columns

**TUI Stack** (Charm libraries):
- `bubbletea` (Elm architecture), `lipgloss` (styling), `bubbles` (components), `huh` (forms)
- Use `lipgloss.Place()` for alignment, never `strings.Repeat` for padding
- Use `key.Matches()` with KeyMap structs, never `msg.String()`
- Wrap file I/O in `tea.Cmd` to prevent UI blocking

**Key Principles**:
- Use `samber/lo` for slice/map operations, avoid raw `for` loops for transformations
- Use `bytedance/sonic` for JSON, not `encoding/json`
- Use `errgroup.Group` over `sync.WaitGroup`
- Config in YAML at `~/.config/prepf/config.yaml`, load in `rootCmd.PersistentPreRunE`
- Return errors to main via `Cobra.RunE`, no `os.Exit` in library code

**Commits**: Use semantic commits (`fix:`, `feat:`, `chore:`, `refactor:`, `docs:`, `sec:`). Keep to one line unless additional context is truly necessary.
