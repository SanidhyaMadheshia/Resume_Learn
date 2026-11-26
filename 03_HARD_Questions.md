# 03 — HARD Questions (deep dives, failure modes, scaling)

These separate "used the tool" from "understands the system". Structure:
**Clarify → Approach → Deep detail → Failure modes → Trade-offs → What I'd change.**
It's fine to pause, draw, and say "let me think about that".

---

## H1. "Draw LocalArena's architecture and walk me through a purchase end to end."

```
Browser / Go bots ──WebSocket──► Envoy ──► Game servers (match actors, 20 Hz)
                                   │
                                   └──► API (Go) ──► PgBouncer ──► PostgreSQL (primary + replica)
                                          │                         │
                                          └──► Valkey (cache,       └─ outbox table ─► Debezium (WAL/CDC)
                                               presence, rate limit)                     │
                                                                                         ▼
                                                                                      Kafka
                                                                     ┌─────────────────┼──────────────┐
                                                               Leaderboard        Anti-cheat      Sessionizer
                                                              (Kafka Streams, Java) ...
All ─► OpenTelemetry ─► Prometheus/Grafana (metrics, SLO alerts), Loki (logs), Tempo (traces)
```

**Purchase flow (`POST /v1/purchases`, with `Idempotency-Key`):**
1. Envoy routes to an API replica (timeouts/retries configured, **dependency 503s not retried**).
2. API authenticates (JWT) and authorizes (OPA policy; if OPA is down, **fail closed**).
3. One PostgreSQL transaction:
   - (a) `INSERT idempotency (player_id, key) ON CONFLICT DO NOTHING` — a concurrent duplicate blocks then **replays** the original result.
   - (b) Conditional update: `UPDATE cosmetics SET stock = stock-1 WHERE id=$1 AND stock > 0`; same for credits.
   - (c) Grant the item to inventory.
   - (d) Insert the **outbox row** with a deterministic event ID.
4. Commit → state + event are atomic.
5. Debezium reads the WAL, publishes to `game.economy-events.v1` keyed by `player_id` (per-player ordering).
6. Kafka Streams / consumers update projections idempotently.

**Failure modes to mention:** Kafka down → outbox rows wait, nothing lost (measured: 60 s outage, 0 lost). Valkey down → read from Postgres. Retry storm → see H4.

---

## H2. "Exactly-once in Kafka — does it exist? What did you actually guarantee?"
> "Kafka's `exactly_once_v2` gives exactly-once **within Kafka**: consume → process → produce, plus state stores and offset commits, all in one Kafka transaction. It does **not** make side effects outside Kafka — a PostgreSQL write, an HTTP call, an email — exactly once.
>
> So my end-to-end guarantee is **at-least-once delivery + idempotent processing = effectively-once results**. Deterministic event IDs, unique constraints, and dedupe in consumers. I tested it by replaying 1.45M duplicate events: zero duplicate rewards."

**Follow-up:** "What does a consumer do if it crashes after writing to the DB but before committing the offset?" → It re-reads and re-processes; the DB unique constraint/dedupe makes the second write a no-op. Optionally store the offset in the same DB transaction.

---

## H3. "Kafka ordering — how do you guarantee events for a match are in order? What breaks it?"
> "Kafka orders only **within a partition**. Match events are keyed by `match_id`, so all of one match's events go to one partition in order. Economy events are keyed by `player_id` for per-player ordering.
>
> Things that break it: changing the **partition count** (key→partition mapping changes), producer **retries** with `max.in.flight > 1` without idempotence (enable `enable.idempotence=true`), and **repartitioning** — e.g. leaderboards need data by player, so match events get re-keyed by `player_id`, which creates a new ordering scope. Consumers must use event time, not arrival order, and handle late events with grace periods."

---

## H4. "Tell me about the retry storm." (failure from FINAL_REPORT, week 3)
**Situation:** Simulated DB brownout.
**Observed:** **7,482 proxy retries** — Envoy kept retrying, multiplying load on a DB that was already slow.
**Root cause:** The **per-try timeout at the proxy was shorter than the DB timeout**, so Envoy gave up and retried while the original query was still running — every retry added another query.
**Fix — a timeout budget:** `DB timeout < per-try timeout < API deadline < route timeout`, and **don't retry dependency 503s** (if the dependency says it's overloaded, retrying makes it worse). Retries only with backoff + jitter and a retry budget.
**Lesson:** Retries are a load multiplier. Timeouts must be ordered inner < outer.

