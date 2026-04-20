---
applyTo: "server/db/**,server/store/bbolt/**,server/store/kafka/**"
---
# Database Domain — Scoped Instructions

> Full domain reference: `claude/database.md`

## Key Patterns (extracted from domain doc)

## Three-Tier DB Architecture
## bbolt Store (`server/store/bbolt/`)
## blevedb (`server/db/blevedb/`)
- `db.go` — `DB` struct implementing `db.DB`, exposes `Search(q SearchQuery)`
- `indexer.go` — indexes schedules into bleve
- `updater.go` — listens to WatchableStore events, updates bbolt + bleve
## Simple DB (`server/db/simple/db.go`)
## REST Store (live data) (`server/store/rest/rest.go`)
## Critical Rules
- **Always `defer store.Close()`** — bbolt stores must be closed; bleve indexes too
- **DATA_ROOT_DIR trailing slash** — `config.DataRootDir()` ensures trailing slash; file paths concatenate directly
- **Never query liveDB in a hot loop** — every call makes HTTP requests to all scheduler instances
- **bleve index path must be stable** — changing DATA_ROOT_DIR abandons the existing index; rebuilds from scratch
- **bbolt is the write-ahead store** — bleve is rebuilt from bbolt on restart if bleve index is missing/corrupt

> See `claude/database.md` for the complete reference.
