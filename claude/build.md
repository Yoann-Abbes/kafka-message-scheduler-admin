# Build, Test & CI Reference — kafka-message-scheduler-admin

## Package Manager & Runtime

- **Package manager**: Go modules (`go.mod`) + npm (client only)
- **Runtime**: Go 1.24+ (server) · Node.js (client, via Vite)
- **Build tag**: `-tags musl` required on all `go build`/`go test`/`go run` commands

## Build Commands

| Scope | Command |
|-------|---------|
| All (verify) | `cd server && go build -tags musl -v ./...` |
| Binary (admin) | `cd server && go build -tags musl -v -o bin/admin ./cmd/kafka` |
| Binary (mini) | `cd server && go build -tags musl -v -o bin/mini ./cmd/mini` |
| Client | `cd client && npm run build` |

## Test Commands

| Scope | Command |
|-------|---------|
| All server tests | `cd server && go test -v -tags musl -count=1 ./...` |
| Single package | `cd server && go test -v -tags musl -count=1 ./path/to/pkg/...` |
| CI mode | `cd server && go test -v -tags musl -count=1 ./...` |
| With coverage | `cd server && go test -cover -tags musl ./...` |
| Integration tests | `cd server && RUN_INTEGRATION_TESTS=yes go test -v -tags musl -failfast -count=1 ./...` |
| Client tests | `cd client && npm test` (vitest run) |
| All (lint+unit+integration) | `cd server && make tests` |

## Lint & Format Commands

| Action | Command |
|--------|---------|
| Check (report only) | `cd server && golangci-lint --timeout 5m --build-tags musl run` |
| Fix (auto-fix) | `cd server && golangci-lint --timeout 5m --build-tags musl run --fix` |
| Format only | `cd server && gofmt -w .` |

## CI Pipeline

GitHub Actions (`.github/`):
1. `golangci-lint --build-tags musl run`
2. `go test -tags musl -count=1 ./...`
3. Docker build

## Local Development

### Prerequisites

- Go 1.22+
- Docker + Docker Compose (for Kafka + scheduler)
- Node.js 18+ (for client)
- confluent-kafka-go requires librdkafka (or build with `-tags musl` for static linking)

### Quick Start

```bash
# Start Kafka + scheduler dependencies
cd server && make up    # docker-compose -p dev up -d kafka scheduler

# Run server
cd server && make start    # go run -tags musl ./cmd/kafka

# Run client (separate terminal)
cd client && npm start    # vite dev server on :5173
```

### ⚠️ Pitfalls (from lessons)

- **`-tags musl` is mandatory** — omitting it causes build/test failures with confluent-kafka-go
- **Integration tests require running Kafka** — set `RUN_INTEGRATION_TESTS=yes` only when infra is up
- **Client needs `npm install` first** — run `cd client && npm install` before `npm start` or `npm test`
- **DATA_ROOT_DIR** — defaults to `./.db` in CWD of server binary, not repo root
