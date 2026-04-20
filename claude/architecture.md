# Architecture Reference — kafka-message-scheduler-admin

Admin UI and REST API server for monitoring and inspecting [kafka-message-scheduler](https://github.com/etf1/kafka-message-scheduler) instances. Consumes Kafka topics to maintain three parallel views of scheduled messages (cold/live/history) and serves a React SPA.

## Workspace Layout

| Directory | Purpose |
|-----------|---------|
| `server/` | Go backend — REST API, Kafka consumer, DB layers |
| `client/` | React/TypeScript SPA — Vite + Bulma CSS |
| `docs/` | Project documentation |
| `claude/` | AI assistant knowledge base |

## Service / Module Catalog

| Service | Directory | Purpose | Stack |
|---------|-----------|---------|-------|
| `admin` | `server/cmd/kafka/` | Full server: API + SPA — primary binary | Go, confluent-kafka-go, bbolt, bleve |
| `mini` | `server/cmd/mini/` | Minimal server variant | Go |
| `restapi` | `server/restapi/` | HTTP routes for coldDB / liveDB / historyDB | Go, gorilla/mux |
| `runner/kafka` | `server/runner/kafka/` | Orchestrates all DB layers + HTTP server lifecycle | Go |
| `store/kafka` | `server/store/kafka/` | Kafka consumer → WatchableStore (events) | Go, confluent-kafka-go |
| `store/bbolt` | `server/store/bbolt/` | Persistent key-value store | Go, bbolt |
| `store/rest` | `server/store/rest/` | Live schedules fetched from scheduler HTTP API | Go |
| `db/blevedb` | `server/db/blevedb/` | Full-text search DB backed by bbolt + bleve | Go, blevesearch |
| `db/simple` | `server/db/simple/` | Simple DB wrapping a store | Go |
| `resolver/schedulers` | `server/resolver/schedulers/` | Discovers scheduler instances via DNS + HTTP | Go |
| `decoder/httpdecoder` | `server/decoder/httpdecoder/` | Optional HTTP decoder for Kafka message bodies | Go |
| `client` | `client/src/` | Admin UI — schedule browser, scheduler list, stats | React 18, TypeScript, Bulma |

## Service Types (Summary)

| Type | Examples | Stack |
|------|----------|-------|
| Backend API + SPA server | `admin`, `mini` | Go, gorilla/mux, confluent-kafka-go |
| Kafka consumer stores | `store/kafka` | confluent-kafka-go, bbolt |
| Search DB | `db/blevedb` | bleve, bbolt |
| HTTP adapter stores | `store/rest`, `decoder/httpdecoder` | net/http |
| Frontend SPA | `client` | React 18, TypeScript, Vite, Bulma |

## Shared Packages / Libraries

| Package | Directory | Key exports |
|---------|-----------|-------------|
| `store` | `server/store/` | `Store`, `WatchableStore`, `BatchableStore`, `MutableStore`, `Event`, `EventType` interfaces |
| `db` | `server/db/` | `DB` interface, `SearchQuery`, `Filter`, `EpochRange` |
| `config` | `server/config/` | `SchedulersAddr()`, `KafkaMessageBodyDecoder()`, `DataRootDir()`, `APIServerOnly()` |
| `helper` | `server/helper/` | HTTP utils, startup/shutdown, JSON decode |
| `resolver/schedulers` | `server/resolver/schedulers/` | `Resolver`, `Scheduler` interfaces |
| `_common` | `client/src/_common/` | Shared API utilities |
| `_core` | `client/src/_core/` | URL config, shared services |

## Package Aliases / Import Conventions

Client TypeScript aliases (vite.config / tsconfig):
- `_common` → `client/src/_common/`
- `_core` → `client/src/_core/`

Go module: `github.com/etf1/kafka-message-scheduler-admin/server`

## Key Infrastructure

| Component | Purpose |
|-----------|---------|
| Kafka (confluent-kafka-go) | Consumes scheduler topics for cold+history stores |
| bbolt | Local embedded KV store for persistent schedule data |
| bleve | Full-text search index over bbolt data |
| kafka-message-scheduler | External scheduler instances — admin connects via `SCHEDULERS_ADDR` |
| Docker Compose | Local dev: `docker-compose -p dev up -d kafka scheduler` |

## Namespace / Environment Map

| Env Var | Default | Purpose |
|---------|---------|---------|
| `SCHEDULERS_ADDR` | `localhost:8000` | Comma-separated scheduler hosts |
| `SERVER_ADDR` | `:9000` | Admin server listen address |
| `METRICS_ADDR` | `:9001` | Prometheus metrics endpoint |
| `API_SERVER_ONLY` | `false` | Serve API only (no SPA) |
| `DATA_ROOT_DIR` | `./.db` | bbolt + bleve data directory |
| `KAFKA_MESSAGE_BODY_DECODER` | `""` | Optional HTTP decoder URL |
| `LOG_LEVEL` | `info` | logrus log level |

## Three-DB Architecture

The admin exposes three views of schedules via separate DB instances:

```
coldDB   = bbolt (persist) + bleve (search) + WatchableStore (Kafka feed)
liveDB   = simple.DB wrapping rest.HTTPRetriever (live HTTP calls)
historyDB = bbolt + bleve + WatchableStore (history Kafka topic)
```

Routes:
- `/scheduler/{name}/schedules` — coldDB (persisted)
- `/live/scheduler/{name}/schedules` — liveDB (real-time)
- `/history/scheduler/{name}/schedules` — historyDB (executed)
