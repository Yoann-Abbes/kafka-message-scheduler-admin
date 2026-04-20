# Webhooks & Callbacks — kafka-message-scheduler-admin

## Scheduler Info Callback

When the runner starts, it discovers schedulers via `httpresolver.Resolver.List()` which calls `GET /info` on each scheduler instance. This is the primary "callback" pattern in this project.

## WatchableStore Watch() channel

`store.WatchableStore.Watch()` returns `chan store.Event` — the consumer of this channel receives all schedule events asynchronously. This is an internal event-delivery pattern (not HTTP webhooks), but follows the same idempotency requirements.

## Event Idempotency

Events from Kafka topics are consumed with `auto.offset.reset: earliest` and `enable.auto.commit: false`. On restart, all messages are replayed from the beginning. Downstream handlers (blevedb updater) must be idempotent:

- `UpsertType` → re-upsert is safe (overwrite with same or newer data)
- `DeletedType` → re-delete is safe (no-op if already deleted)
- `StoreResetType` → triggers full store wipe + rebuild; must be handled; not idempotent itself

## HTTP Response Pattern (`server/restapi/routes.go`)

The REST API uses chunked JSON streaming for schedule lists:
```go
fmt.Fprintf(w, "{%q: %d, %q: [", "found", found, "schedules")
// stream each schedule via json.Encoder
fmt.Fprintf(w, "]}")
```

Do not buffer entire result sets — schedules are streamed from `chan schedule.Schedule`.

## Critical Rules

- **Never block the Watch() channel consumer** — the channel has `ChanSize = 10000` buffer; if consumer is slow, events drop
- **StoreReset is not idempotent** — it wipes state; duplicate StoreResets are safe but wasteful; root cause is config change
- **HTTP /info endpoint must be available** — resolver fails hard if `/info` is unreachable; ensure scheduler is running before starting admin
- **Streaming response must not close early** — `routes.go` streams directly to `http.ResponseWriter`; closing before channel drains causes partial JSON
