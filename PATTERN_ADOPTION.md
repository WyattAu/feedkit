# PATTERN_ADOPTION

feedkit declares its relationship to the WyattAu pattern canon
(github.com/WyattAu/pattern-standards). Validated by
`pattern-standards/scripts/adoption-check.py`.

The crate is poll-only feed ingestion: adapters fetch upstream feeds,
normalize payloads into typed events, and dedupe across polls. There
are no webhook receivers (hinted P54 does not apply: nothing listens,
every path is outbound polling), so P54 is omitted.

| Pattern | Status | Evidence |
|---|---|---|
| T1 | adopted | src/usgs.rs:50 (GeoJSON feature parsed to a typed Observation at the boundary; downstream consumes structs, no re-validation), src/event.rs:117 (RFC 3339 parsed once to epoch ms), src/news.rs:63 (RSS parsed once on ingest) |
| T13 | adopted | src/source.rs:24 (Source::poll is the single I/O boundary; the platform owns the polling loop), src/usgs.rs:50 (normalization is an I/O-free transform over the payload) |
| P51 | partial | src/usgs.rs:29 (circuit breaker wired on the shared client used by usgs, eonet and swpc), src/nvd.rs:29 (nvd and news builders stay retry-only, no breaker) |
| P70 | partial | src/usgs.rs:59 (optional fields default, provider-added fields cannot break the parse), src/usgs.rs:117 (the envelope itself stays strict: missing features array is Malformed) |
| P107 | partial | src/usgs.rs:52 (seen-set inbox gates re-emission of already-delivered ids across polls; contract stated at src/source.rs:20), src/usgs.rs:121 (capacity overflow clears the set wholesale, so cleared ids can double-apply after a flush) |
| P21 | partial | src/event.rs:36 (occurred_at_ms and fetched_at_ms carried per item, so each caller declares staleness against fetch time instead of a global TTL), src/event.rs:45 (ts_ms on Quote carries the same raw material; no staleness gate in-crate) |
