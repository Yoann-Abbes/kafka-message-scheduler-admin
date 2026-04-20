---
applyTo: "server/store/**,server/runner/**,client/src/business/scheduler/**"
---
# Lifecycle Domain — Scoped Instructions

> Full domain reference: `claude/lifecycle.md`

## Key Patterns (extracted from domain doc)

## Schedule States
## Store Event Lifecycle
## Schedule Fields
## WatchableStore Lifecycle
## Critical Rules
- **Never call `Close()` on WatchableStore before stopping all Watch() consumers** — causes "send on closed channel" panic
- **StoreResetType must clear downstream state** — if a handler ignores StoreReset, stale schedules accumulate
- **`enable.auto.commit: false`** — the Kafka consumer never auto-commits; offsets not tracked between restarts → always reads from earliest
- **WatchableStoreFromResolver** (`server/runner/kafka/watchable_store.go`) polls resolver on a timer to detect new scheduler instances; bucket changes trigger StoreReset
## Resolver Lifecycle

> See `claude/lifecycle.md` for the complete reference.
