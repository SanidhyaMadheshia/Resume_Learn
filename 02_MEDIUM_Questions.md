# 02 — MEDIUM Questions (design choices, trade-offs, debugging)

Answer structure: **Decision → Why → Alternative I rejected → Trade-off → Evidence/number.**

---

## A. Amazon (keep it high level — no internals)

### M1. "How would you design recordInteraction to handle 4,400 TPS?"
**Model answer:**
1. **Keep the synchronous path thin:** validate input, assign/accept an idempotency or interaction ID, publish the event, return `202 Accepted`/`200`. No heavy work in the request.
2. **Async processing:** publish to **SNS → SQS** (or Kafka); consumers do enrichment, storage, and feed recommendations.
3. **Idempotency:** clients and SQS can deliver twice, so consumers dedupe on the interaction ID.
4. **Failure handling:** retries with backoff, a **dead-letter queue (DLQ)** for poison messages, alarms on DLQ depth and queue age.
5. **Scale:** stateless service behind a load balancer → horizontal scaling; consumers scale on queue depth.
6. **Observability:** p99 latency, error rate, queue age, DLQ count dashboards.

> "That's the general pattern; I can't share our internal specifics, but this is how I reason about it."

### M2. "Why make the event flow asynchronous instead of writing directly to the DB in the request?"
> "Three reasons: **latency** (the user doesn't wait for downstream work), **isolation** (if a downstream system is slow or down, the API still succeeds — the queue buffers), and **decoupling** (new consumers can subscribe without changing the API). The cost is **eventual consistency** and needing idempotent consumers."

### M3. "What happens if an SQS consumer crashes mid-message?"
> "SQS uses a **visibility timeout**: the message is hidden while being processed; if the consumer doesn't delete it before the timeout, it becomes visible again and another consumer picks it up. That's **at-least-once** delivery — so the consumer must be idempotent. After N failed receives (`maxReceiveCount`), the message moves to the **DLQ**."

### M4. "How do you test a Spring Boot service?"
> "**Unit tests** with JUnit + Mockito for service logic (mock the repository/clients). **Integration tests** with the Spring context (`@SpringBootTest`) and test containers or local stubs for the DB/queues. Contract tests for API shape. All run in the CI pipeline before deploy."

### M5. "What did you learn about working in a big codebase?"
> "Read before writing — understand existing patterns and follow them; small, reviewable changes; tests with every change; and code review is where you learn the most. I'd rather ask a good question early than make a wrong assumption."

---

## B. Inbotiq

### M6. "Walk me through one call end to end."
1. Telephony provider receives the call → **webhook** `call.started` → FastAPI creates a session row in **PostgreSQL**.
2. Audio streams in → **Silero VAD** detects speech start/end.
3. On end-of-speech → **STT** → text.
4. Text + conversation history (from session state) → **LLM** → response text.
5. **TTS** → audio streamed back to the caller.
6. Loop 2–5 until hang-up → webhook `call.ended` → finalize the session.

### M7. "Where is the latency in a voice bot and how do you reduce it?"
> "STT + LLM + TTS add up, and humans notice > ~1 s gaps. Techniques: **streaming** STT and TTS (start speaking before the full response is ready), stream LLM tokens, keep a short prompt/history, keep connections warm, and use VAD to detect end-of-speech quickly instead of waiting for a long silence."

### M8. "Why store session state in PostgreSQL rather than in memory?"
> "In memory is lost if the process restarts or the next webhook hits a different instance. PostgreSQL makes state **durable and shared**, so any instance can handle the next event. If latency became an issue, I'd put Redis in front for hot session state, keeping Postgres as the source of truth."

### M9. "Webhook providers retry. How did you avoid processing an event twice?"
> "Use the provider's event ID as an idempotency key — store it with a unique constraint; if the insert conflicts, it's a duplicate and we return 200 without reprocessing. Also return 2xx quickly, then process, so the provider doesn't retry unnecessarily."

---

## C. LocalArena

### M10. "What does server-authoritative mean and why?"
> "The server owns position, collisions, score and inventory; clients only send **inputs** (move left, shoot). It prevents cheating — a client can't claim it teleported or scored. The cost is the server must simulate everything, so tick performance matters — which is why I measured p99 tick time."

### M11. "Why a 20 Hz fixed tick?"
> "Every 50 ms the server applies queued inputs and advances the world deterministically. A **fixed** tick makes simulation **deterministic** and replayable (same inputs → same result), and gives a clear budget: each tick must finish in 50 ms. At 1,000 players p99 was 0.76 ms — about 1.5% of the budget."

