---
name: performance-optimization
description: Optimizes backend performance — queries, caching, memory, and throughput — by measuring first. Use when performance requirements exist, when a regression is suspected, or when profiling reveals a bottleneck such as N+1 queries.
---

# Performance Optimization

## Overview

Measure before optimizing. Profile, find the actual bottleneck, fix that one thing, measure again. Optimize only what measurements prove matters.

## When to Use

- Performance requirements exist (latency SLAs, throughput targets)
- Monitoring or users report slow behavior, or a change may have caused a regression
- Building features that handle large datasets or high traffic

**When NOT to use:** no evidence of a problem — premature optimization adds complexity that costs more than it gains.

## The Workflow

1. **Measure** — baseline with real data (APM, query logs, profiler), not a local guess.
2. **Identify** — find the actual bottleneck, not the assumed one.
3. **Fix** — address that specific bottleneck, one change at a time.
4. **Verify** — measure again under the same conditions; record before/after numbers.
5. **Guard** — add a metric, alert, or test that catches the regression next time.

## Common Bottlenecks

| Symptom | Likely Cause | Investigation |
|---------|-------------|---------------|
| Slow endpoint | N+1 queries, missing index, unoptimized query | SQL logging (`show-sql`, Hibernate statistics), `EXPLAIN ANALYZE` |
| Memory growth | Leaked references, unbounded caches, large result sets | Heap dump (`jmap`, JFR) |
| CPU spikes | Heavy synchronous computation, regex backtracking | JFR / async-profiler CPU sampling |
| High latency, low CPU | Missing cache, pool exhaustion, blocking I/O on the wrong thread | Thread dump, connection pool metrics |

## Fix Rules

- **N+1:** fetch related data in the same round trip (fetch join, batch fetch, or an `IN` query) — whatever the ORM, never one query per row.
- **Unbounded fetches:** every list endpoint paginates. Prefer keyset pagination (`WHERE created_at < :cursor ORDER BY created_at DESC LIMIT n`) over large offsets, which scan and discard every skipped row.
- **Caching:** cache only reads that are frequent and rarely change. Every cache has a TTL *and* a size bound — an unbounded cache is a memory leak. Local (Caffeine) is enough for per-instance data; use a shared cache (Redis) when instances must agree on a value, and decide how it's invalidated on write.
- **Indexes:** add one for the query you measured, then confirm with `EXPLAIN` that the planner uses it.

## Common Rationalizations

| Rationalization | Reality |
|---|---|
| "This optimization is obvious" | If you didn't measure, you don't know. Profile first. |
| "It's fast on my machine" | Your machine isn't production. Profile under realistic load and data volume. |
| "The ORM handles performance" | It won't fix an N+1 query you wrote. |

## Red Flags

- Optimization without profiling data to justify it
- N+1 query patterns in data-fetching code
- List endpoints without pagination
- Caches with no TTL or size bound

## Verification

- [ ] Before/after measurements exist, with specific numbers
- [ ] The specific bottleneck is identified and addressed
- [ ] No N+1 queries in new data-fetching code
- [ ] Existing tests still pass (behavior unchanged)
