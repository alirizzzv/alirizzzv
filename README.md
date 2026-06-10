<h1 align="center">Ali Husain Rizvi</h1>

<p align="center">
  B.Tech · Information Technology · NSUT Delhi &nbsp;|&nbsp; 2023–2027
</p>

<p align="center">
  <a href="https://www.linkedin.com/in/ali-rizvi-4628ab286/">LinkedIn</a> &nbsp;·&nbsp;
  <a href="mailto:alirizviatwork@gmail.com">Email</a>
</p>

<p align="center">
  <img src="https://img.shields.io/badge/Python-3776AB?style=flat-square&logo=python&logoColor=white"/>
  <img src="https://img.shields.io/badge/TypeScript-3178C6?style=flat-square&logo=typescript&logoColor=white"/>
  <img src="https://img.shields.io/badge/JavaScript-F7DF1E?style=flat-square&logo=javascript&logoColor=black"/>
  <img src="https://img.shields.io/badge/C++-00599C?style=flat-square&logo=c%2B%2B&logoColor=white"/>
  <img src="https://img.shields.io/badge/React-20232A?style=flat-square&logo=react&logoColor=61DAFB"/>
  <img src="https://img.shields.io/badge/Flask-000000?style=flat-square&logo=flask&logoColor=white"/>
  <img src="https://img.shields.io/badge/Docker-2496ED?style=flat-square&logo=docker&logoColor=white"/>
  <img src="https://img.shields.io/badge/LightGBM-9cf?style=flat-square"/>
</p>

---

I build systems where the constraints are real — sub-5ms ML inference under live auction pressure, browser-native security engines that intercept before a single byte leaves the page, real-time pipelines that stream millions of events without ever touching RAM. I care about data structures, honest benchmarks, clean architecture, and shipping things that hold up in production.

Pre-final year B.Tech student at NSUT Delhi, actively looking for **SDE internships** — backend, full-stack, or applied ML.

---

## 🔨 Projects

---

### [📈 ADPULSE](https://github.com/alirizzzv/ADPULSE) — Real-Time Bidding DSP
#### *[Live Demo](https://adpulse-5r4y.onrender.com) · [Dashboard](https://adpulse-5r4y.onrender.com/dashboard) · [Bid Tester](https://adpulse-5r4y.onrender.com/bidtester)*

> A production-deployed, full-stack **Demand-Side Platform** — the system that sits on the buy side of a real-time ad auction, decides whether to bid on each impression, and how much.

Every time a webpage loads, a millisecond auction happens invisibly for that ad slot. ADPULSE is the bidder: it receives a bid request, predicts the value of the impression using two ML models, prices a bid using a formula derived from the advertiser's objectives, and returns a decision — in under 5ms, at the scale of a live stream, without ever running out of memory.

**The ML layer**

