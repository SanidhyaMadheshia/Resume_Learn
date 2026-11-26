# Curated Tech Blogs — Backend · LLM Serving · Inference Engineering

Short list on purpose: **9 posts**, each one you can talk about for 1–2 minutes. Every link was checked on 25 Nov 2026.
For each post: **what it's about → 3 facts to remember → how to bring it up in an interview.**

**How to use in an interview:** don't recite. Drop it naturally when relevant:
> "That reminds me of a post from [company] engineering where they… — I liked it because… In my project I did something similar…"

| # | Company | Post | Topic | Date |
|---|---|---|---|---|
| 1 | **Eternal (Zomato)** | Building Infra AI Core: the platform behind Eternal's SRE agents | LLM agents + infra | Oct 2026 |
| 2 | **Eternal (Zomato)** | Building Gavel: AI evaluation infrastructure across Eternal | LLM evals in production | Jul 2026 |
| 3 | **Eternal (Zomato)** | Eliminating Bottlenecks in Real-Time Data Streaming: A Zomato Ads Flink Journey | Backend / streaming | Apr 2025 |
| 4 | **Google Research** | Looking back at speculative decoding | Inference engineering | Dec 2024 |
| 5 | **vLLM** | vLLM: Easy, Fast, and Cheap LLM Serving with PagedAttention | LLM serving (foundational) | Jun 2023 |
| 6 | **NVIDIA** | When One GPU Isn't Enough: Serving GLM-5.3 Across Two NVIDIA DGX Stations with vLLM | Distributed LLM serving | Nov 2026 |
| 7 | **Netflix** | Integrating Netflix's Foundation Model into Personalization applications | ML serving + recsys | Oct 2026 |
| 8 | **Amazon (Builders' Library)** | Timeouts, retries, and backoff with jitter | Backend reliability (classic) | Dec 2019 |
| 9 | **Stripe** | Designing robust and predictable APIs with idempotency | Backend APIs (classic) | Feb 2017 |

---

## 1. Eternal — Building Infra AI Core: the platform behind Eternal's SRE agents
**Link:** https://www.zomato.com/blog/building-infra-ai-core-the-platform-behind-eternals-sre-agents (Akshit Sehgal, 29 Oct 2026)

**What:** Eternal's SRE team built **one AI layer** between agents and their infrastructure (instead of a chatbot per problem), living in Slack, incident threads and editors. Agents triage alerts, explain cost anomalies, draft readiness reviews; engineers review and decide.

**Remember:**
- **"The model was never the hard part."** Reasoning was good from day one; the bottleneck was **safe, fast access to context** — logs, metrics, changelog, owners.
- "Early answers were **articulate and wrong**" — because the model couldn't see, e.g., the config change from 20 minutes before.
- Every tool call goes through **one audited layer** — safety + access control.

**Interview hook:**
> "It matches what I saw debugging my own incident in LocalArena: the decisive signal was a trace with no SQL span and a PgBouncer `SHOW POOLS` — context, not cleverness. An agent is only as good as the telemetry it can reach."

---

## 2. Eternal — Building Gavel: AI evaluation infrastructure
**Link:** https://www.zomato.com/blog/building-gavel-ai-evaluation-infrastructure-across-eternal (Anshuman Mishra, 9 Jul 2026)

**What:** In-house evaluation platform for AI products across Zomato, Blinkit and District — images, chatbot transcripts, voicebot calls.

**Remember:**
- Gen AI moved the bottleneck from **producing** output to **knowing if it's good**; humans can't QC at that scale (first project: ~35,000 generated images).
- **Over a million items** evaluated in six months.
- Most important lesson: **"look at your data"** — evals are a discipline, started as scripts → SDK → central platform.

**Interview hook (great for your Inbotiq voice bot):**
> "At Inbotiq we ran a voice bot with STT → LLM → TTS on 100+ calls a day. The hard part wasn't calling the LLM, it was knowing when a call went badly. Eternal's Gavel post argues exactly that evals are infrastructure, not an afterthought — I'd add transcript-level evals as the first step."

---

## 3. Eternal — Zomato Ads Flink Journey
**Link:** https://www.zomato.com/blog/eliminating-bottlenecks-in-real-time-data-streaming-a-zomato-ads-flink-journey (Zomato Engineering, 22 Apr 2025)

**What:** Real-time ads feedback pipelines (clicks, impressions) in Flink had **huge dedup state (> 150 GB)** → slow checkpoints, restart failures, memory spikes, downtime.

**Remember:**
- Added **reconciliation** — a background job that re-verifies counts and fills gaps, a safety net for the streaming job.
- Moved to **Flink SQL** and shrank state.
- Results: **99% reduction in state size**, **> $3,000/month** infra savings, **downtime eliminated**.

**Interview hook (ties to Kafka Streams in LocalArena):**
> "Streaming state is the hard part — in LocalArena my Kafka Streams state restore took ~11 s. Zomato's answer was: don't keep everything in state; make the stream fast and approximate, and let a reconciliation job make it exact. Same idea as my reliance on idempotent consumers plus replay to prove correctness."

---

## 4. Google Research — Looking back at speculative decoding
**Link:** https://research.google/blog/looking-back-at-speculative-decoding/ (Leviathan, Kalman, Matias — 6 Dec 2024)

**What:** A small **draft model** guesses several tokens; the big model **verifies them in one parallel pass**.

**Remember:**
- Works because LLM decoding is **memory-bandwidth bound**, not compute bound — each token reads all weights, leaving spare compute.
- **Same output distribution** as normal decoding — no quality loss.
- **~2–3× speedups**; e.g. T5-XXL (11B) ~3× faster with a 60M T5-small drafter. Used in **AI Overviews in Google Search**.

**Interview hook:**
> "It's essentially CPU speculative execution applied to LLMs. The NVIDIA vLLM post I read uses the same idea — multi-token prediction with 4 draft tokens."

---

## 5. vLLM — PagedAttention
**Link:** https://vllm.ai/blog/2023-06-20-vllm (20 Jun 2023 — the foundational LLM serving post)

**What:** The **KV cache** (stored attention keys/values for every token) is huge and grows per request. Existing systems wasted **60–80%** of KV memory through fragmentation and over-reservation.

**Remember:**
- **PagedAttention** = OS virtual-memory **paging** for the KV cache: fixed-size blocks, non-contiguous, a **block table** maps them; allocate on demand. Waste **< 4%**.
- Shared prompts share blocks with **reference counting + copy-on-write** → up to **55%** less memory for parallel sampling/beam search.
- Throughput: up to **24× vs HuggingFace Transformers**, up to **3.5× vs TGI**.

**Interview hook (perfect OS crossover):**
> "If asked about paging in OS, I mention PagedAttention — the same idea let vLLM fit far more concurrent requests on a GPU, which is what drives throughput and cost in LLM serving."

---

## 6. NVIDIA — Serving GLM-5.3 across two DGX Stations with vLLM
**Link:** https://developer.nvidia.com/blog/when-one-gpu-isnt-enough-serving-glm-5-3-across-two-nvidia-dgx-stations-with-vllm/ (Fatima Fall, Nov 2026)

**What:** A **753B-parameter MoE** model (~464 GB in NVFP4) doesn't fit on one GB300 (252 GB HBM) → split across two machines.

**Remember:**
- **Tensor parallelism (TP=2)** — every layer's weights split across both GPUs; both work on every request. Linked over **RoCE / RDMA** (NCCL) — check it's not falling back to TCP.
- vLLM config: **multi-token prediction (4 draft tokens)**, **FP8 KV cache**, **prefix caching**, 93% GPU memory utilisation, max 16 concurrent sequences.
- **Throughput vs latency trade-off:** total throughput peaked at **~608 tok/s at concurrency 16**, then *dropped* at 32; per-user speed fell from **115 → 13 tok/s** as concurrency rose.

**Interview hook (shows you understand serving trade-offs):**
> "It's the same throughput-vs-tail-latency curve as any backend: past a saturation point, more concurrency just queues. In LocalArena I documented my saturation point (2,000 bots, fan-out-bound) instead of only reporting the good number."

---

## 7. Netflix — Integrating the Foundation Model into Personalization
**Link:** https://netflixtechblog.com/integrating-netflixs-foundation-model-into-personalization-applications-cf176b5860eb (Oct 2026)

**What:** Netflix centralised member-preference learning into one **foundation model**; the post is about **how to integrate a big model into existing production systems** with different latency needs.

**Remember:**
- **3 integration approaches:** **embeddings**, **subgraph**, **fine-tuning** — chosen per team's latency, stack and appetite.
- Embeddings path: model **pre-trained monthly**, **fine-tuned daily**, **batch inference** refreshes user/item embeddings into an **Embedding Store**; **embedding stabilisation** keeps daily embeddings in the same space.
- Trade-off: embeddings are **cheap and high-leverage** but can be **stale** (time gap between compute and inference) — not ideal for real-time adaptation.

**Interview hook (ties to your Alexa recommendation work — high level only):**
> "I work on recommendation serving, so this resonated: precomputed embeddings are cheap at serving time but stale; online inference is fresh but costs latency at high TPS. Choosing per use case is the real engineering decision."

---

## 8. Amazon Builders' Library — Timeouts, retries, and backoff with jitter
**Link:** https://aws.amazon.com/builders-library/timeouts-retries-and-backoff-with-jitter/ (Marc Brooker, 2019)

**Remember:**
- Set timeouts from the dependency's **latency percentiles** (e.g. p99.9).
- Retries are **"selfish"**; retrying at every layer **multiplies** load — **3 retries × 5 layers = 243×** at the database. Retry at **one** layer, use a **retry budget/token bucket**.
- **Capped exponential backoff + jitter**; only retry **idempotent** operations.

**Interview hook (your retry-storm story):**
> "I hit exactly this in LocalArena — 7,482 proxy retries during a DB brownout because the per-try timeout was shorter than the DB timeout. I fixed the timeout order (DB < per-try < API < route) and stopped retrying dependency 503s — what this Builders' Library article recommends."

---

## 9. Stripe — Designing robust and predictable APIs with idempotency
**Link:** https://stripe.com/blog/idempotency (Brandur Leach, 2017) — full notes in `../05_READING_LIST_and_Blogs.md`.
**One-liner:** Idempotency-Key header + backoff + jitter = safe retries for payments. Your LocalArena purchases and SolanaPay are direct applications.

---

## Concepts these posts make you fluent in (inference/LLM serving glossary)
- **Prefill vs decode:** prefill processes the whole prompt in parallel (compute-bound); decode generates one token at a time (memory-bound).
- **KV cache:** stored attention keys/values so earlier tokens aren't recomputed; the main memory cost of serving.
- **Continuous batching:** add/remove requests from a running batch at every step instead of waiting for the whole batch → much higher GPU utilisation (vLLM, TGI).
- **Prefix caching:** reuse KV cache for shared prompt prefixes (system prompts, chat history).
- **Speculative decoding / multi-token prediction:** draft several tokens, verify in one pass.
- **Quantisation:** FP8/INT4/NVFP4 weights or KV cache → less memory, faster, small quality cost.
- **Tensor vs pipeline parallelism:** split each layer across GPUs vs split layers into stages.
- **Disaggregated serving:** run prefill and decode on different GPU pools (NVIDIA Dynamo, llm-d).
- **Key metrics:** **TTFT** (time to first token), **TPOT/ITL** (time per output token), tokens/s, p99 latency, cost per 1M tokens, GPU utilisation.

## Blogs worth following (company engineering blogs)
Eternal/Zomato (zomato.com/blog) · Google Research (research.google/blog) · AWS ML blog & Builders' Library · NVIDIA Developer blog · vLLM blog · Netflix Tech Blog · Meta Engineering (engineering.fb.com) · Cloudflare blog · Uber Engineering · Stripe Engineering.
