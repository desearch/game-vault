---
title: LOKA — Artifact Economy & Creative Yield System
topic: loka
---

# Artifact Economy & Creative Yield System

---

## What Is an Artifact?

Any durable creation: structure, music, written text, painted image, crafted tool, map, covenant, recipe. Every artifact has:

- **Material composition** — physical resources used
- **Intent signature** — creator's intent vector at creation time (embedded, invisible, retrievable)
- **Timeline provenance** — when and where made
- **Chronicle reference** — which narrative event it's part of
- **On-chain ownership record** — creation and transaction history

### Media Creation as Gameplay

Players create **real media** with functional in-world effects:

| Medium | Creation Tool | Game Effect |
|---|---|---|
| Music | In-world instruments + AI (Smith + music model) | Played in settlement → productivity aura for all passing players |
| Visual art | Painted, sculpted, architecturally composed | — |
| Written texts | Letters, manifestos, histories, fiction (Weaver assists) | Manifesto displayed publicly → shifts cultural drift of settlement |
| Maps | Hand-drawn cartography | Shared map → reduces another player's fog of war |
| Blueprints | Technical schematics | Sold → buyer builds faster |

---

## The Three-Layer Economy

**Layer 1 — In-world barter** (no real money)
Timeline-appropriate trade: goods, services, knowledge, access.

**Layer 2 — LOKA Credits** (in-game earned currency)
Earned via exploration milestones, creative output, encounter quality, ancestry activation. Spent on intelligence tier upgrades, timeline passes, Private Bubble expansions.

**Layer 3 — True Ownership (on-chain)**
Artifacts of sufficient quality (validated by community panel + AI) are minted as chain-registered assets. Tradeable outside the game. Creator receives **perpetual royalties** on all future transactions via smart contract.

Quality threshold is deliberately high — **Proof of Intent** required: artifact must have a traceable, coherent intent lineage to qualify.

---

## The Creative Yield System (CYS)

An **economic truth machine** — live AI assessment of what creative output is genuinely worth.

### Core Principle
Most games reward time or grind. LOKA rewards **irreplaceability**. The central question: *could anyone have made this, or did it require exactly you?*

### The Five Creative Signals

| Signal | What It Measures | How AI Reads It |
|---|---|---|
| **Originality** | How unlike prior artifacts in the timeline? | Cosine distance from existing artifact embedding space |
| **Intent Coherence** | Does output reflect genuine, consistent desire? | Alignment between PIM vector and artifact signature |
| **World Impact** | Did other players' behaviour change because of this? | Downstream player action attribution in event log |
| **Craft Depth** | How much layered decision-making went into construction? | Decision tree complexity from build events |
| **Temporal Durability** | Still being interacted with weeks/months later? | Decay-weighted engagement score over time |

These combine into the **Creative Yield Score (CYS)** — computed per artifact, per session, and per player lifetime.

### Three Earnings Channels

**Channel 1 — Immediate: Session Pulse**

```
Session Credits = (Originality × 0.25) + (Intent Coherence × 0.20)
               + (Craft Depth × 0.20) + (World Impact × 0.20)
               + (Temporal Durability × 0.15)
               × Session Duration Weight (capped 4hrs, diminishing after 2hrs)
```

Duration weight prevents grinding. One exceptional artifact in 45 minutes earns more than 6 hours of forgettable output.

**Channel 2 — Deferred: World Impact Royalties**
When your artifact influences another player's behaviour (they build near it, reference it, trade it, the world generates events around it), you receive passive LOKA Credit royalties. Continues as long as World Impact lasts — months or years.

- Deepest earners are not grinders but players whose output becomes *infrastructure*
- A musician whose composition plays in a settlement 200 players pass through earns every week
- An architect whose city layout becomes a generational template earns for years

**Channel 3 — Real-World: Chain Registration Earnings**
- Primary sale: 90% to creator, 10% to LOKA treasury
- Secondary sales: ERC-2981 royalty — 7.5% back to original creator perpetually
- Cross-timeline licensing: stablecoin license fees, directly withdrawable
- Real-world withdrawal available for chain-registered earnings only

---

## Creative Level Architecture

Public, AI-maintained credential — affects what the world offers, not what you're allowed to do:

```
Level 0 — Wanderer      (baseline, all new players)
Level 1 — Maker         (first artifact with CYS > 40)
Level 2 — Craftsman     (10 artifacts averaging CYS > 55)
Level 3 — Artisan       (World Impact on 3+ artifacts simultaneously)
Level 4 — Luminary      (artifact achieving Temporal Durability > 90 days)
Level 5 — Progenitor    (your output influenced another player's Creative Level rise)
Level 6 — Mythwright    (your work referenced in LOKA's annual Chronicle)
Level 7 — Unnamed       (AI determined this. No criteria published.)
```

Each level increase:
- Raises base multiplier on all future CYS calculations
- Unlocks new creative tools and material palettes
- Makes Pantheon ranking visible in additional categories
- Level 5+: **Progenitor Dividend** — micro-share of all chain-registered transaction revenue

**Level 7 cannot be pursued.** Any player appearing to game their way there gets demoted to Level 4. Optimization behaviour = evidence of inauthenticity.

---

## Anti-Gaming Architecture

**1. Intent Coherence as master signal**
PIM knows what you actually want. Creating for yield (not expression) collapses Intent Coherence → CYS drops dramatically. You cannot fake wanting something.

**2. Peer Validation for Chain Registration**
Blind review by three randomly selected Artisan-level+ players who stake LOKA Credits on their assessment. Collusion is statistically detectable — colluders share similar low-originality profiles, flagged by AI.

**3. World Impact is graph-verified**
Impact measured by downstream player decision changes in the event graph. Bot farms produce statistically identical, correlated behaviour — trivially detectable in Neo4j lineage graph. Real influence produces chaotic, organic patterns.

**4. Temporal Durability has a 21-day minimum**
No artifact can score above 50 on Temporal Durability in less than 21 real-world days. No shortcuts. The world must actually age around your work.

---

## Economic Reality Check

| Creative Level | Realistic Earnings |
|---|---|
| Level 3 (Artisan), 5hrs/week | Sufficient Credits for Tier 3 AI + 2 timeline passes without subscription; $50–$300/month real-world royalties |
| Level 6 (Mythwright), deep World Impact | Structurally possible to support oneself from LOKA earnings (~0.1% of active playerbase) |

> *The system does not guarantee income. It guarantees that genuine creative value will not go uncompensated.*

---

## Key Takeaways

- The CYS is the most important anti-P2E-failure mechanism: it rewards irreplaceability, not grind — making it nearly impossible to automate
- Three-channel structure (immediate, deferred, real-world) creates different risk/reward profiles for different player types
- World Impact Royalties are where the deepest value accumulates — players who become infrastructure earn passively for years
- Proof of Intent mechanism prevents NFT flooding by requiring traceable creative lineage
- Level 7 actively penalizing pursuit is the design signal that the system values truth over performance

## Related Articles
- [[game-concept]] — Monetization framework and Pantheon overview
- [[intent-engine]] — How Intent Coherence signal is generated
- [[social-and-ancestry]] — Co-owned artifacts from Expeditions
- [[technical-architecture]] — Blockchain stack (Base/Arbitrum, ERC-2981, Reservoir)
