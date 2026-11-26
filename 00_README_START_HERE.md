# Backend Interview Prep Package — Sanidhya Madeshia

For **any backend / SDE / software engineer intern or new-grad role**. Everything is built from **your resume** and **your own GitHub repos** (LocalArena `FINAL_REPORT.md` + `docs/`, ttl-based-chats `docs/`, solanaPaymentsStripe backend code). Every number you say is on your resume — know where each came from.

## What's in this folder

| File | What it is | When to use |
|---|---|---|
| `resume/Sanidhya_Resume_General_overleaf.txt` | Your backend resume (paste into Overleaf) | Every application |
| `01_EASY_Questions.md` | Intro, "walk me through", project pitches, CS basics | Day 1 — must be fluent |
| `02_MEDIUM_Questions.md` | Design choices, trade-offs, debugging stories | Day 2–3 |
| `03_HARD_Questions.md` | Deep dives, system design, failure modes, "what would you change" | Day 4–5 |
| `04_BEHAVIORAL_STAR.md` | Behavioral stories in STAR format + closing questions | Before every interview |
| `05_READING_LIST_and_Blogs.md` | Must-read blog (summarised) + your own repo docs + fundamentals | Day 5 |
| `06_CHEAT_SHEET_Numbers.md` | Every number on your resume and how it was measured | Night before + 10 min before |
| `07_Core_Subjects/` | Full guides: OS, CN, DBMS, OOP & LLD — simple explanations, worked examples, runnable code, **Top 40 most-asked questions** + last-minute revision in each | 2–3 days per subject alongside DSA |
| `08_General_Interview_Questions/` | HR, logistics, work-style, end-of-interview questions | Before HR / manager rounds |
| `09_Tech_Blogs/` | 9 curated blogs (Eternal, Google, Amazon, NVIDIA, vLLM, Netflix, Stripe) + LLM-serving glossary | 1 blog/day, 10 min |

## 7-day revision plan

1. **Day 1:** `01_EASY` — say every answer **out loud**, under 2 minutes each. Record yourself once.
2. **Day 2:** `02_MEDIUM` sections A–C (Amazon, Inbotiq, LocalArena).
3. **Day 3:** `02_MEDIUM` sections D–F (SolanaPay, TTL chat, CS fundamentals).
4. **Day 4:** `03_HARD` — draw LocalArena's architecture on paper from memory.
5. **Day 5:** `03_HARD` remaining + read the blog in `05_READING_LIST`.
6. **Day 6:** `04_BEHAVIORAL_STAR` — tell each story out loud in ≤ 2 minutes.
7. **Day 7:** `06_CHEAT_SHEET` + mock interview with a friend (they pick random questions).

## Before each specific interview (30 min)
- Read the job description; note 5 keywords and map each to a line on your resume.
- Read 1–2 posts from the company's engineering blog → fill in `E3 "Why do you want to join us?"` in `01_EASY`.
- Pick which 3 stories from `04_BEHAVIORAL_STAR` fit their values.

## Online assessment (coding round) plan
Most backend roles start with a DSA round. Focus in this order:
1. Arrays/strings, two pointers, sliding window, prefix sums
2. HashMap/HashSet counting
3. Sorting + binary search (including "binary search on answer")
4. Stack/queue, monotonic stack
5. Trees (BFS/DFS), graphs (BFS, DFS, topological sort, Dijkstra)
6. Dynamic programming (1D, 2D grid, knapsack, LIS)
7. Heaps (top-K, merge K lists), intervals (merge/insert)

Practise in **C++** (your strongest DSA language; use STL fluently). Be ready to explain in Java if asked. ~3 timed problems/day.

## Golden rules for every answer

1. **Structure:** *Context → What I did → How (tech) → Result (number) → What I learned.*
2. **Say "I"**, not "we", for your own work — but be honest about team work.
3. **Amazon confidentiality:** describe *what* and *why*, not internal service names, code, or metrics beyond what's on your resume. Say *"I can't go into internal details, but at a high level…"* — interviewers respect this.
4. **Never bluff.** "I haven't used X, but I've used Y which is similar, and here's how I'd learn it" beats a fake answer.
5. **Tie back to the job description:** scalable APIs, event-driven systems, databases, CI/CD, reliability — your resume covers all of them. Say so.

## Beyond DSA — what to prepare
1. **Core subjects** (`07_Core_Subjects/`): DBMS > OS > CN > OOP/LLD. ~1 hour/day each, rotating.
2. **Your projects**: be able to draw LocalArena and explain any number (`03_HARD`, `06_CHEAT_SHEET`).
3. **Basic system design**: URL shortener, rate limiter, chat app, notification system, distributed cache (H7).
4. **LLD**: 4 classic designs in Java (`07_Core_Subjects/04_OOP_and_LLD.md`).
5. **Tech awareness**: 1 blog/day from `09_Tech_Blogs/` — enough to discuss LLM serving and backend trends.
6. **Mock interviews**: 1/week with a friend or Pramp; record yourself.
7. **Profile polish**: pin LocalArena, TTL chat, SolanaPay on GitHub with good READMEs; update LinkedIn headline to match the resume.
