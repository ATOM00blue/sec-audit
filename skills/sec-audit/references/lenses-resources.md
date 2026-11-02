# Specialist lens: resource exhaustion and cost

Use when untrusted requests, messages, files, or agent work can consume CPU, memory, connections, worker slots, queue capacity, or paid APIs, or can crash or deadlock a shared service.

## Discipline

- Require an input-to-cost path, a missing effective bound, and impact on another user, a shared service, or operator spend. Self-limiting work is not a finding.
- A missing rate limit is not enough. Check body caps, concurrency, queues, deadlines, database constraints, gateways, and per-tenant quotas first.
- Never validate by stressing anything. Reason from complexity and code.
- State attacker cost, service work, persistence, scope, and recovery. One small input with lasting effect is very different from sustained volume.
- Upstream caps, autoscaling, and platform limits outside the repo make the claim `needs_validation`.

## Amplification

- **Superlinear work:** regex backtracking, nested parsing, recursive validation, graph traversal, template expansion. Derive accepted depth and complexity.
- **Expansion:** compressed or nested input growing past the checked size. Limits must apply after each expansion.
- **Query amplification:** broad scans, joins, unbounded sorts, or many downstream calls from one request, because depth, filters, pagination, or expansions aren't bounded.

## Accumulation

- **Unbounded cardinality:** bodies, streams, sessions, unique cache keys, metric labels, log fields, subscriptions, or pending jobs with no per-item and aggregate limit. Find cleanup on disconnect, timeout, cancel, and partial parse.
- **Leaks:** malformed or cancelled work holding sockets, cursors, timers, or subprocesses from a shared pool.
- **Detached work:** a client timeout or cancel returns, but database, model, or worker work keeps running and keeps costing. Trace cancellation through every layer, including LLM streams.

## Quotas and scheduling

- **Pre-auth imbalance:** expensive parsing, key lookup, crypto, or outbound calls before authentication and the first size or rate gate.
- **Quota scope:** accounting keyed on something the attacker influences, such as IP, route, key prefix, or task ID, so one principal escapes its budget or spends another's. Check races, retries, reconnects, account switching, reset windows, and counting requested versus actual usage.
- **Global versus instance:** a limit held in one worker's memory isn't global. A check-then-spend across concurrent requests can overspend.
- **Starvation:** attacker jobs holding locks, workers, database connections, or scheduler priority beyond their deadline.

## Failure and recovery

- **Reachable fatal error:** input reaching an unhandled exception, process exit, lock cycle, or infinite loop. State what restarts and what stays down. In Workers, one isolate failing is much smaller than a whole service.
- **Retry storm:** failures causing synchronized or unbounded retries with no jitter, ceiling, circuit breaker, or dedupe.
- **Poison record:** one bad message blocking a shared queue, startup scan, migration, or recovery loop. Check quarantine and who shares the blocked unit.
- **Unsafe recovery:** a restart or restore that rebuilds unbounded state, ignores current quotas, or reloads the input that caused the failure.

## Moves

Build a table for each input: earliest accepted size, work before auth, downstream fan-out, persistence, shared pool, who owns limit and cleanup, and recovery. Compare aggregate limits with per-object limits: ten thousand small valid items can evade a per-message cap.
