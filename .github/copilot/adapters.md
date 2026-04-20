# Adapters & Integrations — kafka-message-scheduler-admin

## Adapter Interfaces

### `store.Store` interface (`server/store/store.go`)

All data sources implement this interface:
```go
type Store interface {
    Get(schedulerName string, scheduleID string) ([]Schedule, error)
    List(schedulerName string) (chan Schedule, error)
}
```

Extended interfaces: `WatchableStore`, `MutableStore`, `BatchableStore`.

### `decoder.Decoder` interface (`server/decoder/decoder.go`)

Optional transformation layer for Kafka message bodies:
```go
type Decoder interface {
    Decode(s schedule.Schedule) (schedule.Schedule, error)
}
```

Configured via `KAFKA_MESSAGE_BODY_DECODER` env var → activates `httpdecoder.Decoder`.

## HTTP Adapter: `rest.HTTPRetriever` (`server/store/rest/rest.go`)

Adapter from scheduler HTTP API to `store.Store`:
- `List(schedulerName)` → `GET {instance}:{port}/schedules` for each instance of the named scheduler
- `Get(schedulerName, scheduleID)` → same, then filter by ID
- Uses `httpresolver.Resolver` to discover live instances
- Decodes via optional `decoder.Decoder` if configured
- `DefaultTimeout = 5s` per HTTP request

**Pattern**: resolver discovery + HTTP GET + JSON decode + optional decode pass.

## HTTP Adapter: `httpdecoder.Decoder` (`server/decoder/httpdecoder/`)

Sends Kafka schedule body to an external HTTP decoder service:
- POST to `KAFKA_MESSAGE_BODY_DECODER` URL with schedule payload
- Returns transformed `schedule.Schedule`
- Decode errors are non-fatal: logged as warning, original schedule used as fallback

## Resolver Adapter: `httpresolver.Resolver` (`server/resolver/schedulers/httpresolver/http.go`)

Adapts scheduler instances (DNS-resolved) to `schedulers.Resolver`:
- Input: `SCHEDULERS_ADDR` (comma-separated `host:port` or `host`)
- DNS `LookupIP` → keep IPv4 only
- For each IP: `GET {ip}:{port}/info` → parse `kafka.info{BootstrapServers, Topics, HistoryTopic}`
- Output: `[]schedulers.Scheduler` with `Instances[]` per hostname

Error handling:
- `ErrNoResults` — all hosts failed (DNS or HTTP)
- `ErrPartialResults` — some hosts failed (partial list returned)

## Critical Rules

- **Decode errors are non-fatal** — `httpdecoder` returns original schedule on failure; do not make this fatal
- **Resolver partial results are acceptable** — `WatchableStoreFromResolver` continues with available schedulers
- **All HTTP adapters must respect timeouts** — `rest.DefaultTimeout = 5s`, `httpresolver.DefaultTimeout = 2s`
- **Never share `http.Client` across adapters without timeout** — default client has no timeout
- **Resolver is called on a polling timer** — do not add expensive operations to the resolver path
