---
applyTo: "server/restapi/**,server/store/**"
---
# Webhooks Domain — Scoped Instructions

> Full domain reference: `claude/webhooks.md`

## Key Patterns (extracted from domain doc)

## Scheduler Info Callback
## WatchableStore Watch() channel
## Event Idempotency
- `UpsertType` → re-upsert is safe (overwrite with same or newer data)
- `DeletedType` → re-delete is safe (no-op if already deleted)
- `StoreResetType` → triggers full store wipe + rebuild; must be handled; not idempotent itself
## HTTP Response Pattern (`server/restapi/routes.go`)
## Critical Rules
- **Never block the Watch() channel consumer** — the channel has `ChanSize = 10000` buffer; if consumer is slow, events drop
- **StoreReset is not idempotent** — it wipes state; duplicate StoreResets are safe but wasteful; root cause is config change
- **HTTP /info endpoint must be available** — resolver fails hard if `/info` is unreachable; ensure scheduler is running before starting admin
- **Streaming response must not close early** — `routes.go` streams directly to `http.ResponseWriter`; closing before channel drains causes partial JSON

> See `claude/webhooks.md` for the complete reference.
