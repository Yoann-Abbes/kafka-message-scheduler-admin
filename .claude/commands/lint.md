---
description: Lint and format code with project linter
disable-model-invocation: true
effort: low
argument-hint: "[check|fix|format|file-path|changed]"
---

Lint and format code: $ARGUMENTS

## Instructions

Uses **golangci-lint** for linting/formatting. Config: `{{LINTER_CONFIG_FILE}}` at repo root.
Style: Project defaults

### Determine scope from arguments:

| Argument | Action |
|----------|--------|
| `check` or `all` | `golangci-lint run` (report only) |
| `fix` or `write` | `golangci-lint run --fix` (auto-fix) |
| `format` | `gofmt -w .` (format only) |
| `<file-path>` | `golangci-lint run --fix <file-path>` (single file) |
| `changed` | Lint only files changed vs main |

### Common workflows:

1. **Before commit**: `golangci-lint run --fix <changed-files>`
2. **CI equivalent**: `golangci-lint run`

### ⚠️ Pitfalls:
<!-- Add linter-specific pitfalls as discovered -->

