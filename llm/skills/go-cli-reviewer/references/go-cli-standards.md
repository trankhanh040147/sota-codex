# Go CLI Coding Standards

Agent-optimized rules for coding in Go CLI projects. Read this file when reviewing code to understand project-specific patterns and conventions.

## Core Stack

- **Go 1.25+** | **Cobra** | **Charm** (Bubble Tea, Lip Gloss, Bubbles, Log) | **SQLite** | **sonic** (JSON) | **samber/lo** (slices) | **fantasy** (LLM)
- Directory: `internal/` (agent/, tui/, tools/, session/, config/, tool_docs/); `cmd/` thin entry point only
- Config: `~/.config/prepf/config.yaml`, env vars as constants in `internal/config/constants.go`, inject via context (never global getters)

## Critical Bans (NEVER Do These)

| Ban | Why |
|-----|-----|
| `os.Exit()` in library code | Return errors from Cobra RunE instead |
| `fmt.Scanln`, `fmt.Sscanf` | Use `bufio.Reader` + `strconv` |
| `encoding/json` | Use `bytedance/sonic` only |
| Manual loops for transform | Use `lo.Map()`, `lo.Filter()`, `lo.Find()` |
| `strings.Repeat()` for padding | Use `lipgloss.Place()` for alignment |
| Hardcoded config keys | Use constants in `internal/config/constants.go` |
| `os.Getenv()` in display logic | Use single `cfg.*` struct (source of truth) |
| LLM calls from TUI | Emit `tea.Cmd` to trigger Agent instead |
| Blocking I/O in TUI | Wrap in `tea.Cmd` or channels |
| Global flag getters | Inject config via context |
| Comments for user comms | Only add comments if user explicitly asks |

## Code Organization

### File & Function Size
- **CRITICAL**: Files <300 lines, functions <25 lines. Split aggressively.
- Reduce token overhead + improve code review
- Extract after 2x repetition (DRY enforcement)

### Error Handling
- Wrap all: `fmt.Errorf("context: %w", err)`
- Log before fallback
- Return from Cobra `RunE` (never `os.Exit()`)

### Input Safety
- Bounds check: `0 ≤ idx < len()` before access. Reset to -1 on empty.
- Single `bufio.Reader` per source. Parse with `strconv`.

### Concurrency
- **MUST use** `errgroup.Group` (propagates errors); pass `context.Context` through stack
- Background I/O: Return `tea.Cmd` spawning goroutine; handle results via messages
- Channels + `tea.Batch` for concurrent updates

### Data & Mutation
- Pre-allocate slices; never address map index directly
- Safe edit: Cache IDs **before** editing, execute against cached values

### Constructors
- Return pointers: `func NewType(...) *Type`

## TUI (Bubble Tea)

### State Management
- Explicit states: `Idle`, `Thinking`, `ExecutingTool`, `StreamingResponse`, `Error`
- Type-based switching in `Update()` to dispatch handlers
- **Never** call LLM directly from TUI; emit `tea.Cmd`

### Streaming & Rendering
- Don't wait for full response. Render on every token.
- Channel pattern: LLM → channel → TUI (concurrent)
- Batch tokens until punctuation/newline (reduce redraws)
- Show spinner while thinking

### Input
- Centralized `KeyMap` struct. Use `key.Matches()` always (never `msg.String()`)
- Vim keys: `j/k`, `Enter`/`Tab` select, `?` help, `Esc` back

### Styling
- Centralized `internal/tui/styles/` package
- Use `lipgloss.Place()` for alignment, disable colors for `NO_COLOR`/non-TTY
- Width safety: Guard `strings.Repeat()` with `max(0, count)`; default to 80 if `Width() == 0`

### Component Reuse
- Prefer `bubbles` (Editor, List, Viewport, Spinner). Wrap, don't rewrite.
- Decompose `Update()/View()` >20 complexity into helpers

## Configuration

### Viper Setup
```go
// ONLY in Load()
setupViper()
viper.ReadInConfig()

// Save() MUST NOT call setupViper() or ReadInConfig()
// Use viper.ConfigFileUsed() or default to Config.ConfigDir
```

### Flag Parsing
- Unique short aliases. Parse in root `PersistentPreRunE`.
- Inject via context: `GetConfig(cmd)` (never global)
- YAML errors: Silent on file-not-found; warn to stderr on parse errors