Two LightGBM gradient-boosted models are trained on the [IPinYou RTB dataset](https://contest.ipinyou.com/) — 1.82 million real impressions across 5 advertiser campaigns. One model predicts **CTR** (click-through rate), the other predicts **CVR** (conversion rate). Both use scikit-learn pipelines for feature scaling. Features are engineered from raw log fields: ad slot dimensions, visibility tier, format, ad exchange, region, city, advertiser ID, floor price, and hashed domain/URL signals.

The bidding formula is:
```
bid = base_bid × CTR × (1 + N × CVR)
```
CTR is the primary gate — a low click probability shrinks the entire bid. The CVR multiplier scales the bid upward for advertisers where conversions matter more. `N` is campaign-specific: for advertiser `3476` (Tire), N=10, meaning a high-conversion-probability impression can multiply the base bid by up to 11×.

**The hot path — why it's fast**

Models are loaded once at startup into memory. `getBidPrice()` does pure in-memory inference with no I/O, no DB calls, no locks — just numpy arrays and the LightGBM C++ runtime. This is how the system sustains **50K+ bid decisions per second** at **~5ms p99**.

**The data layer — O(1) memory at multi-GB scale**

The full IPinYou dataset is multiple gigabytes across seven days of logs. Loading it naively would blow the memory budget. Instead, a generator reads the log **line by line**, yields one `BidRequest` at a time, and loops forever — the entire week of data never enters RAM. An auto-detecting parser handles both the 20-column bid log and the 24-column impression/click/conversion log, mapping both onto a common schema.

**The real-time layer**

A single background producer thread runs the auction loop and feeds two things simultaneously: in-memory aggregate stats and a Flask-SocketIO broadcast. The frontend — a dark HUD-style operator dashboard built in vanilla JS + Chart.js — receives every auction event over WebSocket and renders live KPIs, bid-price trends, CTR/CVR charts, and a Three.js / globe.gl **3D globe** that visualises bid flow by geography in real time.

**Deployment**

Containerised with Docker (`python:3.9-slim` + `libgomp1` for LightGBM's OpenMP dependency), deployed to Render via a single `render.yaml` Blueprint. Graceful degradation: if the dataset isn't present, it transparently falls back to a synthetic generator — so the live demo runs anywhere with zero data setup.

**Outcome modelling vs. ground truth**

Real display CTR is ~0.06% — far too sparse to render meaningfully in a live view. The live dashboard uses model-predicted outcomes (amplified by a configurable scale factor) for visibility. An `OUTCOME_MODE=real` flag switches to ground-truth labels for offline validation. This is exactly how production DSP dashboards work: live modelled performance, reconciled with sparse actuals in batch.

```
Stack: Python · LightGBM · scikit-learn · Flask · Flask-SocketIO · Vanilla JS · Chart.js · Three.js · Docker · Render
```

---

### [🛡 SENTINEL](https://github.com/alirizzzv/SENTINEL) — AI Prompt Security Gateway
#### *[Live Demo](https://alirizzzv.github.io/SENTINEL/) · [Architecture Docs](https://github.com/alirizzzv/SENTINEL/blob/main/docs/ARCHITECTURE.md)*

> A Manifest V3 Chrome extension that intercepts prompts **before** they reach ChatGPT, Claude, or Gemini — detecting API keys, credentials, PII, and prompt-injection attacks in-page in under 5ms, with zero network calls and nothing stored but anonymised metadata.

Every day, people paste credentials, internal docs, and customer data into LLMs to "just debug this." The instant they hit send, that data is on a third-party server — permanently outside their control. SENTINEL intercepts before the send, runs entirely in the browser, and works across every major LLM with one install.

**The detection engine**

The core is a framework-free JavaScript engine (`src/engine`) — no dependencies, loads instantly, runs synchronously in-page. Detection is a multi-stage pipeline:

```
Raw prompt
  → Normalize
  → Aho-Corasick candidate scan       (one linear pass, all patterns simultaneously)
  → Regex validation + Luhn check + sub-match suppression
  → Injection detection                (3 layers: phrase patterns · heuristics · confidence score)
  → Risk scoring                       (max-heap composite → SAFE / CAUTION / HIGH)
  → Redaction                          (merge-intervals → [PLACEHOLDERS])
  → Result: { level, score, threats[], redactedText }
```

**Why Aho-Corasick?**

Naive multi-pattern matching runs each regex independently — O(n × p) for n characters and p patterns. Aho-Corasick builds a Trie from all patterns, then adds BFS failure links to turn it into a DFA. One linear pass over the text — O(n + m + z) where m is total pattern length and z is match count — finds all patterns simultaneously with no catastrophic backtracking, even on adversarial inputs designed to trigger worst-case regex behaviour.

**The data structures, chosen deliberately**

| Problem | Structure | Why |
|---------|-----------|-----|
| Multi-pattern scan | Aho-Corasick DFA | O(n+m+z), adversarial-safe |
| Threat prioritisation | Hand-built max-heap | O(k log k), no library overhead |
| Adapter lookup (per LLM site) | Hash map | O(1) dispatch |
| Local event history | Circular buffer (FIFO, 500 cap) | O(1) amortised, bounded memory |
| Overlap resolution in redaction | Merge intervals | O(n log n), handles nested matches |

**Performance — measured, not estimated**

`npm run bench` on Node 25:

| Input | Size | p50 | p99 |
|-------|------|-----|-----|
| Typical prompt | ~50 B | 0.002 ms | 0.01 ms |
| Realistic mixed | ~1 KB | 0.023 ms | 0.045 ms |
| Large document | ~10 KB | 0.26 ms | 0.38 ms |
| Adversarial near-miss | ~80 KB | 2.4 ms | 2.6 ms |

**Privacy by architecture**

SENTINEL requests only `storage` + the three LLM host permissions. It makes **zero network requests during scanning** — verifiable in DevTools → Network. Prompt text and detected values are never persisted; only anonymised metadata (pattern category, risk level, timestamp) is stored in a local IndexedDB ring buffer capped at 500 events.

**Extension architecture**

The same engine that passes 91 unit tests (Vitest + pytest, including adversarial corpus and persistence tests) is the exact bundle that ships in the content script — built with esbuild, single source of truth. The adapter pattern means adding a new LLM takes ~5 lines. An optional FastAPI + Postgres enterprise backend provides org-scoped sync, entirely opt-in.

**The dashboard**

A full React + Vite SPA (built to static files, bundled into the extension) shows risk trends, threat category breakdown, filterable detection history, and settings — reading only the local IndexedDB metadata.

```
Stack: TypeScript · React · Vite · Manifest V3 · Aho-Corasick · FastAPI · SQLAlchemy · Vitest · pytest · Docker
```

---

## 🛠 Skills

| Area | Technologies |
|------|-------------|
| **Languages** | Python · JavaScript · TypeScript · C++ · SQL |
| **Backend** | Node.js · Express.js · Flask · FastAPI · REST APIs |
| **Frontend** | React.js · Next.js · Tailwind CSS · Vite · Vanilla JS |
| **ML / Data** | LightGBM · scikit-learn · Pandas · NumPy |
| **Databases** | MongoDB · MySQL · PostgreSQL · IndexedDB |
| **Infra & Tools** | Docker · Git · GitHub Actions · esbuild |
| **CS Fundamentals** | DSA · OOP · OS · DBMS · Algorithm Design · System Design |

---

## 🎓 Education

**B.Tech, Information Technology** — Netaji Subhas University of Technology (NSUT), Delhi &nbsp;·&nbsp; 2023–2027

| | |
|---|---|
| JEE Mains | Top 10 percentile nationally |
| VITEEE | All India Rank 16,500 |
| CBSE Class XII | 91% with distinction in PCM |
| Scholarship | 100% Educational Scholarship (2018–2022) for national-level sports performance |
| Athletics | 3× Gold Medal, CBSE National Championship |

---

## 📌 Positions of Responsibility

**Team Lead — Security, Moksha Fest** *(Largest cultural fest in Northeast India)*
Led a team responsible for event security planning, coordination, and on-ground execution across the full duration of the fest.

---

<p align="center">
  Open to SDE internships — backend, full-stack, or applied ML.<br/>
  <a href="mailto:alirizviatwork@gmail.com">alirizviatwork@gmail.com</a> &nbsp;·&nbsp;
  <a href="https://www.linkedin.com/in/ali-rizvi-4628ab286/">LinkedIn</a>
</p>
