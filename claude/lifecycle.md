# Schedule Lifecycle — kafka-message-scheduler-admin

## Schedule States

A schedule progresses through states exposed via three DB views:

```
Kafka topic → WatchableStore → blevedb (coldDB)   → GET /scheduler/{n}/schedules
                             → rest.HTTPRetriever  → GET /live/scheduler/{n}/schedules
                             → blevedb (historyDB) → GET /history/scheduler/{n}/schedules
```

Client ScheduleType: `'live' | 'all' | 'history'` (`client/src/business/scheduler/type/index.ts`)

## Store Event Lifecycle

`store.EventType` (`server/store/store.go`):

```go
const (
    UpsertType    EventType = iota  // schedule created or updated
    DeletedType                      // schedule deleted from Kafka topic (tombstone)
    StoreResetType                   // all schedules cleared — topic/consumer change
)
```

`StoreResetType` fires when a scheduler's bootstrap servers or topics change (`WatchableStore.AddBuckets` in `server/store/kafka/watchable.go`). Downstream must clear + rebuild their local state.

## Schedule Fields

Go side (`server/store/rest/rest.go`):
```go
type Schedule struct {
    ScheduleID         string `json:"id"`
    ScheduleEpoch      int64  `json:"epoch"`        // Unix epoch — when to fire
    ScheduleTimestamp  int64  `json:"timestamp"`     // creation time
    MessageTargetTopic string `json:"target-topic"`
    MessageTargetKey   string `json:"target-key"`
    MessageTopic       string `json:"topic"`
    MessageValue       []byte `json:"value"`
}
```

TypeScript side (`client/src/business/scheduler/type/index.ts`):
```ts
type ScheduleInfo = {
    id: string; scheduler: string;
    timestamp: number; epoch: number;
    targetTopic: string; targetId: string;
}
type Schedule = ScheduleInfo & { topic: string; value: string; }
```

## WatchableStore Lifecycle

`server/store/kafka/watchable.go` — `WatchableStore` manages Kafka consumers per scheduler bucket:

1. `NewWatchableStore(dec, buckets...)` — creates consumers, starts processor goroutine
2. `AddBuckets(buckets...)` — dynamically adds/replaces consumers when scheduler config changes
3. `Watch()` → `chan store.Event` — downstream subscribes to receive Upsert/Deleted/StoreReset
4. `Close()` — stops all consumers, waits for poll timeout (1s sleep needed to avoid channel panic)

## Critical Rules

- **Never call `Close()` on WatchableStore before stopping all Watch() consumers** — causes "send on closed channel" panic
- **StoreResetType must clear downstream state** — if a handler ignores StoreReset, stale schedules accumulate
- **`enable.auto.commit: false`** — the Kafka consumer never auto-commits; offsets not tracked between restarts → always reads from earliest
- **WatchableStoreFromResolver** (`server/runner/kafka/watchable_store.go`) polls resolver on a timer to detect new scheduler instances; bucket changes trigger StoreReset

## Resolver Lifecycle

`httpresolver.Resolver.List()` (`server/resolver/schedulers/httpresolver/http.go`):
1. Iterates `SCHEDULERS_ADDR` hosts
2. DNS lookup → keeps only IPv4
3. `GET /{host}:{port}/info` → extracts `bootstrap_servers`, `topics`, `history_topic`
4. Returns `ErrNoResults` if all hosts fail; `ErrPartialResults` if some fail
5. Errors are non-fatal for WatchableStoreFromResolver — partial results still used