### M12. "How did you handle slow clients?"
> "Each match is an **actor** — one goroutine owns the world, so no locks. Each client has a **capacity-1, latest-wins** mailbox: a new snapshot replaces an unread one, because old game state is useless. Then I found a surprise: Linux socket buffers absorbed hundreds of snapshots, so clients were seconds behind while the server thought they were fine. Fix: the client reports the last tick it processed; if it's > 1 s behind, the server **throttles it to 1 snapshot/s** instead of disconnecting it — because a disconnected client can never prove it caught up."

### M13. "Why Kafka here? Isn't it overkill for a game?"
> "Kafka is **not in the gameplay hot path** — inputs and snapshots go over WebSocket directly. Kafka carries **durable events** after the server decides what happened: match completed, rewards, purchases. That gives replay, multiple independent consumers (leaderboard, anti-cheat, sessionizer), and decouples the game server from analytics. The tick never waits on Kafka — a bounded background producer batches events."

### M14. "What is the dual-write problem and how did you solve it?"
> "If you write to PostgreSQL and then publish to Kafka, a crash in between leaves them inconsistent — lost or phantom events. **Transactional outbox:** write the event as a row in the **same DB transaction** as the state change. **Debezium CDC** reads the committed rows from Postgres's WAL and publishes them to Kafka in commit order. Event exists if and only if the commit happened."

### M15. "Debezium is at-least-once. How did you get zero duplicate rewards?"
> "Not by pretending delivery is exactly-once. Every event has a **deterministic `event_id`** and reward IDs are deterministic too; consumers **dedupe** by ID, and the DB has unique constraints so a duplicate insert is a no-op. I proved it by replaying **1.45M duplicate events** — the totals didn't change."

### M16. "Tell me about a bug you found in LocalArena." (great story — use STAR)
**S:** Load test of 200 buyers purchasing 20 limited items.
**T:** Purchases had to be correct (no oversell) and fast.
**A:** With `SELECT ... FOR UPDATE`, **every** buyer deadlocked — 0 of 200 succeeded. Root cause: inserting the purchase row takes a `FOR KEY SHARE` lock on the item via the foreign key; `FOR UPDATE` conflicts with that, so all buyers waited on each other. I switched to `FOR NO KEY UPDATE`, and then to a **conditional update** (`UPDATE ... SET stock = stock - 1 WHERE stock > 0`), which folds the check into the write.
**R:** **129 ms** for all 200 buyers vs 832–984 ms for the locking versions, zero oversell, plus a fixed global lock order and jittered retry for the remaining deadlock cases.
**Learning:** Know your database's lock modes; the fastest correct solution is often one atomic statement.

### M17. "How did you do zero-downtime deploys?"
> "Kubernetes rolling updates with **readiness probes** (a pod gets traffic only when ready), **graceful shutdown** (drain in-flight requests on SIGTERM), Envoy health checks, and a canary/blue-green strategy via Helm and Argo CD. Measured: rolling, canary, blue-green and rollback — **0 of 90,000 requests failed**. One lesson: readiness must check **only this replica**, not the DB — otherwise a DB blip marks every pod unready at once."

### M18. "Optimistic vs pessimistic locking — which did you use?"
> "Both, by use case. **Optimistic** (version column, `UPDATE ... WHERE version = $read`, retry on 0 rows) for **lobbies**, where conflicts are rare. **Pessimistic/atomic** for **purchases** under heavy contention — optimistic needed 206 retries there, the conditional update needed none."

### M19. "What's in your CI pipeline?"
> "GitHub Actions: `go vet` + `gofmt`, **27 Go test suites** with the **race detector** and integration tests, Java tests via Gradle for the Kafka Streams apps, **gosec** and **govulncheck** for security, OPA policy tests, Prometheus alert-rule tests with promtool, **Helm lint**, **OpenTofu validate**, schema compatibility checks, and **Trivy** image scanning that fails on HIGH/CRITICAL."

---

## D. SolanaPay

### M20. "How does on-chain verification work?"
> "After the customer signs and sends the transaction, we take its **signature/hash**, fetch the parsed transaction from a Solana RPC node, and check: it's **confirmed/finalized**, the **recipient** is the merchant's wallet, the **amount** in lamports matches the price, and the platform-fee transfer is present. Only then do we mark the checkout paid and fire the webhook. Never trust the client saying 'I paid'."

