# pi-modes

> **Adapted from [claude-code-modes](https://github.com/nklisch/claude-code-modes) by
> [Nathan Klisch](https://github.com/nklisch).** This is a rebuild for the
> [pi](https://github.com/badlogic/pi-mono) coding agent.

A CLI launcher for pi that layers behavioral system prompt instructions on top of
pi's defaults, using the same Agency/Quality/Scope axis model as claude-code-modes.

## How It Works

```
pi-mode (bash script)
  ├─ Parses CLI args (preset, axis overrides, modifiers)
  ├─ Resolves preset → axis values + modifiers
  ├─ Reads matching .md fragments from ~/.pi/agent/pi-mode/
  ├─ Concatenates them into an --append-system-prompt string
  └─ exec's pi --append-system-prompt "$ASSEMBLED" -- "$@"
```

Pi's default system prompt stays intact. Behavioral instructions are appended at
the end where recency bias gives them priority.

## Quick Start

```bash
# Copy launcher to PATH
cp pi-mode /usr/local/bin/pi-mode
chmod +x /usr/local/bin/pi-mode

# Copy prompts to data directory
mkdir -p ~/.pi/agent/pi-mode/
cp -r prompts/* ~/.pi/agent/pi-mode/
```

## Usage

Pick a preset that matches your task:

```bash
pi-mode create "Build a REST API with Go"
pi-mode extend "Add rate limiting to the existing server"
pi-mode safe "Fix the login redirect bug"
pi-mode refactor "Consolidate shared types into one package"
pi-mode explore "How does the auth middleware work?"
pi-mode debug "Investigate why payments are failing intermittently"
pi-mode methodical "Add input validation to all form handlers"
```

| Preset | Agency | Quality | Scope | Use |
|--------|--------|---------|-------|-----|
| `create` | autonomous | architect | unrestricted | Build from scratch |
| `extend` | autonomous | pragmatic | adjacent | Extend existing projects |
| `safe` | collaborative | minimal | narrow | Surgical production changes |
| `refactor` | autonomous | pragmatic | unrestricted | Restructure codebase |
| `explore` | collaborative | architect | narrow | Read-only exploration (auto `--readonly`) |
| `debug` | collaborative | pragmatic | narrow | Investigation-first debugging |
| `methodical` | surgical | architect | narrow | Step-by-step precision |
| `none` | — | — | — | No behavioral instructions |

### Axis Overrides

Override any axis of a preset:

```bash
pi-mode create --quality pragmatic
pi-mode safe --scope adjacent
```

Or compose them free-form (defaults are collaborative/pragmatic/adjacent):

```bash
pi-mode --agency autonomous --quality architect --scope narrow "Build a CLI tool"
```

Available values:
- `--agency`: `autonomous`, `collaborative`, `surgical`
- `--quality`: `architect`, `pragmatic`, `minimal`
- `--scope`: `unrestricted`, `adjacent`, `narrow`

### Modifiers

Layer additional behaviors on top of any preset:

```bash
pi-mode create --bold --tdd "Add user authentication"
pi-mode explore --context-pacing "Walk me through the routing layer"
pi-mode safe --readonly "Show me what you'd change for the caching fix"
```

| Flag | Effect |
|------|--------|
| `--readonly` | No file modifications; explain what you WOULD do |
| `--context-pacing` | Pause at natural boundaries; document remaining work |
| `--bold` | Confident, idiomatic code; no hedging |
| `--tdd` | Test-driven development workflow |
| `--modifier <name>` | Load from `~/.pi/agent/pi-mode/modifiers/<name>.md` |
| `--modifier-file <path>` | Load modifier from arbitrary file path |

### Pi Passthrough

Everything after `--` goes to pi:

```bash
pi-mode create -- --model sonnet --thinking high
pi-mode explore "How does X work?" -- --model opus
```

### Other Commands

```bash
pi-mode --help       # Full usage with preset table and examples
pi-mode --version    # Show version
pi-mode create --dry-run "..."  # Print assembled prompt without launching pi
```

## Environment

Default prompt directory: `~/.pi/agent/pi-mode/`. Override with `PI_MODE_DIR`:

```bash
export PI_MODE_DIR=/path/to/custom/prompts
```

## Architecture

See [SPEC.md](SPEC.md) for the full design specification, including edge case
handling, future directions (v2), and prompt assembly details.

## Dependencies

- **bash** (any POSIX-compatible shell)
- **[pi](https://github.com/badlogic/pi-mono)** on PATH

## License

MIT — see [LICENSE](LICENSE).

This project is an adaptation of [claude-code-modes](https://github.com/nklisch/claude-code-modes)
by Nathan Klisch, also MIT-licensed. The prompt fragments in `prompts/` are derived from
the original claude-code-modes system prompts, modified for pi's tool set and conventions.

## Related

- [claude-code-modes](https://github.com/nklisch/claude-code-modes) — the original project this adapts
- [pi](https://github.com/badlogic/pi-mono) — the pi coding agent