## Tools (Stateless Capabilities)

- Each tool = single, pure function: params → result string or error
- **Contract**: Stateless, one thing, fail fast (LLM decides recovery)
- **Safety**: Permission checks, file path validation, command whitelisting
- **Documentation**: `internal/tool_docs/{name}.md` with schema, output format, constraints, examples
- Output: Structural data (paths, results) as raw lists; verbose (logs) truncated/summarized

## LLM & Context

- System prompts: Markdown in `internal/agent/templates/`, embedded with `//go:embed`
- Token budget: Track usage; >80% threshold → auto-summarize or rolling window (keep system + last N + file refs)
- RAG: Never dump entire codebase. Let LLM request files explicitly via tool calls.
- Streaming: Channel pattern, batch before rendering, user cancellation via `context.Context`

## Testing

- **Mock providers** for testing UI without API calls (use interfaces)
- **Golden files** (`.golden`) for regression testing
- **Deterministic**: Same inputs → same outputs
- **Debug logs** to file (`debug.log`), not stdout

## Workflow

### Before Acting
1. Search codebase for relevant files (grep, glob, agent)
2. Read entire files before editing (note exact whitespace)
3. Check memory for stored commands

### While Acting
1. **Read before edit**: Verify exact indentation, whitespace
2. **Use exact text**: Find/replace match character-for-character (spaces, tabs, newlines)
3. One logical change per edit. Test after each.
4. Fix immediately if tests fail. Keep going until task fully resolved.

### Before Finishing
1. Verify entire query resolved (re-read original)
2. All described next steps completed
3. Run full test suite; lint if available
4. Response <4 lines (concise)

## Git & Communication

- **Commit only when explicitly asked**. Never push to remote unless asked.
- Message: Concise (1-2 sentences), focus on **why** not **what**
- Comments: **Only if user explicitly requests**. Focus on **why**, never for user comms.
- Go style: Informal, concise, idiomatic. No fluff; use bullet points.

## Bug Fix Protocol

1. Global search: Find all similar patterns in codebase
2. Fix all occurrences (root cause everywhere, not one patch)
3. Test all modes: interactive, piped (`|`), non-interactive

## Security & Decision-Making

- **Only assist with defensive security.** Refuse malicious code.
- Never log secrets. Validate all external input.
- **Be autonomous**: Search, read, infer, decide, act. Don't ask; exhaust alternatives.

### Stop & Ask User Only If:
- Truly ambiguous business requirement (not inferable)
- Multiple valid approaches with major tradeoffs
- Could cause data loss
- Hit hard external limit (missing credentials, permissions)

### Continue If:
- Task seems large → break down & complete
- Multiple files → change them
- Many steps → do all steps

## Common Patterns

### Safe Array Access
```go
// BAD
item := items[selectedIdx]

// GOOD
if selectedIdx >= 0 && selectedIdx < len(items) {
    item := items[selectedIdx]
}
```

### Error Wrapping
```go
// BAD
return err

// GOOD
return fmt.Errorf("failed to load config: %w", err)
```

### Config Injection
```go
// BAD
apiKey := os.Getenv("API_KEY")

// GOOD
type Config struct { APIKey string }
func GetConfig(cmd *cobra.Command) *Config {
    return cmd.Context().Value(configKey).(*Config)
}
```

### TUI Command Pattern
```go
// BAD - blocks TUI
func (m Model) Update(msg tea.Msg) (tea.Model, tea.Cmd) {
    response := callLLM() // WRONG!
    return m, nil
}

// GOOD - non-blocking
func (m Model) Update(msg tea.Msg) (tea.Model, tea.Cmd) {
    return m, func() tea.Msg {
        response := callLLM()
        return ResponseMsg{response}
    }
}
```

### Transform with lo
```go
// BAD
var names []string
for _, user := range users {
    names = append(names, user.Name)
}

// GOOD
names := lo.Map(users, func(u User, _ int) string { return u.Name })
```

### Viper Config
```go
// BAD - in Save()
func Save() error {
    setupViper()        // WRONG!
    viper.ReadInConfig() // WRONG!
    return viper.WriteConfig()
}

// GOOD - only in Load()
func Load() error {
    setupViper()
    return viper.ReadInConfig()
}

func Save() error {
    return viper.WriteConfigAs(viper.ConfigFileUsed())
}
```
