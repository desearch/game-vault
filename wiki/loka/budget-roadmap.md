---
title: LOKA — Budget, Timeline & OpEx Scaling
topic: loka
source: LOKA_Budget_Analysis.docx + Claude conversation log
---

# Budget, Timeline & OpEx Scaling

All figures in INR. Assumes AI-augmented team (Claude Code, Cursor, Codex). India-based team. Exchange rate: ₹84/USD.

---

## Foundational Assumptions

| Parameter | Traditional Build | LOKA AI-Augmented | Impact |
|---|---|---|---|
| MVP developer count | 15–18 engineers | 8 engineers | 55% headcount reduction |
| MVP timeline | 10–14 months | 6 months | 40–55% faster |
| Coding agent stack | — | Claude Code + Cursor + Codex | ₹1L/month, replaces ~6 devs |
| Salary band (senior) | ₹1.5–2L/month | ₹2–3L/month | Premium for AI-fluency |

**AI Coding Agents are infrastructure, not perks.** They are the single biggest lever on the entire budget.

---

## Phase 1 — MVP (6 months | 100 Alpha Users | Flat screen + API)

*Core loop proven: Intent Engine, Oracle + Weaver agents, Private Bubble, single timeline TL-2024*

### 1A. Team

| Role | Count | ₹/Month | Monthly Total | 6-Month Total |
|---|---|---|---|---|
| Tech Lead / Architect | 1 | ₹3,00,000 | ₹3,00,000 | ₹18,00,000 |
| Senior Full-Stack Engineer (AI-augmented) | 2 | ₹2,00,000 | ₹4,00,000 | ₹24,00,000 |
| AI / ML Engineer | 1 | ₹2,50,000 | ₹2,50,000 | ₹15,00,000 |
| Unreal Engine 5 Developer | 1 | ₹2,00,000 | ₹2,00,000 | ₹12,00,000 |
| DevOps / Cloud Engineer | 1 | ₹1,50,000 | ₹1,50,000 | ₹9,00,000 |
| QA Engineer (automated-first) | 1 | ₹1,00,000 | ₹1,00,000 | ₹6,00,000 |
| Product Designer | 1 | ₹1,20,000 | ₹1,20,000 | ₹7,20,000 |
| **TEAM TOTAL** | **8** | | **₹15,20,000** | **₹91,20,000** |

### 1B. AI Coding Tools

| Tool | Users | Monthly | 6-Month Total |
|---|---|---|---|
| Claude Code (heavy usage) | 4 devs | ₹67,000 | ₹4,02,000 |
| Cursor Pro | 6 devs | ₹20,000 | ₹1,20,000 |
| GitHub Copilot | 6 devs | ₹9,500 | ₹57,000 |
| **TOOLING TOTAL** | | **₹96,500** | **₹5,79,000** |

### 1C. AI / LLM API Costs

| Service | Purpose | Monthly | 6-Month Total |
|---|---|---|---|
| Claude API (Sonnet) | Oracle, Weaver, Challenger agents | ₹1,67,000 | ₹10,02,000 |
| ElevenLabs | NPC spatial voice | ₹42,000 | ₹2,52,000 |
| Suno API | In-world music generation | ₹25,000 | ₹1,50,000 |
| Stable Diffusion XL (cloud) | Artifact image generation | ₹33,000 | ₹1,98,000 |
| Whisper API | Player speech to intent | ₹8,000 | ₹48,000 |
| **AI API TOTAL** | | **₹2,75,000** | **₹16,50,000** |

*Self-hosting becomes essential at 5,000+ DAU — pre-planned for Phase 2.*

### 1D. Cloud Infrastructure (AWS / Azure)

| Component | Spec | Monthly | 6-Month Total |
|---|---|---|---|
| Compute (ECS / EC2) | Game servers, API layer, agent mesh | ₹67,000 | ₹4,02,000 |
| Databases | PostgreSQL, ClickHouse, Redis | ₹25,000 | ₹1,50,000 |
| Kafka (MSK) | Intent event streaming | ₹17,000 | ₹1,02,000 |
| Object Storage (S3 / R2) | Artifacts, world state snapshots | ₹8,500 | ₹51,000 |
| CDN + Networking | Cloudflare, data transfer | ₹17,000 | ₹1,02,000 |
| Monitoring (Grafana Cloud) | Observability, logging | ₹8,500 | ₹51,000 |
| **INFRA TOTAL** | | **₹1,43,000** | **₹8,58,000** |