---

## H5. "Walk me through an incident you debugged." (PM-001 — strongest debugging story)
**S:** Game-day drill with a hidden injected fault; 100 req/s mixed load.
**T:** I was on call: detect, diagnose, mitigate.
**A:**
- **Detect:** fast-burn SLO alert paged at **1 m 40 s** ("7.7% 5xx over 1 h").
- **Narrow down:** 5xx only on routes that need the **primary** DB pool; replica-pool routes healthy → not the whole API.
- **Logs (Loki):** `context canceled`, but **no SQL errors**.
- **Traces (Tempo):** the slow span was a 2.5 s HTTP span **with no SQL child span** → time was spent **before** reaching Postgres.
- **Postgres:** 1 active session, 0 lock waits → not the DB.
- **PgBouncer `SHOW POOLS`:** `cl_waiting=30, sv_active=0`; `SHOW DATABASES`: `paused=1` → the pool was paused.
- **Mitigate:** `RESUME` → 5xx dropped to 0 instantly.
**R:** Diagnosed in **1 m 6 s**, total outage **3 m 46 s**, **no data lost** (failed requests never committed; purchases idempotent).
**Follow-ups I wrote:** add PgBouncer metrics/alerts (the decisive signal came from a manual command), and make errors carry the cause.
**Lesson:** The **absence** of a signal (no SQL span) is a signal. Traces beat guesses.

---

## H6. "Your health check (readiness) caused an outage. Explain." (week 3)
> "Readiness checked the database. When the DB had a blip, **every** API pod failed readiness simultaneously, Kubernetes removed all of them from the load balancer, and the outage got worse — traffic that could have been served from cache got nothing. Also a single operator pod and the pooler and API were on one node, so node loss meant no failover.
> Fix: **readiness = can *this replica* serve?** (not its dependencies), ran 2 replicas with pod anti-affinity/spread, and let dependency failures surface as 503s that the client or edge can handle."

---

## H7. "Design a distributed cache for a read-heavy / write-heavy service" (classic system design)
Examples interviewers use: product prices, hotel/flight rates, user sessions, feature flags.

**Clarify first:** read vs write rate, latency target (e.g. p99 < 50 ms), data size, how stale is acceptable, what happens on a miss.

**Design:**
1. **Key design:** e.g. `item_id:variant:region`; compact serialized value. Batch reads with **MGET**.
2. **Store:** in-memory **Redis-compatible cluster** (hash slots → add nodes and rebalance without downtime).
3. **Expiry:** write with **TTL** — no separate cleanup job (same idea as my TTL chat).
4. **Miss path:** **cache-aside** — on miss read source, populate cache. **Request coalescing / single-flight** so 1,000 concurrent misses on one key cause one backend call (**stampede** protection); jittered TTLs so keys don't expire together.
5. **Write path:** updates via **Kafka** → consumers write to cache (partition by key for ordering); or write-through for strong freshness.
6. **Availability:** replicas, automatic failover, multi-AZ; clients use timeouts and fall back to the source of truth (LocalArena: Valkey down → read Postgres).
7. **Migrating from an old store:** dual-write / **shadow reads with parity check** (compare old vs new), gradual traffic shift, then decommission.
8. **Observability:** hit ratio, p99 latency, eviction rate, memory, staleness.

**Trade-offs to say out loud:** consistency vs latency, memory cost vs hit ratio, TTL length vs staleness.

---

## H8. "How would you scale TTL Chat to 1 million users?"
1. **Fix known bugs first:** WS auth, exact-match origin allowlist (CSWSH), message TTL, client reconnect with backoff.
2. **Stateless WS gateways** behind a load balancer; move presence/voice roster out of process memory into Redis.
3. **Cross-instance fan-out:** **Redis Pub/Sub** (or Kafka/NATS) — a message published on gateway A reaches subscribers on gateway B.
4. **Shard rooms** with consistent hashing to owner shards; **per-room sequence numbers** for ordering and resumable reconnect.
5. **Media via SFU** + **TURN fleet** with ephemeral credentials; simulcast.
6. **Reconnect storms:** jittered backoff, connection rate limiting.
7. **Memory math:** ~2 goroutines × few KB stack + buffers ≈ tens of KB per connection → 1M connections ≈ tens of GB across a fleet; tune buffers.