### M21. "Why bundle payment and fee into one transaction?"
> "A Solana transaction is **atomic** — all instructions succeed or none do. Putting the merchant payment and platform fee in one transaction means we can never end up with the merchant paid but the fee missing, or vice versa."

### M22. "How do you store API keys securely?"
> "Never store the raw key. We show the key (`sk_...`) to the merchant **once**, store only its **SHA-256 hash**. On each request, hash the Bearer token and look up a non-revoked match. If the DB leaks, keys can't be used. Revocation is a flag. (A fast hash is OK here because API keys are long and random, unlike passwords, which need bcrypt/argon2.)"

### M23. "What is idempotency in a payments API and how would you implement it?"
> "Client sends an `Idempotency-Key` header with 'create checkout'. Server stores key → response with a unique constraint. Retry with the same key → return the stored response; same key with a different body → 422. That's exactly how Stripe does it."

### M24. "Why Socket.io and webhooks — aren't they the same?"
> "Different audiences. **Socket.io** pushes real-time status to the **customer's browser** on the checkout page. **Webhooks** notify the **merchant's server** so they can fulfil the order. The webhook is the source of truth for the merchant; the socket is UX."

---

## E. TTL Chat

### M25. "How does the hub pattern work?"
> "Each connection has a **reader goroutine** (reads frames, runs handlers) and a **writer goroutine** (the only one that writes to the socket — gorilla/websocket allows one concurrent writer). A central **hub goroutine** handles register/unregister/broadcast over channels. Broadcast does a **non-blocking send** to each client's buffered channel; if it's full, the client is a **slow consumer** and gets evicted so it can't stall everyone."

### M26. "Why Redis as the only datastore?"
> "The product is ephemeral by design. Redis gives **O(1)** hash/list/set operations, in-memory speed, and **built-in TTL**, so expiry is enforced by the datastore instead of a cleanup job. Room metadata is a HASH, messages a LIST, members a SET, all with `EXPIRE`."

### M27. "Mesh vs SFU for WebRTC?"
> "**Mesh**: each peer connects to every other — no media server, but each user uploads n−1 streams, so it's O(n²) total; fine up to ~8. **SFU**: everyone sends one stream to a server that forwards it — scales to large rooms, costs server bandwidth. I chose mesh with a cap of 8; the scaling roadmap moves to an SFU (LiveKit/mediasoup/Pion)."

### M28. "What are STUN and TURN?"
> "**STUN** tells a peer its public IP/port so peers behind NAT can connect directly. **TURN** relays the media when direct connection fails (strict/symmetric NAT). ICE tries candidates in order: host → STUN-reflexive → TURN relay."

---

## F. CS fundamentals (medium)

### M29. "How does a HashMap work? What's the complexity?"
> "Array of buckets; key's `hashCode()` → bucket index; collisions handled by a linked list, and in Java 8+ a bucket turns into a **red-black tree** after 8 entries. Average O(1) get/put, worst O(log n) in Java 8+. Resizes (doubles) at load factor 0.75. Keys must implement `equals` and `hashCode` consistently."

### M30. "Explain database isolation levels."
| Level | Prevents | Allows |
|---|---|---|
| Read Uncommitted | — | dirty reads |
| Read Committed (Postgres default) | dirty reads | non-repeatable reads, phantoms |
| Repeatable Read | + non-repeatable reads | (Postgres also blocks phantoms) |
| Serializable | everything | — (may abort with serialization failure) |

### M31. "What is caching and what are the invalidation strategies?"
> "**Cache-aside** (app reads cache, on miss reads DB and populates), **write-through** (write cache + DB together), **write-behind** (write cache, flush later). Invalidate with **TTL** and/or delete-on-write. Watch for **stampede** on popular keys (use locking/request coalescing or jittered TTLs). In LocalArena, if Valkey goes down reads fall back to Postgres."

### M32. "What is a race condition? How do you detect one in Go?"
> "Two goroutines access shared data concurrently and at least one writes, without synchronization. Fix with a mutex, channels, or single ownership (the actor pattern). Detect with `go test -race` — LocalArena's CI runs every test with the race detector."

### M33. "Horizontal vs vertical scaling?"
> "Vertical = bigger machine (simple, has a ceiling). Horizontal = more machines (needs stateless services, load balancing, and data partitioning/sharding). Rigid sharding schemes are painful — you can only grow in fixed steps and must remap data — which is why consistent hashing or hash-slot clusters (Redis Cluster) are popular: add a node and move only some keys. In LocalArena I used Jump Consistent Hash: going from 2→3 shards moved only ~33% of keys, all to the new shard."