### 1E. One-Time Costs

| Item | Cost |
|---|---|
| Company registration (Pvt Ltd) | ₹50,000 |
| Legal (IP, employment contracts, TOS/PP) | ₹1,50,000 |
| Development hardware (VR headsets ×3, GPU workstations ×2) | ₹5,00,000 |
| Unreal Engine license | ₹0 (free < $1M annual revenue) |
| Domain, email, SaaS setup | ₹50,000 |
| Security audit (initial penetration test) | ₹1,50,000 |
| **ONE-TIME TOTAL** | **₹9,00,000** |

### Phase 1 Summary

| Line Item | Amount |
|---|---|
| Team | ₹91,20,000 |
| AI Coding Tools | ₹5,79,000 |
| AI / LLM APIs | ₹16,50,000 |
| Cloud Infrastructure | ₹8,58,000 |
| One-Time Costs | ₹9,00,000 |
| Subtotal | ₹1,31,07,000 |
| Contingency (15%) | ₹19,66,050 |
| **PHASE 1 TOTAL** | **≈ ₹1.5 Crore** |

*Traditional equivalent (15 engineers, 12 months): ₹2.8–3.5 Cr. Savings: ₹1.3–2 Cr, 6 months faster.*

---

## Phase 2 — Encounter Layer (Months 7–12 | 10K Beta Users | Multiplayer + VR debut)

*Shared Territory, Expeditions, Challenger agent, Artifact economy, On-chain, 3 Pantheon categories, Meta Quest 3 beta*

### 2A. Team Additions to Phase 1 Base

| Role | New Hires | ₹/Month | Monthly Addition | 6-Month Cost |
|---|---|---|---|---|
| Phase 1 base team (8 people) | — | — | ₹15,20,000 | ₹91,20,000 |
| Senior Backend / Scale Engineer | 2 | ₹2,00,000 | ₹4,00,000 | ₹24,00,000 |
| Blockchain / Smart Contract Developer | 1 | ₹2,00,000 | ₹2,00,000 | ₹12,00,000 |
| VR / XR Developer (UE5 + OpenXR) | 1 | ₹2,00,000 | ₹2,00,000 | ₹12,00,000 |
| Security Engineer | 1 | ₹2,00,000 | ₹2,00,000 | ₹12,00,000 |
| Community Manager | 1 | ₹80,000 | ₹80,000 | ₹4,80,000 |
| **PHASE 2 TEAM TOTAL (13 people)** | | | **₹26,00,000** | **₹1,56,00,000** |

### 2B. AI Infrastructure — Hybrid Model Begins

| Component | Strategy | Monthly | 6-Month Total |
|---|---|---|---|
| Self-hosted GPU (4× A100 on RunPod) | 70% of inference (fine-tuned Llama) | ₹2,70,000 | ₹16,20,000 |
| Claude API (complex narrative, Weaver) | 30% high-quality tasks | ₹1,67,000 | ₹10,02,000 |
| ElevenLabs (10K users, bulk plan) | NPC voice at scale | ₹1,26,000 | ₹7,56,000 |
| Suno / MusicGen API | Higher volume | ₹84,000 | ₹5,04,000 |
| Stable Diffusion (local on GPU cluster) | No extra cost | ₹0 | ₹0 |
| **AI TOTAL** | | **₹5,47,000** | **₹32,82,000** |

*Hybrid model drops per-user AI cost from ₹275/month → ₹55/month — the critical unit economics inflection point.*

### 2C. Cloud Infrastructure (10K Users, Spatial MMO scale)

| Component | Monthly | 6-Month Total |
|---|---|---|
| Game servers (AWS GameLift / SpatialOS) | ₹4,20,000 | ₹25,20,000 |
| Kafka cluster (MSK) — higher throughput | ₹84,000 | ₹5,04,000 |
| Neo4j (lineage graph) | ₹59,000 | ₹3,54,000 |
| ClickHouse (analytics) | ₹67,000 | ₹4,02,000 |
| PostgreSQL + TimescaleDB | ₹50,000 | ₹3,00,000 |
| CDN + storage (artifact scale) | ₹59,000 | ₹3,54,000 |
| Monitoring + security tooling | ₹42,000 | ₹2,52,000 |
| **INFRA TOTAL** | **₹6,81,000** | **₹40,86,000** |