---

## H9. "How would you make SolanaPay production-ready?"
- **Verification worker + queue:** devnet RPC limits (~100 req/10 s per IP), so move verification off the request path to a queue with retry/backoff and rate-limited workers.
- **Confirmation level:** wait for `finalized` (not just `confirmed`) before fulfilling high-value orders; handle **expired blockhash** (tx never lands).
- **Webhooks to merchants:** sign payloads with **HMAC** + timestamp (replay protection), retry with exponential backoff, DLQ, idempotent event IDs.
- **Idempotency** on checkout creation; unique constraint on tx signature so one transaction can't pay two checkouts (**double-spend of a receipt**).
- **Security:** rate limiting per API key, key scopes, rotate keys, audit log.
- **Reconciliation job:** compare on-chain transfers vs DB nightly.
- **Mainnet:** multiple RPC providers with failover.

---

## H10. "Why did the throttled client stay throttled forever?" (subtle bug, shows rigor)
> "The lag check compared the client's last processed tick `st` to the **current** server tick. A throttled client only receives 1 snapshot/s, so it can only ever report old ticks — it always looked lagging. Fix: compare `st` to the tick **last sent** to that client. When it catches up to what it was actually sent, throttling releases. Covered by `TestLaggingClientIsThrottled`."

---

## H11. "What did you get wrong in LocalArena and what would you change?"
Pick 2–3 (from the FINAL_REPORT failure table):
- **"No rewards since week 5"** — a change disabled the `?player=` param, bots became guests, rewards silently stopped. Technical metrics were all green. **Lesson:** alert on **business outcomes** (rewards/min), not just CPU and 5xx.
- **False page with no games running** — `1 - 0/ε` evaluated as 100% bad. Guard `total > 0` + regression test for alert rules with promtool.
- **Helm vs Argo CD conflicts / rollback deleted the pooler** — two deployers fighting over resources. **One deployer per app**, `resource-policy: keep` for stateful pieces.
- **Change I'd make:** fewer components earlier — I'd start simpler and add Kafka only when a second consumer actually needs the events.

---

## H12. "Explain the PostgreSQL failover issue (3 minutes then 15 more seconds)."
> "On primary failover the pooler kept a *smart shutdown* waiting for clients to disconnect — 3 minutes. Then PgBouncer cached the login failure for 15 s. Fix: `smartShutdownTimeout: 10` and `server_login_retry: 1` → about **1 s**. Measured replica failover: **RTO 0.7 s**, RPO 0 when healthy; with async replication and a 5 s partition we lost 2.8 s of writes — that's the trade-off of async vs sync replication."

---

## H13. "CAP theorem — where does each of your systems sit?"
> "During a **partition** you choose consistency or availability.
> - **Purchases (PostgreSQL primary):** **CP** — better to reject a purchase than double-spend credits.
> - **Presence/cache (Valkey):** **AP** — slightly stale presence is fine.
> - **Multi-region LocalArena:** each region owns its data; remote writes cost **805 ms vs 35 ms local**, and during an inter-region partition I return **503 instead of 500** for dead pooled connections so clients can retry elsewhere."

---

## H14. "If the interviewer asks about a tech you don't know (Scala, C#, Spark, TeamCity, Cassandra):"
**Template:** *Honest → Closest experience → How you'd ramp up.*
> "I haven't used **Scala** in production. I've worked in **Java** on the JVM at Amazon and written Kafka Streams in Java, so the runtime, build tooling and functional-style stream APIs are familiar. I learned Go from scratch for LocalArena by building and load-testing it — I'd do the same: read the team's code, start with small tickets, pair with someone."

- **Cassandra:** wide-column, partition key → node, tunable consistency (ONE/QUORUM/ALL), great for write-heavy time-series; no joins.
- **Spark:** distributed batch/stream processing; lazy transformations, actions trigger jobs; vs Kafka Streams = library inside your app for per-event stateful stream processing.
- **TeamCity / Puppet:** CI server (like GitHub Actions) / configuration management (like Helm/OpenTofu declaring desired state).
