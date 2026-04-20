# Architectural Decisions — kafka-message-scheduler-admin

> **Append-only log.** Record significant choices here when they're made.
> Reference settled decisions in `CLAUDE.md` → `## Key Decisions` section.
> The **why** is more valuable than the what — future you will thank you.

---

## Format

```
### YYYY-MM-DD — [Decision title]
**Decision:** What was decided.
**Why:** The full reasoning — trade-offs considered, constraints, alternatives rejected and why.
**Decided by:** User / team / Claude recommendation accepted by user.
```

---

## Decisions

### 2024 — Three-tier DB architecture (cold/live/history)

**Decision:** Maintain three independent DB instances: `coldDB` (bbolt+bleve, Kafka-fed), `liveDB` (HTTP-only, no persistence), `historyDB` (bbolt+bleve, history topic).

**Why:** Each view has different consistency requirements. Cold schedules need full-text search and persistence across restarts. Live schedules must be real-time (no cache acceptable). History schedules are immutable after execution. A single DB would require complex query filtering and could not serve all three SLAs. Three independent DB instances with separate routes keeps each layer optimal for its purpose.

**Decided by:** Team (reflected in codebase structure from initial commit).

---

### 2024 — `-tags musl` build tag for confluent-kafka-go

**Decision:** All Go builds and tests require `-tags musl`. This is not optional.

**Why:** confluent-kafka-go uses CGO and links against librdkafka. The musl build tag enables static linking of librdkafka, which eliminates the need to install librdkafka on the host. This is critical for Docker-based deployments (Alpine images) and consistent CI environments. Without musl tag, builds succeed on dev machines with librdkafka installed but fail in CI/Docker.

**Decided by:** Team (enforced via Makefile).

---

### 2024 — SPA + API in single binary

**Decision:** React client build is embedded in the Go binary and served by the same process unless `API_SERVER_ONLY=true`.

**Why:** Simplifies deployment — one Docker image, one port, no reverse proxy needed for the admin UI. The admin is an internal tool; operational simplicity outweighs flexibility of separate processes. `API_SERVER_ONLY=true` escape hatch exists for cases where a separate frontend CDN is preferred.

**Decided by:** Team (reflected in runner design).

---

### 2024 — DNS-based scheduler discovery (no service registry)

**Decision:** Scheduler instances discovered via DNS lookup of `SCHEDULERS_ADDR` + HTTP `/info` endpoint. No Consul/etcd/k8s service registry.

**Why:** The target deployment is Docker Compose / simple Docker environments. DNS resolution (via hostname or IP) is universally available. Adding a service registry would require the scheduler itself to self-register, increasing coupling. The polling approach in `WatchableStoreFromResolver` handles instance changes without a registry.

**Decided by:** Team.

---

### 2024 — Kafka consumer with earliest offset + no auto-commit

**Decision:** `auto.offset.reset: earliest`, `enable.auto.commit: false`. Admin always replays from beginning on restart.

**Why:** The admin is a read-only observer — it does not consume messages for business logic. On restart, it must rebuild its local state (bbolt+bleve) from scratch. Committing offsets would leave the local DB in a partial state after a crash. Replaying from earliest guarantees the local DB always converges to the correct state. Cost: startup time proportional to topic size.

**Decided by:** Team.

---

### 2024 — Optional HTTP message body decoder

**Decision:** `KAFKA_MESSAGE_BODY_DECODER` env var activates an optional HTTP decoder for Kafka message values. Decoder errors are non-fatal.

**Why:** Some deployments encode Kafka message values in a custom binary format. The admin can't know the format at compile time. An HTTP decoder service allows decoding without recompiling the admin. Non-fatal errors ensure the admin remains usable even if the decoder is temporarily unavailable — it falls back to raw bytes.

**Decided by:** Team.