### 2D. One-Time Phase 2 Costs

| Item | Cost |
|---|---|
| Smart contract security audit (ERC-2981 + marketplace) | ₹8,00,000 |
| L2 blockchain deployment (Base / Arbitrum) | ₹1,50,000 |
| Meta Quest 3 dev kits ×10 (VR QA fleet) | ₹2,50,000 |
| Penetration testing (Round 2 — multiplayer attack surface) | ₹3,00,000 |
| GDPR / DPDP Act compliance legal review | ₹2,00,000 |
| **ONE-TIME TOTAL** | **₹17,00,000** |

### Phase 2 Summary

| Line Item | Amount |
|---|---|
| Team (13 people × 6 months) | ₹1,56,00,000 |
| AI Infrastructure (hybrid GPU + APIs) | ₹32,82,000 |
| Cloud Infrastructure | ₹40,86,000 |
| One-Time Costs | ₹17,00,000 |
| AI Coding Tools (scaled) | ₹9,00,000 |
| Subtotal | ₹2,55,68,000 |
| Contingency (15%) | ₹38,35,200 |
| **PHASE 2 TOTAL** | **≈ ₹2.95 Crore** |
| **Cumulative (P1 + P2)** | **≈ ₹4.45 Crore** |

---

## Phase 3 — The Manifold (Months 13–24 | 100K–500K Users)

*Full VR, 4 timelines, on-chain economy, Generational Ancestry, Elder AI (Tier 5), full Pantheon — Series A raise targeted end of Phase 3*

### 3A. Team (32 people, 8% salary inflation buffer applied mid-year)

| Department | Headcount | Monthly | 12-Month Total |
|---|---|---|---|
| Engineering (Phase 2 base 13 + 7 new) | 20 | ₹42,00,000 | ₹5,04,00,000 |
| AI / ML Research (2 new) | 2 | ₹6,00,000 | ₹72,00,000 |
| Data Engineering (1 new) | 1 | ₹2,00,000 | ₹24,00,000 |
| Game Design (2 new) | 2 | ₹3,00,000 | ₹36,00,000 |
| Marketing & Growth (2 new) | 2 | ₹3,50,000 | ₹42,00,000 |
| Legal, Finance, HR (3 new) | 3 | ₹5,00,000 | ₹60,00,000 |
| Customer Support (2 new) | 2 | ₹1,60,000 | ₹19,20,000 |
| **PHASE 3 TEAM TOTAL** | **32** | **₹63,10,000** | **₹7,57,20,000** |

### 3B. AI Infrastructure — Self-Hosted Cluster

| Component | Scale | Monthly | 12-Month Total |
|---|---|---|---|
| GPU cluster (20× A100 — owned/long-lease) | 100K concurrent inference | ₹13,44,000 | ₹1,61,28,000 |
| Claude API (Weaver, ~5% of calls) | Narrative crown jewels only | ₹4,20,000 | ₹50,40,000 |
| ElevenLabs Enterprise | 500K user NPC voices | ₹4,20,000 | ₹50,40,000 |
| MusicGen (local GPU) | Zero marginal cost | ₹0 | ₹0 |
| Whisper (local) | Speech-to-intent, local GPU | ₹0 | ₹0 |
| **AI TOTAL** | | **₹21,84,000** | **₹2,62,08,000** |

### 3C. Cloud Infrastructure (100K–500K Users)

| Component | Monthly | 12-Month Total |
|---|---|---|
| Game servers (SpatialOS / custom — multi-region) | ₹25,08,000 | ₹3,00,96,000 |
| Kafka + ClickHouse | ₹3,36,000 | ₹40,32,000 |
| Neo4j Enterprise (ancestry graph) | ₹1,68,000 | ₹20,16,000 |
| PostgreSQL + TimescaleDB cluster | ₹2,52,000 | ₹30,24,000 |
| Storage (R2 + IPFS for artifacts) | ₹2,10,000 | ₹25,20,000 |
| CDN (Cloudflare Enterprise) | ₹1,68,000 | ₹20,16,000 |
| Security, WAF, compliance | ₹84,000 | ₹10,08,000 |
| **INFRA TOTAL** | **₹37,26,000** | **₹4,47,12,000** |

