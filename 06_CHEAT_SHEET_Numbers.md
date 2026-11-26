# 06 — Cheat Sheet: Every Number on Your Resume

Interviewers **will** pick a number and ask "how did you measure that?" Know each one.

## Work experience
| Number | Meaning | How to explain |
|---|---|---|
| **4,400 TPS** | Amazon recommendation service throughput | "Transactions per second the service handles; ≈ 380M/day if sustained." Don't share more internal metrics. |
| **3 messaging systems** | Kafka, AWS SNS, AWS SQS | SNS = push fan-out, SQS = pull queue w/ DLQ, Kafka = replayable log |
| **100+ calls/day** | Inbotiq voice bot production volume | Real production traffic |
| **3 AI stages** | STT → LLM → TTS per call | + VAD in front to detect speech |

## LocalArena
| Number | Meaning | Source / how measured |
|---|---|---|
| **20 Hz** | Fixed tick = every 50 ms | Deterministic simulation budget |
| **1,000 bots / 50 matches** | Load test: Go bot swarm, 20 players per match | Experiment 001 |
| **0.76 ms p99** (p50 0.07, p95 0.21) | Tick duration at 1,000 players | 1.5% of the 50 ms budget, 4 cores |
| Saturation | 2,000 bots / 200 Hz / 1 CPU | Bottleneck was **fan-out** (sending snapshots), not simulation |
| **1.45M** replayed events, **0** duplicate rewards | Idempotency proof | Exp 002/004 — deterministic IDs + dedupe + unique constraints |
| **0 lost events**, 60 s Kafka outage | Outbox + spool | Ticks unaffected |
| **0 of 90,000** failed | Rolling, canary, blue-green, rollback at 600 req/s with HPA 2→5→2 | Exp 007, kind cluster |
| **129 ms** | 200 buyers / 20 items with conditional update (vs 832–984 ms) | Deadlock story |
| **0.7 s RTO** | Async replica failover | RPO 0 healthy; 2.8 s writes lost after 5 s partition |
| **7,482** retries | Retry storm in DB brownout (before fix) | Timeout-ordering story |
| **11** failures fixed | Rows in FINAL_REPORT "what failed" table | Weeks 1–8 |
| **22 ADRs, 8 experiments, 2 postmortems, 5 runbooks** | Documentation | |
| **27** Go test suites | `_test.go` files run in CI with `-race` | GitHub Actions |
| **805 ms vs 35 ms** | Remote vs local region write | Multi-region week |
| p99 **8 ms** at 300 req/s | API through Envoy | |
| 0.8 ms | Indexed match-history query | |

## TTL Chat
| Number | Meaning |
|---|---|
| **2 goroutines per connection** | Reader (handlers) + writer (only socket writer) |
| **O(1)** | Redis HASH/LIST/SET ops used |
| **Up to 8 users** | Mesh cap (upload grows n−1 per user) |
| 640×360 @ ≤30 fps, ~500 kbps/peer | Video cap |
| **79** Mermaid sequence diagrams | Docs |

## SolanaPay
| Number | Meaning |
|---|---|
| **1 atomic transaction** | Merchant payment + platform fee (0.00001 SOL) in one tx |
| **SHA-256** | API keys stored hashed, prefix `sk_`, Bearer auth, revocable |
| Devnet RPC limit | ~100 req / 10 s per IP → why a verification queue is next |

## Leadership
| **30+** developers mentored · **15+** members in weekly sessions |

---

## 10-minute pre-interview checklist
- [ ] Say "Tell me about yourself" once out loud (90 s)
- [ ] Re-read H5 (PgBouncer incident) and S1 (deadlock) — your two best stories
- [ ] 1–2 facts from the company's engineering blog / product
- [ ] 2 questions to ask them
- [ ] Water, resume printout, laptop charged (on-site) / camera + mic tested (virtual)
