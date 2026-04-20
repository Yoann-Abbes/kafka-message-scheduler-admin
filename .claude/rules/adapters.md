---
globs: server/store/rest/**/*.go,server/decoder/**/*.go,server/resolver/**/*.go
alwaysApply: false
domain: adapters
last_verified: 2026-04-20
---

# Adapter Rules

> Full reference: `claude/adapters.md`

- All data sources implement `store.Store` — do not bypass the interface
- `httpdecoder.Decoder` errors are non-fatal — log warning, return original schedule
- `rest.HTTPRetriever` timeout = 5s, `httpresolver` timeout = 2s — never remove timeouts
- `httpresolver.ErrPartialResults` is acceptable — continue with partial list
- `httpresolver.ErrNoResults` = critical — all schedulers unreachable
- Resolver keeps only IPv4 addresses from DNS lookup — do not add IPv6 logic without testing
- New adapters must implement `store.Store` or `decoder.Decoder` — do not add intermediate types