### 3D. Marketing (Phase 3 is the Launch Year)

| Campaign | Budget |
|---|---|
| Performance marketing (Meta, Google, gaming channels) | ₹1,00,00,000 |
| Creator / influencer partnerships | ₹50,00,000 |
| VR launch event + media coverage | ₹30,00,000 |
| Research partnerships (IIT, academic institutions) | ₹10,00,000 |
| IP / Patent filings (Intent Engine, Ancestry System) | ₹10,00,000 |
| **MARKETING TOTAL** | **₹2,00,00,000** |

### Phase 3 Summary

| Line Item | Amount |
|---|---|
| Team | ₹7,57,20,000 |
| AI Infrastructure | ₹2,62,08,000 |
| Cloud Infrastructure | ₹4,47,12,000 |
| Marketing & Growth | ₹2,00,00,000 |
| AI Coding Tools + Misc | ₹18,00,000 |
| Subtotal | ₹16,84,40,000 |
| Contingency (15%) | ₹2,52,66,000 |
| **PHASE 3 TOTAL** | **≈ ₹19.4 Crore** |
| **Cumulative (P1–P3)** | **≈ ₹23.85 Crore** |

*Series A target: ₹25–40 Cr at ₹150–250 Cr valuation. Revenue should cover 40–60% of Phase 3 OpEx by month 22.*

---

## Phase 4 — The Living World (Year 3+ | 1M+ Users | Series B)

*Unlimited timelines, cross-timeline travel, The Unnamed leaderboard, Chronicle publishing, research licensing, owned GPU datacenter*

### Annual OpEx at 1M Users

| Cost Centre | Monthly | Annual | % of OpEx |
|---|---|---|---|
| Team (65–80 people) | ₹1,40,00,000 | ₹16,80,00,000 | 46% |
| AI Inference (owned GPU + spot) | ₹1,00,00,000 | ₹12,00,00,000 | 33% |
| Cloud / Game Servers (multi-region) | ₹60,00,000 | ₹7,20,00,000 | 20% |
| Marketing & Partnership | ₹25,00,000 | ₹3,00,00,000 | 8% |
| Legal, Compliance, Research | ₹15,00,000 | ₹1,80,00,000 | 5% |
| **PHASE 4 TOTAL** | **₹3,40,00,000** | **≈ ₹40.8 Crore/year** | |

*Revenue at 1M users: ₹120–180 Cr/year. Operating margin: 65–75%.*

---

## OpEx Scaling With User Growth

### Per-User Monthly Cost vs. ₹999 Subscription

| Users | Infra/User | AI/User | Team/User | Total OpEx/User | Margin |
|---|---|---|---|---|---|
| 100 (Alpha) | ₹1,430 | ₹2,750 | ₹15,200 | ₹19,380 | Deep loss — expected |
| 1,000 | ₹250 | ₹400 | ₹1,520 | ₹2,170 | Loss — building |
| 10,000 | ₹68 | ₹55 | ₹260 | ₹383 | ₹616 margin begins |
| 100,000 | ₹37 | ₹22 | ₹63 | ₹122 | ₹877 margin (88%) |
| 500,000 | ₹15 | ₹8 | ₹14 | ₹37 | ₹962 margin (96%) |
| 1,000,000 | ₹7 | ₹4 | ₹14 | ₹25 | ₹974 margin (97%) |

**₹10K-user threshold is the critical inflection**: per-user cost drops below ₹400, ₹999 subscription becomes profitable.

### Why Per-User Cost Collapses

- **Infrastructure**: scales with *concurrent* users × world complexity, not total users. Offline players cost near-zero. SpatialOS zone-based = pay for active zones only.
- **AI Inference**: drops 687× from alpha to 1M users. Three forces: (1) self-hosted GPU amortizes over massive volume, (2) PIM becomes more accurate → fewer inference calls per session, (3) world cultural memory = compression layer → AI generates less novelty because player history already contains it.
- **Team**: step function, not linear. 10,000 new users requires zero new hires within a phase.

---

## Revenue vs. OpEx Crossover

