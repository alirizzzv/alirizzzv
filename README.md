<h1 align="center">Ali Husain Rizvi</h1>

<p align="center">
  B.Tech · Information Technology · NSUT Delhi &nbsp;|&nbsp; 2023–2027
</p>

<p align="center">
  <a href="https://www.linkedin.com/in/ali-rizvi-4628ab286/">LinkedIn</a> &nbsp;·&nbsp;
  <a href="mailto:alirizviatwork@gmail.com">Email</a> &nbsp;·&nbsp;
  <a href="https://leetcode.com/">LeetCode</a>
</p>

---

I build systems where performance constraints are real — sub-5ms inference pipelines, browser-native security engines, real-time auction streams. I care about clean architecture, honest benchmarks, and shipping things that actually work end-to-end.

Currently a pre-final year student actively looking for **SDE internships** (backend, full-stack, or applied ML).

---

## Projects

### [ADPULSE](https://github.com/alirizzzv/ADPULSE) — Real-Time Bidding DSP &nbsp;·&nbsp; [Live Demo](https://adpulse-5r4y.onrender.com)

A full-stack Demand-Side Platform that runs live ad auctions in production. Two LightGBM models (CTR + CVR) price every bid in ~5ms; auction results stream live to an operator dashboard with a 3D RTB globe.

- **ML pipeline:** LightGBM CTR & CVR models with scikit-learn feature scaling, trained on the IPinYou RTB dataset (1.82M impressions)
- **Hot path:** pure in-memory inference — zero DB calls, zero I/O — achieving sub-5ms bid decisions at 50K+ bids/sec
- **Data layer:** O(1)-memory streaming reader for multi-GB logs; never loads a file into RAM
- **Realtime:** Flask-SocketIO broadcasts every auction decision live; Chart.js dashboard + Three.js 3D globe
- **Deployed:** Dockerised, running on Render with a single `render.yaml`

`Python` `LightGBM` `Flask` `Socket.IO` `React.js` `Three.js` `Docker`

---

### [SENTINEL](https://github.com/alirizzzv/SENTINEL) — AI Prompt Security Gateway &nbsp;·&nbsp; [Live Demo](https://alirizzzv.github.io/SENTINEL/)

A Chrome extension that intercepts prompts before they reach ChatGPT, Claude, or Gemini, detects sensitive data and prompt-injection attempts in-page, and optionally redacts them — all in under 5ms, with nothing stored but anonymised metadata.

- **Detection engine:** Aho-Corasick (Trie + BFS failure links) for O(n+m+z) multi-pattern scan — no catastrophic backtracking on adversarial input
- **Pipeline:** normalise → candidate scan → regex validation + Luhn check → injection detection (3-layer) → max-heap risk scoring → merge-interval redaction
- **Performance:** p50 of 0.002ms on typical prompts; 2.4ms on 80KB adversarial input
- **Privacy:** zero network calls in the scan path; prompt text is never stored — only anonymised metadata in a local 500-event ring buffer
- **Test coverage:** 91 tests (Vitest + pytest) including adversarial corpus and persistence tests
- **Optional enterprise backend:** FastAPI + Postgres, opt-in, org-scoped

`TypeScript` `React` `Manifest V3` `Aho-Corasick` `Vite` `FastAPI` `Vitest`

---

## Skills

**Languages:** Python · JavaScript · TypeScript · C++ · SQL

**Backend:** Node.js · Express.js · Flask · FastAPI · REST APIs

**Frontend:** React.js · Next.js · Tailwind CSS · Vite

**ML / Data:** LightGBM · scikit-learn · Pandas · NumPy

**Infra & Tools:** Docker · Git · MongoDB · MySQL

**CS Fundamentals:** DSA · OOP · OS · DBMS · Algorithm Design · System Design basics

---

## Education

**B.Tech, Information Technology** — Netaji Subhas University of Technology, Delhi &nbsp;·&nbsp; 2023–2027

- Top 10 percentile, JEE Mains
- AIR 16,500, VITEEE
- 91% in CBSE Class XII (PCM)
- 100% Educational Scholarship (2018–2022) for national-level sports performance
- 3× Gold Medal, CBSE National Championship

---

<p align="center">
  <i>Open to SDE internships — backend, full-stack, or applied ML.</i><br/>
  <a href="mailto:alirizviatwork@gmail.com">alirizviatwork@gmail.com</a>
</p>
