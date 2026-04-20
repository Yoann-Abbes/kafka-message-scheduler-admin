---
description: Build project — all, single service, or specific target
disable-model-invocation: true
effort: low
argument-hint: "[all|service-name|packages|frontend]"
---

Build: $ARGUMENTS

## Instructions

Read `claude/build.md` for build reference.

### Determine scope from arguments:

| Argument | Action |
|----------|--------|
| `all` | `cd server && go build -tags musl -v ./...` |
| `admin` | `cd server && go build -tags musl -v -o bin/admin ./cmd/kafka` |
| `mini` | `cd server && go build -tags musl -v -o bin/mini ./cmd/mini` |
| `client` | `cd client && npm run build` |
<!-- Add more build targets as discovered by /bootstrap -->

### Post-build:

1. Check exit code
2. If build fails, read the error output
3. Diagnose the root cause
4. Fix autonomously and re-build

