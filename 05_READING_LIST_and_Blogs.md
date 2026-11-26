# 05 — Reading List

---

## ⭐ Must-read blog (~15 min)

### "Designing robust and predictable APIs with idempotency" — Brandur Leach, Stripe (2017)
**Link:** https://stripe.com/blog/idempotency

**Why this one:** idempotency is the single concept that runs through your **whole** resume — Amazon's recordInteraction + SQS at-least-once delivery, Inbotiq webhook retries, LocalArena's Idempotency-Key purchases and 1.45M replayed events, and SolanaPay (literally "Stripe for Solana"). Interviewers love asking about it.

**Key points:**
- **The problem:** networks fail. When a request times out, the client **doesn't know** whether the server did the work. Did the payment go through or not?
- **Idempotency keys:** the client generates a unique ID per operation and sends it (Stripe: `Idempotency-Key` header on POST). If a retry carries the same key, the server recognises it and returns the original result instead of doing the work again → no double charge.
- **Retry until verified:** safe retries + idempotency let the client converge to the same state as the server.
- **Exponential backoff:** wait progressively longer after each failure, so you don't hammer a struggling server.
- **Jitter:** add randomness to the wait so thousands of clients don't retry at the same instant (**thundering herd**).

**How to use it in an interview:**
> "I used exactly this pattern in LocalArena. A purchase carries an `Idempotency-Key`; inside the same transaction I `INSERT (player_id, key) ON CONFLICT DO NOTHING`. A concurrent duplicate blocks on the unique index, then replays the original response; the same key with a different item returns 422. Deadlock retries use exponential backoff **with jitter**, because without jitter all the victims retry in lockstep and collide again. I proved it by replaying 1.45M duplicate events — zero duplicate rewards."

**Follow-up questions it prepares you for:**
- "How do you make a POST endpoint safe to retry?"
- "What happens if the client retries while the first request is still running?" (→ unique constraint / lock on the key; second waits then replays)
- "How long do you keep idempotency keys?" (→ TTL, e.g. 24 h — Stripe expires them)
- "Why jitter?"

---

## Optional case study (~15 min) — caching at scale
### "How Agoda Scaled Its Price Cache with DragonflyDB" — Agoda Engineering
**Link:** https://medium.com/agoda-engineering/how-agoda-scaled-its-price-cache-with-dragonflydb-0828e478e84f

A real-world answer to the "design a distributed cache" question (`03_HARD` H7):
- ~300K reads/s, ~1.5M writes/s, p99 < 50 ms, ~1.5 TB in memory.
- Moved off a 72-shard SQL setup that could only scale in rigid steps → Redis-compatible cluster that adds nodes and moves hash slots.
- **Built-in TTL** removed a custom cleanup job (same reason you used Redis TTL in your chat app).
- Migration: populate → **parity check** (read both, compare) → gradual rollout → automatic failover.

---

## Your own repo docs (best revision material — you wrote them)

| What | Link | Priority |
|---|---|---|
| LocalArena final report (all numbers + failures table) | https://github.com/SanidhyaMadheshia/localArena/blob/main/FINAL_REPORT.md | **Must** |
| LocalArena ADR deep dives — each ends with **interview Q&A** | https://github.com/SanidhyaMadheshia/localArena/tree/main/docs | ADR-0003, 0006, 0009, 0010, 0011, 0017, 0019 |
| LocalArena postmortem PM-001 (PgBouncer) | https://github.com/SanidhyaMadheshia/localArena/blob/main/postmortems/PM-001-pgbouncer-paused.md | **Must** |
| TTL Chat interview guide (60+ Q&A) | https://github.com/SanidhyaMadheshia/ttl-based-chats/blob/main/docs/06-interview-guide.md | **Must** |
| TTL Chat scaling to millions | https://github.com/SanidhyaMadheshia/ttl-based-chats/blob/main/docs/05-scaling-to-millions.md | Recommended |

---

## Fundamentals (if time permits)
- *Designing Data-Intensive Applications* (Martin Kleppmann) — ch. 5 Replication, ch. 7 Transactions, ch. 11 Stream Processing.
- PostgreSQL docs — "Explicit Locking" (row-level lock modes table): https://www.postgresql.org/docs/current/explicit-locking.html — explains your deadlock story.
