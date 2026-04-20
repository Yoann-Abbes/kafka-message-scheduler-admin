---
globs: server/db/**/*.go,server/store/**/*.go,server/runner/**/*.go
alwaysApply: false
domain: database
last_verified: 2026-04-20
---

# Database Rules

> Full reference: `claude/database.md`

- Three DB instances: `coldDB` (bbolt+bleve+Kafka), `liveDB` (HTTP), `historyDB` (bbolt+bleve+Kafka history topic)
- Always `defer store.Close()` — bbolt + bleve must be closed
- `DATA_ROOT_DIR` default `./.db` — trailing slash guaranteed by `config.DataRootDir()`
- Never query `liveDB` in a hot loop — each call makes HTTP requests to all scheduler instances
- bleve index is rebuilt from bbolt on corruption — do not delete bbolt without also deleting bleve
- `SearchQuery` filter fields: `SchedulerName`, `ScheduleID`, `EpochRange{From, To int64}`, `Limit.Max`
