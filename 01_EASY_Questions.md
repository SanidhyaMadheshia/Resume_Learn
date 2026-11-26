# 01 — EASY Questions (must be fluent, no hesitation)

Format for every answer: **What they test → Model answer → Likely follow-up.**
Speak each answer out loud. Target: 60–120 seconds.

---

## A. Introduction

### E1. "Tell me about yourself."
**Tests:** communication, clarity, relevance to the role.

**Model answer (≈ 90 s) — Present → Past → Future:**
> "I'm Sanidhya, a final-year Computer Science student at IIIT Kota. **Right now** I'm a Software Development Intern at Amazon on the Alexa Smart Home team, where I work on a recommendation-serving backend in Java and Spring Boot that runs at around 4,400 TPS. I built the `recordInteraction` API, which captures how users interact with recommended content, and I worked on the Kafka and SNS/SQS event flows behind it.
>
> **Before that**, at Inbotiq, I built Python FastAPI services for a production AI voice bot that handles 100+ calls a day — wiring together speech-to-text, an LLM, and text-to-speech, and improving reliability with voice activity detection.
>
> **On the side**, I like building systems end to end. My biggest project is LocalArena, a multiplayer game backend in Go with Kafka, PostgreSQL and Kubernetes, where I load-tested 1,000 concurrent bots at 0.76 ms p99 and ran zero-downtime deployment drills.
>
> **I'm looking for a backend role** because that's exactly what I enjoy — scalable, fault-tolerant APIs, event-driven systems with Kafka, PostgreSQL, and CI/CD — and I'd love to do it at [company]'s scale."

**Follow-up:** "Which of these are you most proud of?" → LocalArena (you designed it end to end, measured everything, wrote postmortems).

---

### E2. "Walk me through your resume."
Same as E1 but chronological, 1 sentence per item, end with "Happy to go deeper into any of these." **Let them choose** — steer toward LocalArena or Amazon.

---

### E3. "Why do you want to join us?" (template — fill per company)
**Before the interview:** read 1–2 posts from the company's engineering blog and their product page.
> "Three reasons. **First, scale/problem:** [one specific technical problem they have — e.g. 'serving X requests/s with low latency', from their blog]. That's the kind of problem I want to work on. **Second, stack overlap:** [their stack ∩ yours — Java/Spring Boot, Go, Kafka, PostgreSQL, Redis, Kubernetes, CI/CD]. **Third, product/impact:** [why their product matters to users / to you]."

**Rule:** always cite one concrete thing you read about them. Generic answers ("great culture") score low.

---

### E4. "Why backend?"
> "I like problems where correctness and scale matter at the same time — what happens when a message is delivered twice, or a database fails over mid-request. Backend is where those questions live. LocalArena was me chasing exactly that."

---

## B. Amazon — Alexa Smart Home

### E5. "What did you do at Amazon?"
> "I'm on the Alexa Smart Home team, working on a recommendation-serving service in **Java with Spring Boot** that handles roughly **4,400 TPS**. Three things: (1) I worked on the backend logic that serves recommendations to Smart Home devices; (2) I built the **recordInteraction** API, which captures, validates and processes user interactions with those recommendations; (3) I integrated publish/subscribe flows with **Kafka and AWS SNS/SQS** so interaction events are processed asynchronously and downstream consumers are decoupled."

**Rule:** don't name internal services or share numbers beyond what's on your resume. Say *"at a high level…"*.

### E6. "What is TPS? Is 4,400 TPS a lot?"
> "Transactions per second — requests the service handles each second. 4,400 TPS is about 380 million requests a day if sustained. At that rate, small inefficiencies matter: an extra 10 ms per request or one extra DB call becomes a real cost, so you care about latency, caching and keeping the hot path lean."

### E7. "What does recordInteraction do?"
> "When a user interacts with recommended content — for example, sees it, taps it, or dismisses it — the device or client calls recordInteraction. The API **validates** the request (required fields, valid IDs), **records** it, and **publishes an event** so other systems can react asynchronously, for example to improve future recommendations. Keeping the API thin and pushing heavy work to async consumers keeps latency low at high TPS."

### E8. "What is the difference between SNS and SQS?"
| | SNS | SQS |
|---|---|---|
| Model | Pub/sub — **push** to many subscribers | Queue — consumers **pull** |
| Fan-out | One message → many subscribers | One message → one consumer (per queue) |
| Retention | No storage (delivers immediately) | Stores messages up to 14 days |
| Typical use | Broadcast an event | Buffer work, retry, smooth spikes |

> "The common pattern is **SNS → multiple SQS queues** (fan-out): one event, each downstream team gets its own queue, so a slow consumer doesn't affect others."

### E9. "Kafka vs SQS?"
> "Kafka is a **distributed log**: messages are retained, consumers track their own offset, you can **replay**, and ordering is guaranteed **per partition**. SQS is a **queue**: once a message is consumed and deleted it's gone; standard queues don't guarantee order (FIFO queues do, with lower throughput). Use Kafka for streams, replay and multiple independent consumers; SQS for simple managed work queues."

---

## C. Inbotiq

### E10. "What did you build at Inbotiq?"
> "Backend services in **Python/FastAPI** for a production AI voice bot handling **100+ calls a day**. Each call goes through **3 AI stages**: speech-to-text converts the caller's voice to text, an LLM generates the response, and text-to-speech speaks it back. The **call lifecycle** — started, ringing, answered, ended — comes in as **webhooks** from the telephony provider, and I handled those events and kept session state in **PostgreSQL**. I also added **Silero VAD** to improve reliability."