| Milestone | Monthly Revenue | Monthly OpEx | Status | Timing |
|---|---|---|---|---|
| 100 alpha users | ₹0 (closed test) | ₹18.9L | Investment | Month 1–6 |
| 1,000 beta (50% paying) | ₹5L | ₹21.7L | Loss: ₹16.7L | Month 7–9 |
| 5,000 users (70% paying) | ₹35L | ₹30L | Near breakeven | Month 10–12 |
| **15,000 users (75% paying)** | **₹1.12 Cr** | **₹42L** | **PROFITABLE ✓** | Month 14–16 |
| 100,000 users (80% paying) | ₹7.99 Cr | ₹1.22 Cr | Strong margin | Phase 3 |
| 1,000,000 users (85% paying) | ₹84.9 Cr | ₹3.4 Cr | Extraordinary | Phase 4 |

*Note: Revenue model assumes ₹999/month base subscription only. Artifact marketplace fees, timeline passes, intelligence tier upgrades, and research licensing add 30–50% to ARPU at scale.*

---

## Master Budget Summary

| Phase | Duration | Users | Team | Budget | Cumulative |
|---|---|---|---|---|---|
| Phase 1 — MVP | 6 months | 100 | 8 | ₹1.50 Cr | ₹1.50 Cr |
| Phase 2 — Encounters | 6 months | 10,000 | 13 | ₹2.95 Cr | ₹4.45 Cr |
| Phase 3 — Manifold | 12 months | 500,000 | 32 | ₹19.4 Cr | ₹23.85 Cr |
| Phase 4 — Living World | Year 3+ | 1,000,000+ | 70+ | ₹40.8 Cr/yr | Series B funded |
| **TOTAL TO SCALE** | **~3 years** | **1M+** | | **~₹65 Crore** | |

---

## Funding Rounds

| Round | When | Amount | Funds | Valuation Target |
|---|---|---|---|---|
| Pre-seed / Bootstrapped | Now | ₹1.5–2 Cr | Phase 1 MVP | Proof of concept |
| Seed | Month 6–8 | ₹5–8 Cr | Phase 2 full + Phase 3 start | ₹25–50 Cr |
| Series A | Month 20–24 | ₹30–50 Cr | Phase 3 completion + Phase 4 start | ₹150–300 Cr |
| Series B | Year 3 | ₹100–200 Cr | Global scaling, hardware, research | ₹800 Cr–₹1,500 Cr |

*MVP is fundable from a single angel or small pre-seed. Series A story: 10K+ paying users, provable unit economics, and a living world no competitor can replicate without 24 months of player history.*

---

## AI Agent Build Savings Summary

| Metric | Traditional Build | LOKA AI-Agent Model | Savings |
|---|---|---|---|
| Phase 1 team size | 15–18 engineers | 8 engineers | 55% smaller |
| Phase 1 duration | 10–14 months | 6 months | 4–8 months faster |
| Phase 1 cost | ₹2.8–3.5 Cr | ₹1.5 Cr | ₹1.3–2 Cr saved |
| Code review cycles | Manual | Agent-assisted | 60% faster iteration |
| Test coverage | 40–60% | 80–90% (agent-generated) | Higher quality at launch |
| Total timeline to 1M users | 5–6 years | 3 years | 2–3 years competitive advantage |

---

## Key Takeaways

- Phase 1 at ₹1.5 Cr validates the one thing that matters: the intent loop
- Self-hosting inflection at ~5K DAU (Phase 2): per-user AI cost drops from ₹275 → ₹55/month — unit economics hinge on this transition
- **Profitable at 15,000 paying users** — achievable in Phase 2 with disciplined marketing
- At 1M users: ₹25/user/month OpEx vs. ₹999 subscription = 97% operating margin
- AI inference drops 687× from alpha to scale — three forces: self-hosting amortization, PIM accuracy, world cultural memory as compression
- Series A is the Phase 3 story: raise on provable unit economics, not just vision
- Subscription ARPU understated by 30–50% — marketplace fees, timeline passes, and intelligence upgrades are additive

## Related Articles
- [[game-concept]] — Phase roadmap and MVP scope definition
- [[technical-architecture]] — Self-hosting stack (Llama, Stable Diffusion, A100 infrastructure)
- [[multi-agent-system]] — Agent inference architecture driving the biggest cost variable
