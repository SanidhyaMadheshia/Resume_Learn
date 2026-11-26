# 04 — Behavioral Questions (STAR format)

**STAR = Situation (1–2 lines) → Task (your goal) → Action (≥ 60% of the answer, say "I") → Result (number) → Learning.**
Keep each story ≤ 2 minutes. One story can answer several questions — map below.

| Question type | Use story |
|---|---|
| Hardest bug / debugging | S1 (deadlock) or S2 (PgBouncer incident) |
| Failure / mistake | S3 (silent reward failure) |
| Learning something fast | S4 (Go + Kafka for LocalArena) / Amazon onboarding |
| Ownership / initiative | S5 (LocalArena end to end) |
| Teamwork / feedback | S6 (Amazon code review) |
| Leadership / mentoring / communication | S7 (Codebase Club) |
| Trade-off / pragmatic decision | S8 (mesh vs SFU) |
| Customer focus | S9 (SolanaPay for Web2 devs) |

---

### S1. Hardest bug — purchase deadlock
- **S:** LocalArena load test: 200 buyers, 20 limited cosmetic items.
- **T:** Correct (no oversell) and fast purchases under contention.
- **A:** I saw **0 of 200** purchases succeed with 344 retries. I read PostgreSQL's lock-mode table and found the purchase insert's foreign key took `FOR KEY SHARE`, which conflicts with my `FOR UPDATE`. I tested three fixes side by side: `FOR NO KEY UPDATE`, optimistic locking, and a single conditional `UPDATE ... WHERE stock > 0`.
- **R:** Conditional update: **129 ms** for all 200 buyers vs 832–984 ms, zero oversell. Added a fixed lock order and jittered retries.
- **Learning:** Measure alternatives instead of guessing; understand the tool one level deeper.

### S2. Debugging under pressure — the paused pool
- **S:** Game-day drill with a hidden fault; ~50% of API requests failing.
- **T:** Detect, diagnose and fix as on-call.
- **A:** Alert paged me; I noticed only primary-DB routes failed; logs showed no SQL errors; traces showed slow HTTP spans **without** SQL child spans; Postgres was idle; PgBouncer showed 30 clients waiting and the pool paused. I resumed it.
- **R:** Diagnosed in **1 m 6 s**, outage 3 m 46 s, zero data lost. Wrote a blameless postmortem with action items (PgBouncer metrics, clearer errors).
- **Learning:** Follow the evidence, use absence of signals, write it down so it's faster next time.

### S3. A failure — rewards silently stopped
- **S:** In week 8 I discovered **no rewards had been granted since week 5**.
- **T:** Find why nothing alerted.
- **A:** An earlier change to game tickets disabled the `?player=` parameter; bots became guests, and guests don't earn rewards. Every technical metric was green. I added an explicit lab flag and a **business-outcome alert** (rewards granted per minute).
- **R:** The gap can't go unnoticed again.
- **Learning:** Monitor what the user/business cares about, not just CPU and error rates. I own the mistake — I didn't have an end-to-end check for it.

### S4. Learning quickly
- **S:** For LocalArena I wanted to learn distributed systems properly, and I had not used Go, Kafka Streams or Kubernetes at depth before.
- **T:** Build something real in 8 weeks, not just tutorials.
- **A:** Planned week by week; each topic followed *learn → design → build → break it → measure → write it down*. 22 ADRs, 8 experiments, 2 postmortems.
- **R:** A working system with measured results (1,000 bots, 0.76 ms p99, 0 failed requests in deploy drills).
- **Learning:** I learn fastest by building and breaking things. **Tie to the role:** "That's how I'd ramp up on your stack."

### S5. Ownership
Use LocalArena: you owned design, code, tests, CI, deploys, on-call drills and documentation. Emphasise **CI** (27 test suites, race detector, security scans) and **zero-downtime** deploys — valued in every backend team.

### S6. Teamwork / receiving feedback (Amazon)
- **S:** My first recordInteraction change at Amazon went through code review.
- **T:** Get it production-ready to team standards.
- **A:** Reviewers pointed out things like validation edge cases, test coverage, and following existing patterns. I asked follow-up questions to understand the *why*, updated the code, and added tests.
- **R:** The change shipped, and later reviews needed fewer rounds because I applied those patterns up front.
- **Learning:** Code review is the fastest way to learn a codebase; ask "why" rather than just fixing.
- *(Adjust details to what actually happened — keep it truthful and non-confidential.)*

### S7. Leadership / communication — Codebase Club
- **S:** Many students in the club wanted to learn blockchain/full-stack but didn't know where to start.
- **T:** As Web3 Lead, get them building, not just watching.
- **A:** Ran **weekly hands-on sessions** for 15+ members: each session ended with something deployed (a smart contract, a dApp page, a Git PR). Paired beginners with stronger members, reviewed their PRs.
- **R:** Mentored **30+ developers**; several shipped their first dApps/contracts.
- **Learning:** Explaining something simply is the best test of whether I understand it.

### S8. Pragmatic trade-off — WebRTC mesh vs SFU
- **S/T:** Add voice/video to TTL Chat with no budget for media servers.
- **A:** Chose **peer-to-peer mesh** (no server media cost) and **capped rooms at 8**, since mesh upload grows with n−1 peers. Server only relays signaling. Documented the SFU path for later.
- **R:** Shipped working voice/video fast and cheaply, with a clear scaling plan.
- **Learning:** Pick the simplest design that meets today's need, and write down when to change it.

### S9. Customer focus — SolanaPay
- **S:** Web2 developers find crypto payments intimidating (wallets, keys, chain details).
- **A:** Designed it to feel like Stripe: products/prices, a checkout API, hosted checkout, webhooks. Merchants never touch blockchain code.
- **R:** Live demo with a full checkout flow on devnet.
- **Learning:** Good APIs hide complexity behind familiar concepts.

---

## Common closing questions

**"Why should we hire you?"**
> "I already work on a high-TPS Java service at Amazon, I've built fault-tolerant event-driven systems with Kafka and PostgreSQL and proven them with measurements, I care about testing, CI/CD and zero-downtime releases, and I've shipped LLM features in production. And I learn fast — LocalArena shows exactly how."

**"Where do you see yourself in 3 years?"**
> "As a strong backend engineer who owns a service end to end — design, on-call, scaling — and mentors newer engineers."

**"Your weakness?"** (real + improving)
> "I tend to go deep and over-build — LocalArena had more components than it strictly needed. I now start with the simplest version, set a time box, and add complexity only when a measurement says I need it."

**"Do you have questions for us?"** (always ask 2)
1. "What does a typical intern/new-grad project look like on this team, and how is success measured?"
2. "How does the team handle on-call and incidents — are there postmortems?"
3. "What's the biggest technical challenge the team is facing this year?"
4. "How do you use AI tools in day-to-day engineering?"
