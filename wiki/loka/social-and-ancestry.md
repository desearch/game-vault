---
title: LOKA — Social Architecture & Generational Ancestry
topic: loka
---

# Social Architecture & Generational Ancestry

---

## Social Architecture

### Encounter Mechanics

Players do not "friend" each other — they **encounter** each other. When two Private Bubbles approach within the Resonance Zone:

1. Game checks **intent compatibility** — are current vectors complementary or orthogonal?
2. Generates an **encounter event** — a shared circumstance (storm, discovery, structural collapse) that makes interaction arise naturally
3. Gives both players **encounter options** without breaking immersion: ignore, acknowledge, join, trade, challenge

No chat UI. Communication is through:
- **In-world speech** (spatial audio)
- **Artifact exchange**
- **Collaborative building**
- Timeline-appropriate translation (TL-1850 uses pidgin/trade symbols, not real-time AI translation)

### Encounter Depth

| Type | Duration | Asset Sharing | Chronicle Impact | World Impact |
|---|---|---|---|---|
| **Casual Encounter** | Minutes | None | Footnote | None |
| **Expedition** | Days–weeks | Temporary | Chapter | Local |
| **Bound Companion** | Permanent | Full | Co-author | Civilizational |

### Collaborative Journeys (Expeditions)

Formal game state where two or more players merge Private Bubbles into a **Shared Expedition**:
- World generates scenarios requiring multiple complementary skills
- Each player's intent is weighted; environment serves both
- Artifacts created are **co-owned on-chain**
- Chronicle records both players, creating a shared legend
- Can be dissolved at any moment — no penalty; *"And on the third cycle, the wanderer chose a different horizon."*

### Bound Companions

The deepest social bond in LOKA:
- Two players write a **Covenant** — a declared intent the world holds as a sacred contract
- World generates scenarios designed to test, reward, and commemorate the covenant
- Full asset sharing and civilizational-scale world impact potential

---

## Generational Ancestry System

Historically unprecedented — designed for multi-decade play.

### How It Works

Every action is stored as an **Intent Event** in an immutable log:

```json
{
  "player_id": "hash",
  "timeline": "TL-2024",
  "timestamp": "epoch",
  "location": [x, y, z],
  "intent_vector": "[256-dim float array]",
  "action_type": "build | explore | create | encounter | challenge",
  "outcome": "...",
  "artifact_ids": [...]
}
```

When a new player joins, the system runs a **Lineage Match** — comparing their early intent vector to historical record. If similarity exceeds a threshold across meaningful dimensions, they are assigned **Generational Ancestry**.

The game then:
- Surfaces ancestors' ruins, artifacts, and cultural traces more prominently
- Allows inheriting (not stealing) abandoned structures from ancestors
- Gives NPCs "memories" of players who share the lineage
- Their Elder AI (Tier 5) eventually becomes an ancestor NPC for their descendants

### Cross-Timeline Ancestry

When a player holds multiple timeline passes, their intent vector creates a **Cross-Timeline Signature**. If another player — in a different timeline, even 50 real-world years later — produces a similar signature:

- World generates **mythological echoes**: stories, artifacts, and landscape features referencing the original without direct attribution
- The original player becomes *legend* without ever knowing it

### Data Model

- **Neo4j graph database** — ancestry is a graph problem
- **ClickHouse** — columnar analytics on billions of intent events
- **TimescaleDB** — temporal queries across timeline eras

---

## Key Takeaways

- No friend lists, no social feeds — all social structure emerges from intent compatibility and proximity
- Bound Companions are the rarest, deepest bond — the Covenant mechanic makes relationships mechanically meaningful, not just cosmetic
- Generational Ancestry is what makes LOKA historically unprecedented — players become myths without knowing it
- Cross-timeline ancestry means playing in multiple timelines creates a signature that echoes across real-world decades
- The immutable intent event log is both the game's memory and its civilizational history

## Related Articles
- [[game-concept]] — World architecture, Private Bubble, and Encounter Zone layout
- [[intent-engine]] — How intent vectors are generated and maintained
- [[economy-and-earnings]] — Co-owned artifacts and royalty splits from Expeditions
- [[technical-architecture]] — Neo4j, Kafka, ClickHouse stack powering ancestry
