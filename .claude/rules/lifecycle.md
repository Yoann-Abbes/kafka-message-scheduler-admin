---
globs: server/store/**/*.go,server/runner/**/*.go,server/db/**/*.go,client/src/**/*.ts,client/src/**/*.tsx
alwaysApply: false
domain: schedule-lifecycle
last_verified: 2026-04-20
---

# Schedule Lifecycle Rules

> Full reference: `claude/lifecycle.md`

- `store.EventType`: `UpsertType`, `DeletedType`, `StoreResetType` — handle ALL three
- `StoreResetType` must trigger full state wipe downstream — never ignore it
- `enable.auto.commit: false` + `auto.offset.reset: earliest` → replays on restart; handlers must be idempotent
- Never call `WatchableStore.Close()` while Watch() consumers are still running — causes channel panic
- ScheduleType `'live' | 'all' | 'history'` — maps to `/live/`, `/`, `/history/` route prefixes
- `epoch` = fire time (Unix), `timestamp` = creation time — never confuse them
