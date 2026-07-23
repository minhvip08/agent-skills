---
name: performance-optimization
description: Optimizes backend application performance — queries, caching, memory, and throughput. Use when performance requirements exist, when you suspect a regression, when N+1 query patterns need fixing, or when profiling reveals bottlenecks.
---

# Performance Optimization

## Overview

Measure before optimizing. Performance work without measurement is guessing. Profile first, identify the actual bottleneck, fix it, measure again. Optimize only what measurements prove matters.

## When to Use

- Performance requirements exist (response time SLAs, throughput targets)
- Monitoring or users report slow behavior
- You suspect a change introduced a regression
- Building features that handle large datasets or high traffic

**When NOT to use:** Don't optimize before you have evidence of a problem — premature optimization adds complexity that costs more than it gains.

## The Workflow

```
1. MEASURE  → Establish baseline with real data (APM, query logs, profiler)
2. IDENTIFY → Find the actual bottleneck, not the assumed one
3. FIX      → Address that specific bottleneck
4. VERIFY   → Measure again, confirm improvement
5. GUARD    → Add monitoring/tests to catch regression
```

## Common Bottlenecks

| Symptom | Likely Cause | Investigation |
|---------|-------------|---------------|
| Slow endpoint | N+1 queries, missing index, unoptimized query | Enable SQL logging (`show-sql`, Hibernate statistics) |
| Memory growth | Leaked references, unbounded caches, large result sets | Heap dump analysis (`jmap`, JFR) |
| CPU spikes | Synchronous heavy computation, regex backtracking | JFR / async-profiler CPU sampling |
| High latency, low CPU | Missing caching, thread pool exhaustion, blocking I/O on the wrong thread | Thread dump, connection pool metrics |

## Fix Common Anti-Patterns

### N+1 Queries

```java
// BAD: one query per task to fetch its owner
List<Task> tasks = taskRepository.findAll();
for (Task task : tasks) {
    User owner = userRepository.findById(task.getOwnerId()).orElseThrow();
}

// GOOD: fetch join in one query
@Query("SELECT t FROM Task t JOIN FETCH t.owner")
List<Task> findAllWithOwner();
```

```kotlin
// GOOD (Kotlin/Spring Data): same fix, fetch join or an entity graph
@EntityGraph(attributePaths = ["owner"])
fun findAll(): List<Task>
```

### Unbounded Data Fetching

```java
// BAD: loads the entire table into memory
List<Task> allTasks = taskRepository.findAll();

// GOOD: paginate
Page<Task> tasks = taskRepository.findAll(PageRequest.of(page, 20, Sort.by("createdAt").descending()));
```

### Missing Caching

Cache reads that are frequent and rarely change; always set a TTL and an eviction policy — an unbounded cache is a memory leak.

```java
@Cacheable(value = "appConfig", cacheManager = "shortLivedCacheManager")
public AppConfig getAppConfig() {
    return configRepository.findFirst();
}
```

A local cache (Caffeine) is enough for single-instance data; use a shared cache (Redis) when multiple instances must see the same value.

## Common Rationalizations

| Rationalization | Reality |
|---|---|
| "We'll optimize later" | Performance debt compounds. Fix obvious anti-patterns now, defer micro-optimizations. |
| "It's fast on my machine" | Your machine isn't production. Profile under realistic load and data volume. |
| "This optimization is obvious" | If you didn't measure, you don't know. Profile first. |
| "The framework handles performance" | Frameworks (JPA, Spring) prevent some issues but won't fix an N+1 query you wrote. |

## Red Flags

- Optimization without profiling data to justify it
- N+1 query patterns in data-fetching code
- List endpoints without pagination
- Caches with no TTL or eviction policy
- No performance monitoring in production

## Verification

After any performance-related change:

- [ ] Before/after measurements exist (specific numbers)
- [ ] The specific bottleneck is identified and addressed
- [ ] No N+1 queries in new data-fetching code
- [ ] Existing tests still pass (optimization didn't change behavior)
