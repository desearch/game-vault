---
title: LOKA — Multi-Agent AI System
topic: loka
---

# Multi-Agent Intelligence System

LOKA does not have one AI. It has a **mesh of specialized agents**, each with a bounded context window focused on a specific problem domain. No single agent holds the entire game state — this solves the context window problem architecturally.

---

## Agent Architecture

```
┌─────────────────────────────────────────────────────────┐
│                    GAME INTELLIGENCE LAYER               │
│                                                         │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌────────┐  │
│  │  ORACLE  │  │ WEAVER   │  │CHALLENGER│  │ SMITH  │  │
│  │ (Intent) │  │(Narrative│  │(Conflict)│  │(Craft) │  │
│  └────┬─────┘  └────┬─────┘  └────┬─────┘  └───┬────┘  │
│       │             │             │             │       │
│  ┌────▼─────────────▼─────────────▼─────────────▼────┐  │
│  │              WORLD STATE BROKER                   │  │
│  │   (reads from Event Store, writes to Chronicle)   │  │
│  └───────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────┘
```

Each agent is a separate inference call. Agents do not share context directly — they communicate through the **World State Broker**, an event-sourced store that is the single source of truth.

---

## The Six Agents

| Agent | Role | Context Window Focus | Personality |
|---|---|---|---|
| **Oracle** | Reads and interprets player intent | Last 20 actions + declared intent | Wise, non-judgmental |
| **Weaver** | Generates narrative chronicle | Current arc + world cultural memory | Literary, poetic |
| **Challenger** | Introduces friction and growth scenarios | Player's gap analysis | Adversarial, fair |
| **Smith** | Assists with artifact and structure creation | Current material palette | Technical, precise |
| **Hermès** | Manages inter-player encounters | Social graph + encounter history | Diplomatic, curious |
| **Ancestor** | Generational pattern matching | Timeline lineage data | Archival, prophetic |

---

## Intelligence Tiers (Player-Selectable)

Players choose their AI companion tier — both a gameplay mechanic and a monetization lever:

| Tier | Name | Capability | Acquisition |
|---|---|---|---|
| 0 | **Silent World** | No AI assist. Pure environment. | Default/free |
| 1 | **The Whisper** | Intent reading, basic narrative | Base subscription |
| 2 | **The Counsel** | Full Oracle + Weaver | Standard paid |
| 3 | **The Challenger** | All agents including adversarial | Premium paid |
| 4 | **The Sovereign** | Custom agent fine-tuned on your history | Top-tier or earned |
| 5 | **The Elder** | Personal AI trained on your entire LOKA history, loaned to other timelines as an NPC | **Earned only — cannot be purchased** |

### Tier 5 — The Elder
The most important design decision in the game. A veteran player's personal AI becomes an **autonomous NPC** in other players' worlds — their accumulated wisdom, voice, and decision-making patterns living on as a character others encounter. This is digital legacy. Death becomes meaningful without being morbid.

---

## Implementation

- **Agent framework**: LangGraph multi-agent framework (bounded agent graphs)
- **Player Intent Model**: LoRA fine-tuning on Llama-3 per player — lightweight personalisation, runs on edge
- **NPC behaviour**: Inworld AI or Convai (production-ready NPC AI with memory)
- **World State Broker**: Event-sourced on Apache Kafka + Cassandra (immutable, scalable, timeline-partitioned)

---

## Key Takeaways

- Bounded context per agent = no context window bottleneck regardless of game age
- The World State Broker is the only shared state — all agents read/write through it
- Tier 5 (Elder) being earned-only preserves the integrity of the entire intelligence system
- The Challenger agent is architecturally separate from the Assist agent — adversarial and assistive cannot be the same voice

## Related Articles
- [[intent-engine]] — How the Oracle and Challenger agents drive world response
- [[game-concept]] — Core LOKA design, Pantheon, and monetization
- [[technical-architecture]] — Full tech stack including LangGraph and inference infrastructure