### E11. "What is VAD and why did you need it?"
> "Voice Activity Detection decides whether a slice of audio contains human speech or just silence/noise. Without it, the bot either cuts the user off or waits too long, and background noise gets sent to STT. With Silero VAD we detect when the user actually starts and stops speaking, so turn-taking is more natural and we don't waste STT/LLM calls on noise."

### E12. "What is a webhook?"
> "An HTTP callback: instead of me polling the provider 'has the call ended?', the provider **POSTs** to my endpoint when the event happens. Key practices: verify the **signature**, respond **quickly** (2xx) and process asynchronously, and make handling **idempotent** because providers retry."

---

## D. Projects — 30-second pitches

### E13. LocalArena
> "A server-authoritative multiplayer game backend in **Go**. Clients send only inputs; the server runs a 20 Hz fixed tick and owns all state. Around it I built a production-style backend: PostgreSQL with PgBouncer, Valkey for cache/presence, **Kafka** with a transactional outbox and Debezium CDC, **Kafka Streams** in Java for leaderboards, and deployment on **Kubernetes** with Helm and Argo CD. I load-tested **1,000 bots across 50 matches at 0.76 ms p99**, replayed **1.45M duplicate events with zero duplicate rewards**, and ran canary/rollback drills with **0 of 90,000 requests failed**."

### E14. SolanaPay
> "Stripe-like payments for Solana: a merchant creates products and prices, calls our **REST API** to create a checkout session, redirects the customer to our hosted checkout, the customer pays from their wallet, and we **verify the transaction on-chain** and notify the merchant via **webhooks** and real-time **Socket.io** updates. Backend is TypeScript/Express/Prisma/PostgreSQL/Redis; API keys are stored as SHA-256 hashes."

### E15. TTL Chat
> "Ephemeral chat rooms that delete themselves. Go WebSocket server, **Redis as the only datastore** with TTLs, so when the timer expires the messages, members and room are gone. I added **peer-to-peer WebRTC** voice/video for up to 8 users; the server only relays signaling, never media."

---

## E. CS fundamentals (asked in almost every backend interview)

### E16. "Four pillars of OOP?"
- **Encapsulation** — hide state behind methods (private fields + getters/validation).
- **Abstraction** — expose *what*, hide *how* (interfaces).
- **Inheritance** — reuse via "is-a" (prefer composition when possible).
- **Polymorphism** — same call, different behavior (overriding / interface implementations).

**Example from your work:** "In Spring Boot, a service depends on a `Repository` **interface**; the implementation can be swapped (DB vs mock in tests) — that's abstraction + polymorphism + dependency injection."

### E17. "What does SOLID stand for?"
- **S**ingle Responsibility — one reason to change.
- **O**pen/Closed — extend without modifying.
- **L**iskov Substitution — subtypes usable wherever the base type is.
- **I**nterface Segregation — small, focused interfaces.
- **D**ependency Inversion — depend on abstractions, not concretions (Spring DI).

### E18. "SQL vs NoSQL?"
> "SQL (PostgreSQL) — fixed schema, joins, **ACID transactions**; great for money, inventory, bookings. NoSQL (Redis, MongoDB, Cassandra) — flexible schema, horizontal scale, specialised access patterns. I used **PostgreSQL** for purchases/credits in LocalArena and SolanaPay because they need transactions, and **Redis** for TTL chat and caching because it's in-memory, fast and has built-in expiry."

### E19. "What is ACID?"
**Atomicity** (all or nothing), **Consistency** (constraints hold), **Isolation** (concurrent transactions don't see each other's partial work), **Durability** (committed = survives crash).

### E20. "What is an index? When is it bad?"
> "A data structure (usually a B-tree) that lets the DB find rows without scanning the table. Downsides: extra storage and slower writes because every insert/update must update the index. In LocalArena, an index on match history made the query 0.8 ms."

### E21. "What is a REST API? GET vs POST vs PUT vs PATCH?"
- GET read (safe, idempotent) · POST create (not idempotent) · PUT replace (idempotent) · PATCH partial update · DELETE remove (idempotent).
- Status codes: 200 OK, 201 Created, 400 Bad Request, 401 Unauthorized, 403 Forbidden, 404 Not Found, 409 Conflict, 422 Unprocessable, 429 Too Many Requests, 500, 503.

### E22. "What is idempotency?"
> "Doing the operation twice has the same effect as doing it once. Critical for payments: if the client retries a 'charge' because of a timeout, you must not charge twice. In LocalArena, a purchase carries an **Idempotency-Key**; the server stores it with a unique constraint, so a retry returns the original result instead of buying again."

### E23. "What is CI/CD?"
> "**CI** — every push builds and runs tests automatically. **CD** — passing builds are deployed automatically (or with one click). In LocalArena, **GitHub Actions** runs 27 Go test suites with the race detector, Java tests, security scans (gosec, govulncheck, Trivy), Helm lint and Terraform validation; **Argo CD** deploys from Git (GitOps)."

### E24. "Process vs thread? Goroutine?"
> "A process has its own memory space; threads share the process's memory. A **goroutine** is a lightweight thread managed by the Go runtime — starts with ~2 KB stack, multiplexed onto OS threads, so you can run hundreds of thousands. In TTL Chat each WebSocket connection uses 2 goroutines (reader + writer)."

### E25. "What is a WebSocket? vs HTTP?"
> "HTTP is request/response — the client must ask. A WebSocket starts as an HTTP request with an `Upgrade` header, then becomes a **persistent, full-duplex** TCP connection so the server can push anytime. Ideal for chat and games."
