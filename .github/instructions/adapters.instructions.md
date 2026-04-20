---
applyTo: "server/store/rest/**,server/decoder/**,server/resolver/**"
---
# Adapters Domain — Scoped Instructions

> Full domain reference: `claude/adapters.md`

## Key Patterns (extracted from domain doc)

## Adapter Interfaces
## HTTP Adapter: `rest.HTTPRetriever` (`server/store/rest/rest.go`)
- `List(schedulerName)` → `GET {instance}:{port}/schedules` for each instance of the named scheduler
- `Get(schedulerName, scheduleID)` → same, then filter by ID
- `DefaultTimeout = 5s` per HTTP request
## HTTP Adapter: `httpdecoder.Decoder` (`server/decoder/httpdecoder/`)
## Resolver Adapter: `httpresolver.Resolver` (`server/resolver/schedulers/httpresolver/http.go`)
- `ErrNoResults` — all hosts failed (DNS or HTTP)
- `ErrPartialResults` — some hosts failed (partial list returned)
## Critical Rules
- **Decode errors are non-fatal** — `httpdecoder` returns original schedule on failure; do not make this fatal
- **Resolver partial results are acceptable** — `WatchableStoreFromResolver` continues with available schedulers
- **All HTTP adapters must respect timeouts** — `rest.DefaultTimeout = 5s`, `httpresolver.DefaultTimeout = 2s`
- **Never share `http.Client` across adapters without timeout** — default client has no timeout
- **Resolver is called on a polling timer** — do not add expensive operations to the resolver path

> See `claude/adapters.md` for the complete reference.
