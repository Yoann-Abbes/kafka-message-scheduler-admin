# Database Reference — kafka-message-scheduler-admin

## Three-Tier DB Architecture

The server maintains three independent DB instances (`server/runner/kafka/kafka.go`):

| DB | Storage | Index | Source | Route prefix |
|----|---------|-------|--------|--------------|
| `coldDB` | bbolt (`schedules.bbolt`) | bleve (`schedules.bleve`) | Kafka topics (WatchableStore) | `/scheduler/` |
| `liveDB` | in-memory only | none | HTTP calls to scheduler instances | `/live/scheduler/` |
| `historyDB` | bbolt (`history.bbolt`) | bleve (`history.bleve`) | Kafka history topic | `/history/scheduler/` |

All three implement `db.DB` interface (`server/db/db.go`):
```go
type DB interface {
    store.Store
    Search(q SearchQuery) (int, chan schedule.Schedule, error)
}
```

## bbolt Store (`server/store/bbolt/`)

Persistent local KV store. Path set by `DATA_ROOT_DIR` env (default `./.db`).
- Files: `{DATA_ROOT_DIR}schedules.bbolt` and `{DATA_ROOT_DIR}history.bbolt`
- Used as `InternalStore` inside blevedb — bbolt is source of truth, bleve is the search index on top

## blevedb (`server/db/blevedb/`)

Full-text search DB = bbolt (persistence) + bleve (index). Three components:
- `db.go` — `DB` struct implementing `db.DB`, exposes `Search(q SearchQuery)`
- `indexer.go` — indexes schedules into bleve
- `updater.go` — listens to WatchableStore events, updates bbolt + bleve

`SearchQuery` parameters (`server/db/db.go`):
```go
type SearchQuery struct {
    Limit  // Max int
    Filter // SchedulerName, ScheduleID, EpochRange{From, To int64}
    SortBy sort.By
}
```

## Simple DB (`server/db/simple/db.go`)

Wraps a `store.Store` with no persistence — used for `liveDB` backed by `rest.HTTPRetriever`.

## REST Store (live data) (`server/store/rest/rest.go`)

No local persistence. On every `List()` / `Get()` call:
1. `resolver.List()` → discover scheduler instances
2. `GET {instance}:{port}/schedules` → parse JSON → filter
3. Optional: decode via `decoder.Decoder` (KAFKA_MESSAGE_BODY_DECODER)

## Critical Rules

- **Always `defer store.Close()`** — bbolt stores must be closed; bleve indexes too
- **DATA_ROOT_DIR trailing slash** — `config.DataRootDir()` ensures trailing slash; file paths concatenate directly
- **Never query liveDB in a hot loop** — every call makes HTTP requests to all scheduler instances
- **bleve index path must be stable** — changing DATA_ROOT_DIR abandons the existing index; rebuilds from scratch
- **bbolt is the write-ahead store** — bleve is rebuilt from bbolt on restart if bleve index is missing/corrupt
