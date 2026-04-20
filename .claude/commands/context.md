---
description: Load all relevant domain context files for a topic area
effort: high
argument-hint: "[domain-keyword e.g. api|database|auth|build|security]"
---

Load all relevant context for the domain area: $ARGUMENTS

## Instructions

Based on the domain keyword(s) provided, read the appropriate knowledge files.

### Domain → Files mapping:
- **build/test/CI/lint** → `claude/build.md`
- **MR/PR/ticket/template** → `claude/templates.md`
- **CVE/security/dependency** → `claude/cve-policy.md`
- **terminal/command/shell** → `claude/terminal-safety.md`
- **schedule lifecycle, store events, WatchableStore, ScheduleType** → `claude/lifecycle.md`
- **database, bbolt, bleve, coldDB, liveDB, historyDB, SearchQuery** → `claude/database.md`
- **adapters, REST store, HTTP decoder, resolver, httpresolver** → `claude/adapters.md`
- **webhooks, event delivery, Watch() channel, streaming JSON** → `claude/webhooks.md`

### Always read:
1. `claude/tasks/lessons.md` (accumulated wisdom)
2. `claude/architecture.md` (system overview)
3. `claude/rules.md` (golden rules)

After reading, provide a brief summary of the loaded context and ask what task to perform.

