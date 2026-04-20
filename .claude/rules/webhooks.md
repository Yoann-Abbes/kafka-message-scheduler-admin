---
globs: server/restapi/**/*.go,server/store/**/*.go
alwaysApply: false
domain: webhooks
last_verified: 2026-04-20
---

# Event Delivery & Streaming Rules

> Full reference: `claude/webhooks.md`

- Watch() channel buffer = 10000 — never block the consumer; slow consumers drop events
- REST API streams JSON directly to ResponseWriter — do not buffer full result set in memory
- StoreResetType is not idempotent — handle it by clearing and rebuilding local state
- HTTP /info must be reachable before starting admin — resolver fails hard if unavailable
- Kafka consumer replay (earliest) means handlers must tolerate duplicate UpsertType events
