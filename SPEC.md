# pi-mode — Behavioral Launcher for pi

A CLI launcher for [pi](https://github.com/badlogic/pi-mono) that layers behavioral system prompt instructions on top of pi's defaults, using the same axis model as [claude-code-modes](https://github.com/nklisch/claude-code-modes).

## Architecture

```
pi-mode (bash script)
  ├─ Parses CLI args (preset, axis overrides, modifiers)
  ├─ Resolves preset → axis values + modifiers
  ├─ Reads matching .md fragments from ~/.pi/agent/pi-mode/
  ├─ Concatenates them into an --append-system-prompt string
  └─ exec's pi --append-system-prompt "$ASSEMBLED" -- "$@"
```

Pi's default system prompt remains intact. Behavioral instructions are appended at the end where recency bias gives them priority.

## CLI Interface

```
pi-mode <preset|none> [axis-overrides] [modifiers] [-- pi-flags]
pi-mode --agency <val> --quality <val> --scope <val> [modifiers] [-- pi-flags]
pi-mode --help
```

### Presets

| Preset | Agency | Quality | Scope | Modifiers |
|--------|--------|---------|-------|-----------|
| `create` | autonomous | architect | unrestricted | — |
| `extend` | autonomous | pragmatic | adjacent | — |
| `safe` | collaborative | minimal | narrow | — |
| `refactor` | autonomous | pragmatic | unrestricted | — |
| `explore` | collaborative | architect | narrow | readonly |
| `debug` | collaborative | pragmatic | narrow | — |
| `methodical` | surgical | architect | narrow | — |
| `none` | — | — | — | — |

### Axis Overrides

```
pi-mode create --quality pragmatic
pi-mode safe --scope adjacent
pi-mode --agency autonomous --quality architect --scope narrow
```

Flags:
- `--agency <autonomous|collaborative|surgical>`
- `--quality <architect|pragmatic|minimal>`
- `--scope <unrestricted|adjacent|narrow>`

Standalone axis composition (no preset): unspecified axes default to `collaborative`/`pragmatic`/`adjacent`.

### Modifiers

```
pi-mode create --readonly
pi-mode create --context-pacing
pi-mode create --bold
pi-mode create --tdd
pi-mode create --modifier custom-name
pi-mode create --modifier-file /path/to/custom.md
```

| Flag | Effect |
|------|--------|
| `--readonly` | No file modifications; explain what you WOULD do |
| `--context-pacing` | Pause at natural boundaries; document remaining work |
| `--bold` | Confident, idiomatic code; no hedging |
| `--tdd` | Test-driven development workflow |
| `--modifier <name>` | Load `~/.pi/agent/pi-mode/modifiers/<name>.md` |
| `--modifier-file <path>` | Load modifier from arbitrary file path |

### Pi Passthrough

Everything after `--` is forwarded to pi verbatim:

```
pi-mode create -- --model sonnet --thinking high
```

Unrecognized flags are also forwarded to pi (e.g., `pi-mode create --verbose` passes `--verbose` to pi).

### Help

```
pi-mode --help        # Show usage with preset table and examples
pi-mode --version     # Show version
```

## File Structure

```
~/.pi/agent/pi-mode/          # Data directory
├── agency/
│   ├── autonomous.md
│   ├── collaborative.md
│   └── surgical.md
├── quality/
│   ├── architect.md
│   ├── pragmatic.md
│   └── minimal.md
└── modifiers/
│   ├── autonomous.md      # (alias/symlink to modifiers dir)
│   ├── adjacent.md
│   └── narrow.md
└── modifiers/
    ├── readonly.md
    ├── context-pacing.md
    ├── bold.md
    └── tdd.md

/usr/local/bin/pi-mode         # The launcher script (symlink or copy)
```

### Fragment Format

Each `.md` file is a standalone markdown section. They are adapted from claude-code-modes, with Claude-specific references removed and pi tool names used where applicable.

## Prompt Assembly

Fragments are concatenated in this order with headers:

```
## Behavioral Instructions

[agency fragment]
[quality fragment]
[scope fragment]
[modifier fragment 1]
[modifier fragment 2]
...

These behavioral instructions take priority over default guidance.
```

The result is passed as a single string to `pi --append-system-prompt`.

## Edge Cases & Error Handling

- **Missing fragment file**: Print error to stderr, exit non-zero
- **Invalid preset name**: Print "Unknown preset: X. Available: ..." to stderr, exit non-zero
- **Invalid axis value**: Print "Invalid --agency value: X. Must be: autonomous, collaborative, surgical" and exit non-zero
- **`none` preset with axis flags**: Ignored — `none` strips all behavioral instructions
- **No args**: Same as `pi-mode safe` (safe default) or print usage? → Print usage
- **No pi on PATH**: Let pi fail naturally
- **Empty modifier file**: Included silently (no effect)
- **Duplicate modifiers**: Included twice (user's responsibility to not duplicate)

## What's NOT in v1

- director, partner, muse presets (pi doesn't have sub-agents; muse/bold can be added via modifiers)
- Config file (`.pi-mode.json`)
- Custom bases (standard vs chill)
- Environment detection baked into prompt
- Temp file management (uses `--append-system-prompt` string)
- Auto-update checking
- Install script (manual copy instructions)

## Dependencies

- **bash** (POSIX shell compatible)
- **pi** on PATH
- Fragment files in `~/.pi/agent/pi-mode/`

## Install

```bash
# Copy launcher to PATH
cp pi-mode /usr/local/bin/pi-mode
chmod +x /usr/local/bin/pi-mode

# Copy prompts to data directory
mkdir -p ~/.pi/agent/pi-mode/
cp -r prompts/* ~/.pi/agent/pi-mode/
```