---
title: LOKA — Technical Reference Document
topic: loka
audience: All Stakeholders — Engineers, Investors, Product, Research, Operations
version: 1.0 | April 2026
---

# LOKA — Technical Reference Document
### Version 1.0 | April 2026

> This document is the single technical source of truth for LOKA. It complements the [[business-proposal]] and all other wiki articles. Investors and non-technical stakeholders should read §1–§3. Engineers and technical partners should read all sections. Each section opens with a plain-language summary before going deeper.

---

## Table of Contents

1. [How to Read This Document](#1-how-to-read-this-document)
2. [System Overview — The Big Picture](#2-system-overview)
3. [Architecture Principles](#3-architecture-principles)
4. [Component Architecture](#4-component-architecture)
5. [The Player Intent Model — Deep Dive](#5-the-player-intent-model)
6. [The Agent Mesh — Deep Dive](#6-the-agent-mesh)
7. [The World State Broker](#7-the-world-state-broker)
8. [Data Architecture & Key Schemas](#8-data-architecture--key-schemas)
9. [World Rendering & Spatial Simulation](#9-world-rendering--spatial-simulation)
10. [The Economy Layer — Technical Design](#10-the-economy-layer)
11. [Infrastructure by Phase](#11-infrastructure-by-phase)
12. [Security Architecture](#12-security-architecture)
13. [Performance Requirements & SLAs](#13-performance-requirements--slas)
14. [Development Workflow](#14-development-workflow)
15. [Testing Strategy](#15-testing-strategy)
16. [Observability & Operations](#16-observability--operations)
17. [Phase-by-Phase Technical Roadmap](#17-phase-by-phase-technical-roadmap)
18. [Technical Risk Register](#18-technical-risk-register)
19. [Glossary](#19-glossary)

---

## 1. How to Read This Document

This document is structured in layers. Each major section opens with a **plain-language summary** box before diving into technical specifics. The depth increases as you go further into each section.

**Read §1–§3** if you want to understand what LOKA is technically, what makes it different, and why the architecture is designed the way it is — without needing an engineering background.

**Read §4–§10** if you are evaluating the technical feasibility, making build/buy decisions, or joining the founding team.

**Read §11–§18** if you are responsible for infrastructure, security, operations, QA, or technical risk assessment.

**§19 (Glossary)** explains every technical term used across all LOKA documents. Refer to it freely.

Every section links to the relevant focused wiki article for additional detail. This document provides the connective tissue — how everything fits together.

---

## 2. System Overview

> **Plain language:** LOKA is built from five interconnected systems. The Player Intent Model reads what you want. The Agent Mesh responds to it. The World Engine renders the result. The Data Layer remembers everything permanently. The Economy Layer lets you own and earn from what you create. All five talk to each other through a single shared memory called the World State Broker.

### 2.1 The Five Systems

```
╔══════════════════════════════════════════════════════════════════════╗
║                         LOKA SYSTEM MAP                             ║
╠══════════════════════════════════════════════════════════════════════╣
║                                                                      ║
║   PLAYER                                                             ║
║   ┌────────────────────────────────────────────────────────────┐    ║
║   │  VR Headset / PC Client (Unreal Engine 5)                  │    ║
║   │  → speech, movement, HRV, micro-expression, controller     │    ║
║   └────────────────────┬───────────────────────────────────────┘    ║
║                        │  raw signals                                ║
║                        ▼                                             ║
║   ┌─────────────────────────────────────────────────────────────┐   ║
║   │  1. PLAYER INTENT MODEL (PIM)                               │   ║
║   │  LoRA fine-tuned Llama-3, per player, edge-deployed         │   ║
║   │  Output: 256-dim intent vector, refreshed every 30 seconds  │   ║
║   └────────────────────┬────────────────────────────────────────┘   ║
║                        │  intent vector                              ║
║                        ▼                                             ║
║   ┌─────────────────────────────────────────────────────────────┐   ║
║   │  2. AGENT MESH (LangGraph)                                  │   ║
║   │  Oracle · Weaver · Challenger · Smith · Hermès · Ancestor   │   ║
║   │  Each agent: separate inference, bounded context, async     │   ║
║   └──────────────────┬──────────────────────────────────────────┘   ║
║                      │  world directives                             ║
║          ┌───────────┼─────────────────┐                            ║
║          ▼           ▼                 ▼                            ║
║   ┌──────────┐  ┌──────────┐  ┌───────────────────────────────┐    ║
║   │ 3. WORLD │  │ 4. DATA  │  │ 5. ECONOMY LAYER              │    ║
║   │ ENGINE   │  │ LAYER    │  │ On-chain · Marketplace · CYS  │    ║
║   │ UE5 +    │  │ Kafka ·  │  └───────────────────────────────┘    ║
║   │ SpatialOS│  │ Cassandra│                                        ║
║   └──────────┘  │ ClickHse │                                        ║
║                 │ Neo4j    │                                        ║
║                 │ TimescaleDB                                        ║
║                 └──────────┘                                        ║
║                        ▲                                             ║
║   ┌─────────────────────────────────────────────────────────────┐   ║
║   │  WORLD STATE BROKER (Event Store)                           │   ║
║   │  Apache Kafka + Cassandra — immutable, timeline-partitioned │   ║
║   │  Every agent, every component reads & writes through here   │   ║
║   └─────────────────────────────────────────────────────────────┘   ║
╚══════════════════════════════════════════════════════════════════════╝
```

### 2.2 A Single Player Session — End to End

To make this concrete, here is what happens technically during a single LOKA session:

**Step 1 — Signal capture (client-side)**
The UE5 client captures voice input (Whisper STT), movement and controller data, optional biometric signals (HRV, face tracking), and combines these with the player's historical session data. This is processed locally on the client before any network call.

**Step 2 — Intent extraction (edge + cloud)**
The Player Intent Model — a lightweight LoRA adapter running locally — takes the signal stream and produces a 256-dimensional float array called the intent vector. This runs every 30 seconds during play, not on every frame. The intent vector is the only representation of the player that travels to the server.

**Step 3 — World State query (server)**
The intent vector is sent to the World State Broker. The Broker queries the Cassandra event store for the relevant timeline's recent state: terrain conditions, cultural drift, active NPCs in range, nearby player signatures, recent events in the Resonance Zone.

**Step 4 — Agent invocations (async, parallel)**
LangGraph orchestrates parallel calls to the relevant agents:
- Oracle: receives intent vector + last 20 actions → produces world response directives (terrain weight adjustments, NPC personality parameters)
- Weaver: receives current arc state → appends to the player's chronicle
- Challenger (if player has Tier 3+): receives gap analysis → optionally triggers a pressure event
- Smith (if player is actively crafting): receives material palette + intent → suggests or generates artifact components

**Step 5 — World directive application (game server)**
Directives from agents are sent to the SpatialOS game server as world update instructions. The game server applies them to the terrain generation weights, NPC behaviour scripts, and event queues for the player's zone.

**Step 6 — Event recording (immutable)**
Every action the player takes — movement, creation, encounter, dialogue — is written as an Intent Event to the Kafka stream and durably stored in Cassandra. This is the permanent record of the player's existence in LOKA.

**Step 7 — Rendered delivery (client)**
The UE5 client receives world state updates from the game server (via SpatialOS), renders the result, and presents it to the player. In VR, this is at 72–90fps with spatial audio.

Total latency from intent update to world response: **target < 2 seconds** for terrain shaping, **< 500ms** for immediate NPC reactions, **< 100ms** for physics and rendering.

---

## 3. Architecture Principles

> **Plain language:** These are the seven rules every technical decision in LOKA is measured against. If a proposed solution violates any of them, it is wrong regardless of how clever it seems.

### Principle 1 — The World State Broker is the only truth

No component in LOKA holds its own state copy that is authoritative. All state lives in the World State Broker (Kafka + Cassandra). Components read from it, write to it, and derive their behaviour from it. This means any component can fail and be restarted without data loss. It also means the entire history of LOKA is auditable from a single source.

**Violation example:** An agent caching player state locally and making decisions from that cache without querying the Broker. This creates split-brain scenarios and destroys ancestry tracking.

### Principle 2 — Agents have bounded, focused context

Each agent knows its specific job and accesses only the data relevant to that job. No agent has access to the full game state. This is not a security restriction — it is an architectural necessity. Context window limits are managed by keeping each agent's domain narrow, not by increasing context window size.

**Violation example:** A "super-agent" that handles intent, narrative, and economy in one inference call. This will fail at scale as history grows, and it conflates concerns that should remain separate.

### Principle 3 — Intent is the primary key

Every significant operation in LOKA is tagged with the player's intent vector at the time it occurred. The intent vector is not metadata — it is the primary key for understanding the operation. A build action without its intent signature is incomplete. An artifact without its creation intent is unverifiable.

**Violation example:** Storing artifacts with only their material and geometric properties, treating intent as optional metadata to be added later. Late intent attribution is gameable.

### Principle 4 — Events are immutable

Nothing in the Kafka stream or Cassandra event store is deleted or modified. Events are appended. This makes LOKA's history tamper-proof and makes the ancestry system reliable. It also means the data set grows indefinitely — storage cost is a known, manageable variable.

**Violation example:** Deleting a player's event history when they request account deletion. The correct approach is to anonymize the data (replace player_id with a cryptographic hash) while preserving the event structure for ancestry integrity.

### Principle 5 — Inference cost scales sublinearly with users

Every architecture decision about AI inference must be evaluated on a per-user-per-session cost basis, not just total cost. A design that costs ₹100 per session at 100 users costs ₹10,000 per session at 1,000 users. The correct direction is always toward local inference (edge PIM), self-hosted models (A100 cluster), and caching of world cultural state to reduce regeneration.

**Violation example:** Using Claude API for all agent calls at every scale. This works at 100 users. At 10,000 users it becomes the dominant cost item and destroys unit economics.

### Principle 6 — The world has no single owner

No game developer at LOKA Studios can edit the world state directly. World changes flow through the same agent pipeline and event store as player actions. This is a governance decision as much as a technical one — it ensures the world's history is authentic and not manipulable by the company that built it.

**Violation example:** A developer writing a direct database update to "fix" a broken world state. The correct process is to inject a correction event through the official pipeline with appropriate labeling.

### Principle 7 — Privacy by architecture, not policy

Player identity is pseudonymous throughout the system. The player's real identity (name, email, payment info) lives only in the authentication layer and is never written to the event store. The event store uses cryptographic hashes as player identifiers. The PIM is trained on behavioural signals, not on personal identity data. Anonymization of research data is enforced at the data access layer, not by analyst discipline.

**Violation example:** Writing a player's email address or real name into an Intent Event for debugging convenience.

---

## 4. Component Architecture

> **Plain language:** LOKA has six major software components. Each is built independently, deployed independently, and communicates through the shared event store. This means each can be scaled, updated, or replaced without taking down the whole system.

### 4.1 Component Map

```
┌─────────────────────────────────────────────────────────────────────┐
│                        LOKA COMPONENTS                              │
│                                                                     │
│  ┌──────────────┐    ┌──────────────┐    ┌───────────────────────┐ │
│  │  1. CLIENT   │    │  2. INTENT   │    │  3. AGENT MESH        │ │
│  │  UE5 + OpenXR│───▶│  SERVICE     │───▶│  LangGraph            │ │
│  │  VR / PC     │    │  PIM + embed │    │  6 agents             │ │
│  └──────────────┘    └──────────────┘    └───────────┬───────────┘ │
│                                                       │             │
│  ┌──────────────┐    ┌──────────────┐    ┌───────────▼───────────┐ │
│  │  6. ECONOMY  │    │  5. DATA     │    │  4. WORLD ENGINE      │ │
│  │  On-chain    │◀──▶│  LAYER       │◀───│  SpatialOS + UE5 srv  │ │
│  │  Marketplace │    │  Kafka+Store │    │  Terrain + NPCs       │ │
│  └──────────────┘    └──────────────┘    └───────────────────────┘ │
│                              ▲                                      │
│            ╔═════════════════╩═════════════════╗                   │
│            ║     WORLD STATE BROKER            ║                   │
│            ║  Single source of truth for all   ║                   │
│            ╚═══════════════════════════════════╝                   │
└─────────────────────────────────────────────────────────────────────┘
```

### 4.2 Component Responsibilities

**Component 1 — Client (UE5 + OpenXR)**
- Renders world at 72–90fps in VR, 60fps+ on flat screen
- Captures player input: voice (Whisper local), movement, controller, biometrics
- Runs the PIM inference locally (LoRA adapter on device GPU)
- Manages Private Bubble spatial partitioning
- Handles offline-capable local state for resilience

**Component 2 — Intent Service**
- Receives raw signals from client every 30 seconds
- Runs the server-side intent validation (cross-checks edge PIM with server-side model)
- Maintains the rolling intent vector history per player
- Exposes the current intent vector to all agents via the World State Broker
- Handles PIM training pipeline: batches session data, triggers LoRA fine-tuning jobs

**Component 3 — Agent Mesh (LangGraph)**
- Orchestrates all six agents (Oracle, Weaver, Challenger, Smith, Hermès, Ancestor)
- Manages agent invocation schedule (Oracle every 30s, Weaver every 5 min, Challenger on trigger, etc.)
- Handles agent failure gracefully — world continues without a specific agent's output
- Publishes agent directives to the World State Broker as typed events

**Component 4 — World Engine**
- Spatial simulation server built on SpatialOS or AWS GameLift
- Applies agent directives as world state mutations (terrain weights, NPC parameters, event queue)
- Manages zone-based spatial partitioning (players only receive updates relevant to their position)
- Handles real-time physics, collision, lighting (UE5 Lumen + Nanite)
- Publishes every player action back to the World State Broker as Intent Events

**Component 5 — Data Layer**
- Kafka: ingests all events (100K+ events/second at scale)
- Cassandra: durable event store, timeline-partitioned
- ClickHouse: analytics and CYS computation
- Neo4j: ancestry graph traversal
- TimescaleDB: time-series world state snapshots
- Feast: ML feature store for PIM training

**Component 6 — Economy Layer**
- LOKA Credits ledger (off-chain, custodial via Sequence/Privy)
- CYS computation engine (reads from ClickHouse, writes scores to Cassandra)
- Chain registration pipeline (Proof of Intent verification → L2 minting)
- Marketplace integration (Reservoir Protocol SDK)
- Royalty distribution (ERC-2981 smart contract event listener → credit distribution)

---

## 5. The Player Intent Model

> **Plain language:** Every player in LOKA has their own personal AI — a small model that learns over time what that specific person genuinely wants. It doesn't read your mind. It reads your patterns. After months of play, it knows the difference between you exploring because you're curious and you exploring because you're bored and avoiding what you actually came here to do. This distinction is what drives the world's response.

### 5.1 Architecture

The PIM is a **LoRA (Low-Rank Adaptation) fine-tuned adapter** applied on top of a base Llama-3-8B model. It is not a separate large model — it is a small set of weight adjustments (~16MB per player) that steer the base model's behaviour toward understanding a specific player's intent patterns.

```
┌─────────────────────────────────────────────────────────────┐
│                  PIM ARCHITECTURE                           │
│                                                             │
│  Base Model: Llama-3-8B (shared across all players)        │
│  ┌────────────────────────────────────────────────────┐    │
│  │  Frozen base weights (not updated per player)      │    │
│  └────────────────────┬───────────────────────────────┘    │
│                       │ +                                   │
│  ┌────────────────────▼───────────────────────────────┐    │
│  │  LoRA Adapter (~16MB per player)                   │    │
│  │  Updated after every session with new signal data  │    │
│  └────────────────────┬───────────────────────────────┘    │
│                       │                                     │
│                       ▼                                     │
│  Output: 256-dimensional intent vector                      │
│  Refreshed: every 30 seconds during active play             │
│  Deployed: edge (client GPU) for real-time inference        │
│  Trained: server-side, after each session                   │
└─────────────────────────────────────────────────────────────┘
```

**Why LoRA:** A full fine-tune of a large model per player would require gigabytes of storage per player and hours of compute per training cycle. LoRA adapters achieve comparable personalisation with 16MB of storage and training times of under 10 minutes on a single A100 GPU — making per-player models economically viable at scale.

**Why edge deployment:** Intent extraction must be real-time. A round-trip to a cloud server (even with low-latency infrastructure) adds 50–200ms of latency. Running the LoRA adapter on the client's GPU (which exists in both VR headsets and gaming PCs) eliminates this latency entirely. The adapter is small enough to reside in VRAM permanently.

### 5.2 The Four Input Signals

| Signal | Capture Method | Update Frequency | Weight in Vector |
|---|---|---|---|
| **Explicit declaration** | Whisper STT → tokenized | On statement | High — player directly stated intent |
| **Behavioural pattern** | Movement/action log | Every 5 seconds | Medium — actions speak louder |
| **Emotional signal** | HRV (wearable), face tracking (VR cam), voice tonality | Continuous | Low — contextual modifier |
| **Historical trajectory** | Aggregated prior session embeddings | Once per session start | High — established pattern |

The emotional signal is weighted lower not because it is unreliable, but because it is the most easily confounded. A player who is anxious in real life brings that anxiety to LOKA — the world should not treat genuine external stress as an intent to seek conflict. The behavioural pattern and historical trajectory signals correct for this.

### 5.3 The Cold Start Problem

New players have no history. The PIM initializes from the **onboarding event** — the player's answer to *"What do you want?"* This single statement, combined with their movement patterns in the first 10 minutes (how fast they move, what they look at, what they touch first), provides sufficient signal for a coarse intent vector that the world can respond to meaningfully.

Cold start precision: approximately 40% of the intent dimensions are accurately initialized from onboarding. The remaining 60% converge over the first 3–5 sessions of play. Players typically do not notice the first-session imprecision because the world is still new to them and any response feels meaningful.

After 10 sessions: 85%+ dimensional accuracy.
After 50 sessions: 95%+ dimensional accuracy.
After 200+ sessions: the PIM has a more complete model of the player's intent patterns than the player has of themselves.

### 5.4 Intent Coherence — The Anti-Gaming Signal

Intent Coherence is the cosine similarity between the player's PIM-predicted intent vector and the intent signature embedded in their artifact at creation time. It answers: *is this artifact an expression of what this player actually wants, or are they producing it for external reasons (yield, leaderboard position, social pressure)?*

```
Intent Coherence Score = cosine_similarity(
    PIM_vector_at_creation_time,
    artifact_intent_signature
)

Range: -1.0 (completely contrary to intent) to 1.0 (perfect expression of intent)
CYS uses: max(0, score) — negative scores treated as zero, not negative
```

A player producing artifacts for CYS points rather than genuine expression will show a consistent pattern: their PIM vector (which evolves based on authentic behaviour signals) will drift away from their artifact signatures (which are shaped by what earns rather than what they want). This drift is detectable after 5–10 sessions and triggers a gradual CYS suppression.

### 5.5 Privacy Architecture of the PIM

The LoRA adapter stores patterns, not raw data. It does not contain any personally identifiable information. It cannot be decoded to reconstruct specific statements or actions — it is a statistical compression of behavioural tendencies.

The training data (raw session signals) is retained server-side for 90 days for model improvement, then deleted. The resulting LoRA adapter is retained for the life of the account. If a player requests data deletion:
- Training data: immediately purged
- LoRA adapter: immediately deleted
- Intent Events in the event store: player_id replaced with cryptographic tombstone hash — the events are preserved for ancestry integrity but are permanently unattributable to the player

---

## 6. The Agent Mesh

> **Plain language:** Six AI agents run alongside every player in LOKA, each responsible for one aspect of the experience. They never talk directly to each other — they each read the shared world record and write their responses back to it. Think of it like six specialists in a hospital: the cardiologist, neurologist, and surgeon each have their expertise, they all read from the same patient chart, and they each write their notes back to it. They don't need to be in the same room simultaneously.

### 6.1 Agent Invocation Schedule

Each agent runs on a different cadence depending on how time-sensitive its output is:

| Agent | Trigger | Approx. Frequency | Output Type |
|---|---|---|---|
| **Oracle** | Intent vector update | Every 30 seconds | Terrain weights, NPC parameters |
| **Weaver** | Significant player action | Every 5 minutes | Chronicle entry, in-world document |
| **Challenger** | Behavioural gap threshold exceeded | Every 3–7 sessions | Pressure event injection |
| **Smith** | Player initiates crafting | On demand | Artifact component suggestions, material combinations |
| **Hermès** | Two players enter Resonance Zone | On proximity trigger | Encounter event, interaction options |
| **Ancestor** | New session start + lineage match check | Once per session | Ancestry assignments, NPC memory injection |

### 6.2 Agent Communication Protocol

All inter-agent communication is indirect — through the World State Broker. No agent holds a reference to another agent.

```
Agent invocation flow:

1. LangGraph Orchestrator reads intent vector from World State Broker
2. Orchestrator determines which agents to invoke (based on schedule + triggers)
3. Each agent is invoked as an independent async task
4. Agent queries World State Broker for its required context slice
5. Agent produces output (directive, event, chronicle entry)
6. Agent writes output back to World State Broker as a typed event
7. World Engine reads relevant events from Broker and applies directives
```

**Context slices per agent (what each agent reads from the Broker):**

```
Oracle context slice:
  - Player intent vector (current + previous 10 snapshots)
  - Last 20 player actions with their intent signatures
  - Current timeline cultural state (aggregate of all player actions in region)
  - Available terrain generation parameters for player's zone

Weaver context slice:
  - Player's existing chronicle (last 3 chapters)
  - Current arc state (what narrative thread is active)
  - Significant events from last 5-minute window
  - Timeline cultural memory (notable events from broader world history)
  - Player's Creative Level and artifact portfolio summary

Challenger context slice:
  - Player's declared intent (from most recent onboarding or re-declaration)
  - Behavioural pattern from last 10 sessions
  - Computed gap: cosine distance between stated intent and actual action distribution
  - Available pressure event templates for current timeline and zone

Smith context slice:
  - Current material palette (what resources player has)
  - Timeline-appropriate material taxonomy
  - Player's artifact history (previous creation styles, CYS scores)
  - Requested artifact type (from player action)

Hermès context slice:
  - Both players' intent vectors
  - Intent compatibility score (cosine similarity between vectors)
  - Both players' social histories (encounter counts, expedition history)
  - Available encounter event templates
  - Timeline-appropriate communication modalities

Ancestor context slice:
  - New player's early intent vector (first 3 sessions)
  - Historical lineage index for the timeline (pre-computed similarity clusters)
  - Candidate ancestor profiles above similarity threshold
  - Available NPC memory templates for ancestor injection
```

### 6.3 Failure Handling

Each agent failure is isolated. The system degrades gracefully:

| Agent Failure | Player Experience | Recovery |
|---|---|---|
| Oracle down | Terrain generation uses last cached weights | Transparent — regenerates next cycle |
| Weaver down | Chronicle pauses, no new entries | Retrospectively filled when service recovers |
| Challenger down | No new friction events | No impact — player experiences smoother session |
| Smith down | Crafting UI shows basic options only | Partial experience — player notified |
| Hermès down | Encounters still happen, no compatibility check | Generic encounter events substituted |
| Ancestor down | No new ancestry assignments | Queued and processed on recovery |

The only agent whose failure has immediate player-visible impact is Smith during active crafting. All others can fail silently and recover without the player noticing.

### 6.4 The Elder AI — Technical Design

The Tier 5 Elder AI is the most technically complex component in LOKA. It is a per-player model that becomes an autonomous NPC in other players' worlds.

**How it is created:**
When a player reaches Tier 5 eligibility (earned, not purchased), a distillation process runs:
1. Their complete LoRA adapter (PIM) is taken as the foundation
2. Their full chronicle (Weaver output over their entire play history) is summarized into a personality tensor
3. Their artifact portfolio's intent signatures are aggregated into a creative fingerprint
4. A specialized NPC agent is instantiated using these three components, running on top of the same Llama-3 base model

**How it is deployed:**
The Elder AI NPC is added to the Ancestor agent's database for that player's timeline. When the Ancestor agent detects that a new player's intent vector matches the Elder's creator's profile, it instantiates the NPC in that player's world — with appropriate narrative framing.

**What the Elder AI knows:**
It knows the patterns and tendencies of its creator. It does not know the creator's real identity. It cannot reveal specific game events (it has no episodic memory). It can exhibit the creator's decision-making style, creative preferences, and approach to challenges — because these are encoded in the LoRA weights and personality tensor, not in raw text.

**Privacy guarantee:**
The Elder AI cannot be interrogated to reconstruct its creator's identity or specific gameplay history. The distillation process is one-way: personality patterns are preserved, raw memories are not.

---

## 7. The World State Broker

> **Plain language:** The World State Broker is LOKA's permanent memory. Everything that has ever happened in the game — every player action, every artifact created, every encounter, every world event — is written here in a format that can never be altered. It is the foundation of the Generational Ancestry System and the Chronicle. If LOKA's servers went offline tomorrow and came back online a year later, all of this history would still be intact.

### 7.1 Technology Stack

```
┌─────────────────────────────────────────────────────────────────┐
│                   WORLD STATE BROKER                            │
│                                                                 │
│  Ingestion Layer                                                │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │  Apache Kafka (MSK)                                     │   │
│  │  Topics: intent-events, agent-directives, world-events, │   │
│  │          artifacts, encounters, economy-events          │   │
│  │  Throughput: 100K+ messages/second at Phase 3 scale     │   │
│  │  Retention: 7 days hot (Kafka) → permanent (Cassandra)  │   │
│  └──────────────────────────┬──────────────────────────────┘   │
│                             │                                   │
│  Storage Layer              ▼                                   │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │  Apache Cassandra                                       │   │
│  │  Partition key: (timeline_id, shard_date)               │   │
│  │  Clustering key: (player_id_hash, timestamp)            │   │
│  │  Replication: 3 nodes minimum, multi-AZ                 │   │
│  │  Consistency: LOCAL_QUORUM for writes (durability)      │   │
│  │              ONE for reads (low latency)                │   │
│  └──────────────────────────┬──────────────────────────────┘   │
│                             │                                   │
│  Analytics Layer            ▼                                   │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │  ClickHouse (Kafka consumer)                            │   │
│  │  Used for: CYS computation, World Impact scoring,       │   │
│  │            Pantheon rankings, research data exports     │   │
│  │  Lag: near-real-time (< 30 second latency from Kafka)   │   │
│  └─────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────┘
```

### 7.2 Kafka Topic Design

```
Topic: loka.intent-events
  Partition key: player_id_hash
  Message schema: IntentEvent (see §8)
  Consumers: Intent Service, ClickHouse, Cassandra sink

Topic: loka.agent-directives
  Partition key: zone_id
  Message schema: AgentDirective
  Consumers: World Engine (SpatialOS bridge)

Topic: loka.artifacts
  Partition key: timeline_id
  Message schema: ArtifactEvent
  Consumers: Economy Layer (CYS engine), ClickHouse, Cassandra sink, 
             Chain Registration Pipeline

Topic: loka.encounters
  Partition key: zone_id
  Message schema: EncounterEvent
  Consumers: Social Layer, Chronicle writer, ClickHouse

Topic: loka.economy-events
  Partition key: player_id_hash
  Message schema: EconomyEvent
  Consumers: Credits ledger, Chain Registration Pipeline, Royalty Distributor
```

### 7.3 The Event Sourcing Pattern

LOKA uses **Event Sourcing** — the current state of any object (a player, an artifact, a region of the world) is derived by replaying its event history from the beginning, not by reading a single mutable record.

**Why this matters for LOKA specifically:**
- The entire Generational Ancestry System depends on having the complete event history of every player, fully intact, indefinitely
- The CYS Temporal Durability signal requires knowing every interaction an artifact has received since creation — not just a current count
- The Chronicle is generated by re-narrating historical events — which is only possible if those events are stored as events, not as aggregated state

**The trade-off:** Current state queries require replaying events or maintaining materialized views (read models). LOKA maintains materialized views in PostgreSQL for frequently-read current state (player position, current artifact ownership, zone occupancy). The Cassandra event store is the source of truth; the PostgreSQL views are caches that can be rebuilt from the event store at any time.

---

## 8. Data Architecture & Key Schemas

> **Plain language:** This section shows the exact structure of the most important data records in LOKA. Non-technical stakeholders can skip the JSON details — the plain-language descriptions capture the key points.

### 8.1 Intent Event — The Atomic Unit of LOKA

Every meaningful action a player takes generates an Intent Event. This is the most important data structure in the system.

```json
{
  "event_id": "uuid-v4",
  "schema_version": "1.0",
  "timeline_id": "TL-2024",
  "shard_date": "2026-04-08",
  "player_id_hash": "sha256-of-player-uuid",
  "session_id": "uuid-v4",
  "timestamp": 1744099200000,
  "sequence": 4721,

  "location": {
    "x": 128430.5,
    "y": 44820.1,
    "z": 234.7,
    "zone_id": "zone-northeast-047",
    "biome": "coastal"
  },

  "intent": {
    "vector": [0.12, -0.34, 0.89, ...],
    "dimensionality": 256,
    "pim_version": "1.4.2",
    "confidence": 0.87,
    "declared_text": "I want to build something that lasts"
  },

  "action": {
    "type": "BUILD",
    "subtype": "STRUCTURE_PLACE",
    "target_id": "artifact-uuid",
    "material_ids": ["stone-block-v2", "iron-beam-v1"],
    "duration_seconds": 847
  },

  "context": {
    "creative_level": 2,
    "intelligence_tier": 3,
    "active_agents": ["oracle", "weaver", "challenger", "smith"],
    "nearby_player_count": 0,
    "active_expedition_id": null,
    "challenger_active": true,
    "challenger_pressure_event_id": "evt-uuid"
  },

  "artifact_ids": ["artifact-uuid"],
  "outcome": "STRUCTURE_COMPLETE"
}
```

**Key design decisions:**
- `player_id_hash` — never the real player UUID; always a SHA-256 hash that can be tombstoned at deletion
- `intent.vector` — embedded at event time, never updated — the record is what the player wanted when they acted
- `intent.confidence` — below 0.5, the intent vector is treated as noisy for CYS purposes
- `shard_date` — Cassandra partition key component; enables efficient time-range queries per timeline

### 8.2 Artifact Schema

```json
{
  "artifact_id": "uuid-v4",
  "schema_version": "1.0",

  "identity": {
    "name": "The Lighthouse of Unseen Shores",
    "type": "STRUCTURE",
    "subtype": "BUILDING",
    "timeline_id": "TL-2024",
    "created_at": 1744099200000,
    "created_by_hash": "sha256-of-player-uuid"
  },

  "creative_signature": {
    "intent_vector_at_creation": [0.12, -0.34, 0.89, ...],
    "declared_intent": "I want to build something that guides people",
    "pim_version_at_creation": "1.4.2",
    "creation_events": ["evt-uuid-1", "evt-uuid-2", "evt-uuid-3"]
  },

  "composition": {
    "material_ids": ["stone-block-v2", "iron-beam-v1", "glass-pane-v3"],
    "construction_duration_seconds": 21600,
    "blueprint_id": "bp-uuid",
    "geometry_hash": "sha256-of-mesh-data"
  },

  "location": {
    "x": 128430.5,
    "y": 44820.1,
    "z": 0.0,
    "zone_id": "zone-northeast-047",
    "timeline_id": "TL-2024"
  },

  "cys": {
    "originality": 0.74,
    "intent_coherence": 0.91,
    "world_impact": 0.43,
    "craft_depth": 0.68,
    "temporal_durability": 0.29,
    "composite_score": 0.0,
    "last_computed": 1744185600000,
    "computation_version": "cys-engine-v2.1"
  },

  "ownership": {
    "current_owner_hash": "sha256-of-player-uuid",
    "co_owners": [],
    "chain_registered": false,
    "token_id": null,
    "contract_address": null,
    "primary_sale_price": null,
    "total_secondary_volume": 0
  },

  "chronicle": {
    "arc_id": "arc-uuid",
    "chronicle_references": ["chapter-uuid-1"],
    "narrative_summary": "A lighthouse built during the storm of the third cycle..."
  },

  "world_impact_log": [
    {
      "event_type": "PLAYER_BUILT_NEAR",
      "player_hash": "sha256-other",
      "timestamp": 1744272000000,
      "proximity_meters": 340
    }
  ]
}
```

### 8.3 Ancestry Graph Schema (Neo4j)

The generational ancestry system is a directed graph where nodes are players and edges represent lineage relationships of varying strength.

```
Node: Player
  properties:
    - id: player_id_hash (string, unique)
    - timeline_id: string
    - first_session: epoch timestamp
    - last_session: epoch timestamp
    - creative_level: integer (0–7)
    - intent_signature: float[256]  (aggregated lifetime intent vector)
    - is_elder: boolean
    - elder_ai_deployed: boolean

Edge: ANCESTOR_OF
  from: ancestor_player_hash
  to: descendant_player_hash
  properties:
    - similarity_score: float (0.0–1.0)
    - dimensions_matched: integer (how many of 256 dims exceeded threshold)
    - confirmed_at: epoch timestamp
    - generation: integer (1 = direct ancestor, 2 = grandancestor, etc.)
    - timeline_id: string

Edge: ENCOUNTERED
  from: player_hash_a
  to: player_hash_b
  properties:
    - encounter_id: uuid
    - encounter_type: CASUAL | EXPEDITION | BOUND_COMPANION
    - timestamp: epoch
    - duration_seconds: integer

Edge: INFLUENCED_BY
  from: influenced_player_hash
  to: influencing_artifact_owner_hash
  properties:
    - artifact_id: uuid
    - influence_type: BUILT_NEAR | TRADED | REFERENCED | CULTURAL_DRIFT
    - timestamp: epoch
```

**Key queries the ancestry system runs:**

```cypher
// Find all descendants of a player within 3 generations
MATCH (ancestor:Player {id: $player_hash})-[:ANCESTOR_OF*1..3]->(descendant:Player)
WHERE ancestor.timeline_id = $timeline_id
RETURN descendant ORDER BY descendant.first_session

// Find top lineage matches for a new player
MATCH (candidate:Player)
WHERE candidate.timeline_id = $timeline_id
  AND candidate.last_session < $cutoff_date
  AND candidate.creative_level >= 2
WITH candidate, 
     gds.similarity.cosine(candidate.intent_signature, $new_player_vector) AS score
WHERE score > 0.72
RETURN candidate ORDER BY score DESC LIMIT 5

// Detect World Impact attribution chain
MATCH (creator:Player)-[:ANCESTOR_OF]->(influenced:Player)
WHERE creator.id = $creator_hash
MATCH (influenced)-[:INFLUENCED_BY {artifact_id: $artifact_id}]->(creator)
RETURN count(influenced) AS impact_depth
```

### 8.4 Player Schema (PostgreSQL — Materialized View)

```sql
CREATE TABLE players (
    player_id         UUID PRIMARY KEY,
    player_id_hash    VARCHAR(64) UNIQUE NOT NULL,  -- SHA-256, used in event store
    timeline_id       VARCHAR(20) NOT NULL,
    
    -- Intelligence
    intelligence_tier  SMALLINT DEFAULT 0,
    pim_version        VARCHAR(20),
    pim_last_trained   TIMESTAMPTZ,
    
    -- Creative
    creative_level     SMALLINT DEFAULT 0,
    cys_lifetime_avg   FLOAT,
    artifact_count     INTEGER DEFAULT 0,
    
    -- Economy
    credits_balance    BIGINT DEFAULT 0,  -- stored in micro-credits to avoid floats
    wallet_address     VARCHAR(42),       -- Ethereum address, optional
    chain_earnings_wei NUMERIC(78,0) DEFAULT 0,
    
    -- Social
    encounter_count    INTEGER DEFAULT 0,
    expedition_count   INTEGER DEFAULT 0,
    bound_companions   UUID[],
    
    -- Ancestry
    ancestor_ids       VARCHAR(64)[],     -- player_id_hash of assigned ancestors
    descendant_count   INTEGER DEFAULT 0,
    
    -- Meta
    first_session      TIMESTAMPTZ NOT NULL,
    last_session       TIMESTAMPTZ,
    total_play_seconds BIGINT DEFAULT 0,
    is_deleted         BOOLEAN DEFAULT FALSE,
    deleted_at         TIMESTAMPTZ
);
```

---

## 9. World Rendering & Spatial Simulation

> **Plain language:** The LOKA world is rendered using Unreal Engine 5 — the same technology used in the most visually impressive games released in recent years. The world's spatial logic (who is near whom, which zones are active, how the world divides itself into manageable pieces) is handled by SpatialOS, a system designed specifically for MMO-scale games where thousands of players share the same world.

### 9.1 Unreal Engine 5 Configuration

**Nanite Virtualized Geometry:** Enables rendering of film-quality geometric complexity without the traditional polygon budget constraints. LOKA's world — with its Memory Strata (visible layers of previous civilizational history) — requires enormous geometric detail in ruined structures and terrain. Nanite makes this economically feasible.

**Lumen Global Illumination:** Real-time, fully dynamic lighting. Critical for LOKA because the world's terrain changes based on player intent — pre-baked lighting would not work for a dynamically responding world. Lumen renders light naturally falling on structures that did not exist when the level was initially created.

**World Partition:** UE5's system for managing enormous open worlds by streaming level chunks dynamically. LOKA's 1:1 Earth-scale world cannot fit in memory — World Partition loads only the player's current area and predictively loads adjacent chunks based on movement vectors.

**Chaos Physics:** UE5's physics engine handles structural simulation for player-built constructions. A building constructed from stone blocks obeys realistic stress physics. If a player builds structurally unsound architecture, it will fail — not as a penalty, but as a consequence of the world's physical rules.

### 9.2 SpatialOS Spatial Partitioning

LOKA's multiplayer fabric uses zone-based spatial partitioning:

```
World divided into zones of ~10km × 10km
Each zone is a SpatialOS "worker" — an independent compute instance

Zone states:
  DORMANT   — no active players nearby; minimal compute (weather sim only)
  AWAKENED  — player approaching (within 20km); zone begins hydrating
  ACTIVE    — player(s) present; full simulation running
  COOLING   — last player departed; 5-minute cooldown before DORMANT

Cost implication: LOKA pays only for ACTIVE zone-time.
At Phase 1 (100 players): ~15–20 zones ACTIVE simultaneously
At Phase 3 (500K players): ~2,000–5,000 zones ACTIVE simultaneously
At Phase 4 (1M players): ~8,000–12,000 zones ACTIVE simultaneously
```

**Private Bubble implementation:**
The Private Bubble is implemented as a SpatialOS interest query constraint. A player in Private Bubble mode has their component interest queries scoped to a 5km radius. Entity updates from beyond that radius are not streamed to their client. The player is invisible to queries from other players' clients until they enter the player's Resonance Zone.

### 9.3 Procedural Terrain Generation

LOKA's terrain is procedurally generated using a layered noise system with **intent-modulated weights**. The pipeline:

```
Base noise layer (Simplex noise, seeded once at launch, never changed)
    + Erosion simulation layer (deterministic, pre-computed)
    + Memory Strata layer (historical player events → terrain features)
    + Intent modulation layer (real-time, per-player, from Oracle directives)
    = Final terrain heightmap for player's zone
```

The intent modulation layer adjusts only the **weights** of terrain generation parameters — it does not overwrite the base map. This means:
- Two players in the same zone experience the same fundamental terrain
- But the camera angles, lighting conditions, available resource node spawns, and NPC placement positions differ based on their intent vectors
- The world is genuinely the same world — experienced differently

---

## 10. The Economy Layer — Technical Design

> **Plain language:** LOKA has two kinds of money. LOKA Credits are like points that exist only inside the game — you earn them by playing well, spend them on game features, and can't directly convert them to real money. On-chain assets are real, permanently owned items that exist on a blockchain — they can be traded outside the game, and their creators earn a percentage of every trade forever. The technical challenge is making both of these work together seamlessly, without exposing players to unnecessary blockchain complexity.

### 10.1 Credits Ledger Architecture

LOKA Credits are managed as an off-chain, custodial ledger. Players do not need a crypto wallet to earn or spend credits.

```
┌─────────────────────────────────────────────────────────────┐
│  CREDITS LEDGER (Sequence/Privy custodial wallet)           │
│                                                             │
│  Player → Credits Account (Sequence smart wallet)          │
│    - Custodial by default (LOKA holds keys)                 │
│    - Player can claim self-custody at any time              │
│    - ERC-20 token on L2 (Base), but gasless for players     │
│                                                             │
│  Credit operations:                                         │
│    EARN: CYS computation → Kafka economy-events → ledger   │
│    SPEND: Intelligence tier, timeline pass, bubble expand   │
│    CONVERT: Credits → stablecoins (time-locked, 30-day)    │
│    GIFT: Player-to-player transfer (no fee)                 │
└─────────────────────────────────────────────────────────────┘
```

**The time-lock on conversion:** Credits can be converted to stablecoins (USDC on Base L2) but with a 30-day lock. This serves two purposes: it prevents speculative farming of credits and then rapid exit, and it gives the system time to run anti-gaming checks before real money leaves.

### 10.2 CYS Computation Pipeline

The Creative Yield Score is computed asynchronously — not in real-time — to allow for correct Temporal Durability measurement.

```
Event triggers for CYS recomputation:
  1. Session end → session credits computed immediately
  2. New World Impact event → affected artifact's CYS updated
  3. 24-hour timer → all artifacts with recent interactions recomputed
  4. 21-day timer → Temporal Durability score unlocked for eligible artifacts

Computation pipeline (ClickHouse):

SELECT
    artifact_id,
    -- Originality: cosine distance from artifact embedding to timeline average
    1 - cosine_similarity(artifact_intent_vector, timeline_avg_vector) AS originality,
    
    -- Intent Coherence: alignment with PIM at creation time
    cosine_similarity(artifact_intent_vector, pim_vector_at_creation) AS intent_coherence,
    
    -- World Impact: downstream action attribution (weighted sum)
    SUM(impact_weight * decay_factor(days_since_impact)) AS world_impact,
    
    -- Craft Depth: decision tree complexity (entropy of build event sequence)
    entropy(build_event_sequence) / LOG(max_possible_decisions) AS craft_depth,
    
    -- Temporal Durability: decay-weighted engagement (0 if < 21 days old)
    CASE WHEN age_days >= 21 
         THEN decay_weighted_interaction_count / baseline_interaction_rate
         ELSE 0.0 END AS temporal_durability

FROM artifact_events
JOIN intent_events ON artifact_events.player_hash = intent_events.player_id_hash
JOIN world_impact_events ON artifact_events.artifact_id = world_impact_events.artifact_id
WHERE artifact_id = $artifact_id
```

### 10.3 Chain Registration Pipeline

Only artifacts that pass the Proof of Intent threshold are eligible for chain registration:

```
Proof of Intent requirements:
  ✓ CYS composite score ≥ 65
  ✓ Intent Coherence ≥ 0.75 (strong alignment between PIM and artifact)
  ✓ Temporal Durability ≥ 0.30 (minimum 21 days old with engagement)
  ✓ No anti-gaming flags in the past 90 days
  ✓ Three Artisan-level peer validators have reviewed and approved

Chain registration flow:
  1. Artifact passes Proof of Intent check
  2. Peer validation pool opens (3 randomly selected Artisan+ players)
  3. Validators blind-review artifact (no creator attribution shown)
  4. 2/3 approval required within 72 hours
  5. On approval: artifact metadata uploaded to IPFS
  6. IPFS CID + artifact_id + creator_hash submitted to smart contract
  7. ERC-721 token minted on Base L2 (creator pays no gas — LOKA subsidizes)
  8. ERC-2981 royalty info set: 7.5% to creator on all secondary sales
  9. chain_registered = true in PostgreSQL, token_id + contract_address stored
```

**Smart contract security:** The royalty contract is audited by a third-party firm (budget: ₹8L in Phase 2) before any real-value artifacts are minted. The ERC-2981 standard has been in production on Ethereum mainnet since 2021 with a strong security track record.

---

## 11. Infrastructure by Phase

> **Plain language:** LOKA's infrastructure grows in three distinct steps matching the user phases. We don't over-engineer for scale we don't have yet — each phase is sized for its actual user count. The key transition is Phase 2, where we shift from paying per-API-call for AI to running our own AI servers.

### 11.1 Phase 1 — MVP Infrastructure (100 Alpha Users)

```
Cloud: AWS (single region: ap-south-1 Mumbai)

Compute:
  Game servers:    2× c5.4xlarge (ECS, auto-scale 1–4)
  API layer:       2× t3.large (load balanced)
  Agent mesh:      2× c5.2xlarge (LangGraph workers)
  
Databases:
  PostgreSQL:      db.t3.medium (RDS, single-AZ)
  Redis:           cache.t3.micro (ElastiCache, session state)
  Cassandra:       3-node cluster, i3.large (self-managed on EC2)
  
Kafka:             MSK, 2 brokers, kafka.t3.small
ClickHouse:        1× c5.2xlarge (self-managed)

AI (all API-based at this scale):
  Claude API:      Anthropic managed
  ElevenLabs:      API
  Stable Diffusion: Replicate API
  Whisper:         OpenAI API

Cost: ~₹1.43L/month
VR: Not deployed at Phase 1 (flat screen only)
```

### 11.2 Phase 2 — Scale Infrastructure (10K Users)

```
Cloud: AWS (primary: ap-south-1, secondary: eu-west-1)

Compute:
  Game servers:    SpatialOS managed (or AWS GameLift FlexMatch)
  API layer:       6× c5.4xlarge (auto-scale, ALB)
  Agent mesh:      8× c5.4xlarge (LangGraph workers, horizontal scale)
  
Databases:
  PostgreSQL:      db.r5.large (RDS Multi-AZ)
  Redis Cluster:   3× cache.r6g.large (session + world state cache)
  Cassandra:       6-node cluster, i3.xlarge (managed via Astra DB or self)
  Neo4j:           AuraDB Professional (managed)
  ClickHouse:      3× c5.4xlarge (self-managed cluster)
  TimescaleDB:     db.r5.large (RDS, TimescaleDB extension)

Kafka:             MSK, 3 brokers, kafka.m5.large

AI (HYBRID — key transition):
  Self-hosted GPU: 4× A100-80GB (RunPod.io or Lambda Labs)
    - Llama-3-8B base (shared)
    - Per-player LoRA adapters served via vLLM
    - Stable Diffusion XL (local)
    - Whisper-large-v3 (local)
  Claude API:      Weaver + complex Oracle only (~30% of calls)
  ElevenLabs:      Bulk plan (NPC voices)

VR deployment:     Meta Quest 3 app build, PCVR (Steam)

Monthly cost: ~₹13.5L
```

### 11.3 Phase 3 — Enterprise Infrastructure (100K–500K Users)

```
Cloud: AWS (3 regions: ap-south-1, eu-west-1, us-east-1)

Compute:
  Game servers:    SpatialOS / custom distributed (multi-region)
  API layer:       Auto-scaling fleet (20–80 instances, c5.4xlarge)
  Agent mesh:      Kubernetes cluster (EKS), 40+ pods

Databases:
  PostgreSQL:      Aurora PostgreSQL (multi-region read replicas)
  Redis:           ElastiCache for Redis Global Datastore
  Cassandra:       18-node cluster (multi-region, strong consistency)
  Neo4j:           AuraDB Business Critical
  ClickHouse:      6-node cluster (self-managed, 2 per region)
  TimescaleDB:     Timescale Cloud (managed)

Kafka:             MSK multi-region replication

AI (PRIMARILY SELF-HOSTED):
  Self-hosted GPU: 20× A100-80GB (long-term lease or owned)
    - vLLM serving Llama-3-8B + all player LoRA adapters
    - Stable Diffusion XL cluster
    - Whisper + local music generation (MusicGen)
  Claude API:      ~5% of calls (Weaver crown jewel narratives only)
  ElevenLabs:      Enterprise agreement (500K users)

CDN:               Cloudflare Enterprise (IPFS artifact gateway, game assets)
Security:          Cloudflare WAF + DDoS, AWS Shield Advanced
Compliance:        Dedicated compliance tooling (DPDP Act, GDPR)

Monthly cost: ~₹59–60L
```

---

## 12. Security Architecture

> **Plain language:** LOKA's security has three main concerns: keeping player accounts safe, preventing cheating and economy manipulation, and protecting the sensitive intent data we collect. We treat these differently because they require different tools — but all three are built in from the start, not added later.

### 12.1 Authentication & Authorization

```
Authentication flow:
  Player → Auth Service (Firebase Auth or Auth0)
         → JWT issued (short-lived, 1 hour)
         → Refresh token (30 days, rotated on use)
         → JWT includes: player_id (UUID), timeline_id, intelligence_tier
  
  All internal services verify JWT signature.
  player_id is never written to the event store — only player_id_hash.
  The hash function: SHA-256(player_id + server_secret)
  
Authorization model:
  Row-level security on PostgreSQL: players can only read their own data
  Agent services run with read-only access to the event store
  Economy service runs with write access to the credits ledger only
  No service has write access to the Cassandra event store except the 
    Kafka Cassandra sink connector (append-only)
```

### 12.2 Anti-Cheat Architecture

LOKA's anti-cheat is AI-native — it uses the same signals as the game to detect fraud.

**Bot detection (World Impact graph):**
Bot farms generate statistically identical, correlated behaviour. Genuine player influence on the World Impact graph produces chaotic, organic downstream patterns. Statistical analysis in Neo4j detects bot clusters via:
- Suspiciously similar intent vectors across multiple accounts
- Correlated timing of actions (bots act on the same trigger simultaneously)
- Absence of emotional signal variance (bots show perfectly flat HRV/facial data)

**CYS gaming detection (Intent Coherence drift):**
Players creating for yield rather than expression show a systematic drift between their PIM vector (which evolves based on authentic signals) and their artifact intent signatures. This drift is measured and triggers a CYS suppression after 5–10 sessions.

**Sybil attack prevention (Peer Validation):**
Multiple accounts controlled by one player attempting to validate each other's artifacts are detectable via:
- Shared IP address / device fingerprint
- Correlated creative output (similar CYS profiles across accounts)
- Network analysis in Neo4j (all accounts in the same social cluster)

**Smart contract security:**
- All royalty distribution logic is on-chain and audited
- Time-lock on Credit → stablecoin conversion prevents rapid extraction
- Multi-sig required for contract upgrades
- Emergency pause function for critical exploits

### 12.3 Data Privacy — DPDP Act Compliance

India's Digital Personal Data Protection Act (2023) governs LOKA's primary market. Key compliance measures:

| Requirement | LOKA's Implementation |
|---|---|
| Consent for data collection | Explicit opt-in at onboarding with granular controls |
| Purpose limitation | Intent data used only for: game operation, PIM training (per player), anonymized research (opt-in) |
| Data minimization | Only intent signals collected — no browsing, location (real-world), contacts |
| Right to erasure | Anonymization pipeline: player_id → tombstone hash; LoRA adapter deleted; training data deleted within 24 hours |
| Data localisation | Phase 1–2: data stored in ap-south-1 (Mumbai). Phase 3+: international replication with user consent |
| Security safeguards | Encryption at rest (AES-256), in transit (TLS 1.3), field-level encryption for PII |

**The anonymization guarantee:**
When a player exercises right to erasure, their real identity is permanently severed from their game history. The Intent Events remain in the event store (ancestry integrity requires this) but become permanently unattributable — the tombstone hash cannot be reversed, and LOKA does not retain any mapping between tombstone hashes and real identities after deletion.

---

## 13. Performance Requirements & SLAs

> **Plain language:** Every part of LOKA has a speed target. VR especially requires that the game feels instant — any delay greater than about 20 milliseconds causes motion sickness. These are the commitments we build toward from day one.

### 13.1 Latency Budgets

| Operation | Target | Maximum Acceptable | Notes |
|---|---|---|---|
| Frame render (VR) | 11ms (90fps) | 14ms (72fps) | Below this causes motion sickness |
| Frame render (flat screen) | 16ms (60fps) | 33ms (30fps) | |
| Player position update (server) | < 50ms | < 100ms | Spatial audio sync requirement |
| NPC reaction to player action | < 500ms | < 1,000ms | Immersion breaks above 1 second |
| Intent vector refresh | < 2,000ms | < 3,000ms | 30-second cycle; latency within cycle |
| Terrain weight update (Oracle) | < 2,000ms | < 5,000ms | Player won't notice < 5s |
| Agent directive application | < 200ms | < 500ms | World Engine applies changes |
| Artifact CYS score (session end) | < 30,000ms | < 60,000ms | Post-session, not real-time |
| Chronicle entry generation | < 5,000ms | < 10,000ms | Background, not blocking |
| Chain registration (end-to-end) | < 10 min | < 30 min | L2 finality time |

### 13.2 Throughput Requirements

| System | Phase 1 (100 users) | Phase 2 (10K users) | Phase 3 (500K users) |
|---|---|---|---|
| Intent Events/second | ~5 | ~500 | ~25,000 |
| Kafka messages/second | ~50 | ~5,000 | ~250,000 |
| Agent invocations/minute | ~200 | ~20,000 | ~1,000,000 |
| Cassandra writes/second | ~10 | ~1,000 | ~50,000 |
| ClickHouse analytics queries/minute | ~10 | ~500 | ~10,000 |

### 13.3 Availability SLAs

| Component | Target Uptime | Acceptable Downtime/Month |
|---|---|---|
| Game servers | 99.5% | 3.6 hours |
| Intent Service | 99.9% | 43 minutes |
| World State Broker (Kafka) | 99.9% | 43 minutes |
| Cassandra (event store) | 99.95% | 22 minutes |
| Economy Layer (credits ledger) | 99.99% | 4 minutes |
| On-chain contracts | 99.99%+ | Ethereum L2 SLA |

The economy layer has the strictest SLA because Credits have real-world value. A player who loses Credits due to a system failure must be made whole — this requires a recovery process backed by an immutable transaction log.

---

## 14. Development Workflow

> **Plain language:** LOKA is built by 8 people in 6 months partly because the team uses AI coding agents as standard infrastructure. Every developer works with Claude Code and Cursor as primary tools. This is not optional — it is how the team achieves the output velocity of a team 3× its size.

### 14.1 AI-Augmented Development Stack

Every developer on the LOKA founding team uses:

| Tool | Purpose | Approx. Productivity Gain |
|---|---|---|
| **Claude Code** | Architecture reasoning, complex implementation, code review | 3–4× on complex tasks |
| **Cursor** | In-editor AI pair programming | 2× on routine implementation |
| **GitHub Copilot** | Autocomplete, boilerplate | 1.5× on repetitive code |

**How the team uses Claude Code specifically:**
- Architecture design sessions: engineer describes requirements, Claude Code designs the component, engineer reviews and refines
- Debugging: full stack traces fed to Claude Code for root cause analysis
- Test generation: Claude Code generates test cases including edge cases the engineer might not have considered
- Documentation: code is explained and documented by Claude Code as it is written

**Team rule:** any component that takes more than 2 hours to design or 4 hours to implement is a signal to pause and work with Claude Code before proceeding.

### 14.2 CI/CD Pipeline

```
Development → Pull Request → CI → Staging → Production

CI pipeline (GitHub Actions):
  1. Unit tests (pytest / Jest)
  2. Integration tests (Docker Compose environment, real Kafka/Cassandra)
  3. Intent loop smoke test (5-minute test session with synthetic player)
  4. Agent mesh contract tests (each agent validated against schema)
  5. Security scan (Snyk for dependencies, Semgrep for code patterns)
  6. Performance benchmark (regression check on latency targets from §13)

Deployment:
  Staging: automatic on PR merge to main
  Production: manual approval gate after staging validation
  Rollback: automated rollback if error rate > 1% in first 10 minutes
```

### 14.3 Branching Strategy

```
main          → always deployable, protected
develop       → integration branch
feature/*     → individual feature work (max 3 days before merge)
hotfix/*      → production fixes (fast-track CI, direct to main)
release/*     → stabilization branch before major phase launches
```

The 3-day feature branch limit is enforced to prevent large merges. The intent loop is complex — long-lived branches diverge badly from the rapidly-evolving agent mesh.

---

## 15. Testing Strategy

> **Plain language:** Testing a game that responds to human intent is fundamentally different from testing conventional software. The system has no single "correct" output — the world should respond appropriately to each player's intent, but appropriate is subjective. The testing strategy handles this by separating objective tests (did the system function correctly?) from subjective tests (did it feel right?).**

### 15.1 Test Pyramid

```
                    /\
                   /  \
                  / E2E \      < 5% of tests
                 /────────\    Real sessions, alpha player feedback
                / Integration\  ~20% of tests
               /──────────────\ Agent mesh, event store, latency
              /   Unit Tests    \ ~75% of tests
             /────────────────────\ Schema validation, CYS computation, PIM

```

### 15.2 The Intent Loop Smoke Test

The most critical automated test in LOKA. Runs in CI on every merge to main.

```python
# Pseudocode — actual implementation in Python with LangGraph test harness

def test_intent_loop_response():
    """
    Verifies that the world produces meaningfully different responses
    to meaningfully different intents. Not testing 'correct' output —
    testing that the loop is functioning and differentiated.
    """
    # Synthetic player 1: builder intent
    builder = SyntheticPlayer(
        declared_intent="I want to build something that endures",
        movement_profile="methodical, slow, resource-gathering",
        timeline="TL-2024"
    )
    
    # Synthetic player 2: explorer intent
    explorer = SyntheticPlayer(
        declared_intent="I want to find what no one has seen",
        movement_profile="fast, wide-ranging, ridge-seeking",
        timeline="TL-2024"
    )
    
    # Run 5-minute simulated sessions
    builder_result = run_session(builder, duration_seconds=300)
    explorer_result = run_session(explorer, duration_seconds=300)
    
    # Assert differentiated world responses
    assert builder_result.terrain_flatness_score > 0.7  # builder got buildable land
    assert explorer_result.unexplored_ridgelines_count > 3  # explorer got ridgelines
    
    # Assert responses are different
    terrain_similarity = cosine_similarity(
        builder_result.oracle_directives, 
        explorer_result.oracle_directives
    )
    assert terrain_similarity < 0.5  # responses should be meaningfully different
    
    # Assert chronicle was generated
    assert len(builder_result.chronicle_entries) >= 1
    assert len(explorer_result.chronicle_entries) >= 1
    
    # Assert latency targets
    assert builder_result.avg_intent_refresh_latency < 2000  # ms
    assert explorer_result.avg_intent_refresh_latency < 2000  # ms
```

### 15.3 Agent Contract Tests

Each agent is tested against its expected schema contract:

```
Oracle contract:
  Input:  IntentVector(256), ActionHistory(last 20), TerrainState
  Output: TerrainDirective(weights[]), NPCParameters[], EventQueueItems[]
  
  Contract test: given valid input, output must:
  - Contain at least one terrain weight adjustment
  - All weights in range [0.0, 1.0]
  - Response latency < 2000ms (P95)

Weaver contract:
  Input:  ChronicleArc, RecentEvents(5min window), CulturalMemory
  Output: ChronicleEntry(text, arc_id, timestamp), optional InWorldDocument
  
  Contract test: given valid input, output must:
  - Contain a non-empty chronicle entry
  - Chronicle entry references at least one event_id from input
  - Literary style matches timeline (tested via classifier)
  - Response latency < 5000ms (P95)
```

### 15.4 Alpha Player Testing Protocol

The MVP is validated by 100 alpha players in a structured 6-week test:

**Week 1–2:** Unguided play. Players given no instructions beyond onboarding. We observe unprompted behaviour.

**Week 3:** Structured feedback session. Players asked: *"Did the world feel like it was listening to you?"* Target: ≥60% yes responses to proceed.

**Week 4:** Intent coherence test. Players who said "yes" are asked to describe what they wanted. We compare their description against their PIM vector. Target: >70% semantic alignment.

**Week 5–6:** Repeat week 1–2 with players who have 2 weeks of history. Measure return rate. Target: ≥50% play again without prompting in week 6.

**Go/no-go criterion for Phase 2 funding:** ≥60% felt heard, ≥50% returned unprompted.

---

## 16. Observability & Operations

> **Plain language:** Once LOKA is live, the team needs to know immediately if something is wrong — before players notice. This section describes the monitoring systems and the process for responding to incidents.

### 16.1 Key Dashboards (Grafana)

**Dashboard 1 — The Intent Loop Health**
Monitors the core mechanic. Shows:
- Intent vector refresh latency (P50, P95, P99)
- Oracle directive application rate (should be 100% of refreshes)
- PIM confidence distribution (how certain the model is about current intent)
- Alert: if P95 latency > 3000ms, page on-call

**Dashboard 2 — Agent Mesh Health**
Shows per-agent:
- Invocation success rate (should be > 99%)
- Response latency distribution
- Error rates by error type
- Alert: if any agent error rate > 5%, page on-call

**Dashboard 3 — World State Broker Health**
Shows:
- Kafka consumer lag per topic (should be < 1000 messages)
- Cassandra write latency (P95 < 50ms)
- Cassandra read latency (P95 < 20ms)
- Event store size growth rate
- Alert: consumer lag > 10,000 → immediate investigation

**Dashboard 4 — Economy Health**
Shows:
- Credits issued/consumed ratio (should be balanced)
- CYS computation queue depth
- Chain registration pipeline status
- Unusual withdrawal patterns (anti-fraud)
- Alert: any single player withdrawing > 10× their historical rate

**Dashboard 5 — Player Experience**
Shows:
- Session duration distribution
- Return rate (% of players who played yesterday who play today)
- Onboarding completion rate
- Support ticket volume and category distribution

### 16.2 Incident Response

```
Severity levels:
  P0: Game completely down for all players
      Response: immediate all-hands, 15-minute SLA to response
      
  P1: Core loop degraded (intent not being read, agents not responding)
      Response: on-call engineer, 30-minute SLA
      
  P2: Economy anomaly, data pipeline lag > 30 minutes
      Response: on-call engineer, 2-hour SLA
      
  P3: Single agent down, non-critical feature degraded
      Response: next business day

Runbook locations: /ops/runbooks/ in the engineering repo
On-call rotation: established at Phase 2 (13-person team)
```

### 16.3 Disaster Recovery

| Scenario | Data Loss Risk | Recovery Time | Recovery Method |
|---|---|---|---|
| Single game server crash | Zero | < 1 minute | SpatialOS auto-restart; zone state from Cassandra |
| Kafka broker failure | Zero | < 5 minutes | MSK managed recovery; consumers resume from offset |
| Cassandra node failure | Zero | < 10 minutes | Replication factor 3; quorum maintained |
| Full region outage | Zero | < 30 minutes (P2+) | Multi-region failover (Phase 2+) |
| Accidental event deletion | Not possible | N/A | Event store is append-only; deletion requires cluster rebuild |
| L2 contract exploit | Potential | Pause contract immediately; assess damage | Emergency pause function in contract |

---

## 17. Phase-by-Phase Technical Roadmap

### Phase 1 — The Kernel (Months 1–6)

**What gets built:**
- UE5 client (flat screen, Windows/Mac)
- Intent Service v1 (PIM initialization, basic intent extraction)
- Oracle + Weaver agents only
- Private Bubble world (no multiplayer)
- Single timeline (TL-2024)
- Basic artifact creation (structures, written texts)
- LOKA Credits ledger (off-chain, no withdrawal)
- World State Broker (Kafka + Cassandra, single region)
- Intent loop smoke test suite

**What explicitly does NOT get built (scope control):**
- VR deployment
- Multiplayer / Encounter system
- Challenger or Smith agents
- Chain registration or blockchain
- Generational Ancestry system
- Pantheon / Leaderboard
- Voice input (text-only intent declaration at Phase 1)

**Key technical milestone:** Alpha player feels the world responding to their stated intent within 3 sessions (measured by structured feedback week 3–4).

### Phase 2 — Encounter Layer (Months 7–12)

**What gets built:**
- Multiplayer (SpatialOS or GameLift, Shared Territory + Encounter Zone)
- VR client (Meta Quest 3 + PCVR)
- Challenger + Smith + Hermès agents
- Expedition system (shared Private Bubble, co-owned artifacts)
- Voice input (Whisper STT + speech-to-intent)
- Artifact economy (Layers 1 + 2, LOKA Credits earnings)
- Chain registration pipeline (Proof of Intent, peer validation)
- Basic Pantheon (3 categories: Wanderers, Architects, Connectors)
- GPU self-hosting transition (4× A100, vLLM)
- Multi-region deployment (ap-south-1 + eu-west-1)

**Key technical milestone:** Per-user AI cost < ₹60/month (proving self-hosting economics).

### Phase 3 — The Manifold (Months 13–24)

**What gets built:**
- 3 additional timelines (TL-1850, TL-1970, TL-2045)
- Generational Ancestry System (full Neo4j graph, Lineage Match, ancestor NPCs)
- Elder AI (Tier 5) — per-player model distillation + NPC deployment
- Full Pantheon (all 8 categories including The Unnamed)
- Creative Level architecture (Levels 0–7, Progenitor Dividend)
- Cross-timeline travel (timeline passes)
- Research data licensing pipeline (anonymized export API)
- GPU cluster expansion (20× A100, multi-region)
- Chronicle publication pipeline (annual output, human-edited)

**Key technical milestone:** First player achieves Tier 5 (Elder AI distilled and deployed).

### Phase 4 — The Living World (Year 3+)

**What gets built:**
- Unlimited timeline engine (dynamic timeline creation from player demand)
- Cross-timeline ancestry matching
- Owned GPU infrastructure (replace rental with owned servers)
- Research API (academic institution access with data governance layer)
- Enterprise data licensing (AI training data sales)
- The Unnamed leaderboard activation (criteria deliberately never published)

---

## 18. Technical Risk Register

| Risk | Likelihood | Impact | Mitigation | Owner |
|---|---|---|---|---|
| Intent loop feels like random generation to players | Medium | Critical | 6-week alpha test with explicit measurement before Phase 2 invest | Tech Lead |
| PIM cold start produces poor first-session experience | High | Medium | Onboarding void design compensates; 3-session convergence acceptable | AI/ML Engineer |
| Claude API latency degrades in production | Medium | High | Self-hosting migration at Phase 2; Claude used for <30% of calls | DevOps |
| Cassandra event store size grows beyond budget | Low | Medium | Tiered storage (hot/warm/cold) implemented at Phase 2; cost modeled | DevOps |
| SpatialOS pricing change after adoption | Medium | High | Architecture designed to be portable to AWS GameLift with 4-week migration | Tech Lead |
| LoRA adapter storage at 1M+ players (16MB × 1M = 16TB) | Low | Medium | Adapter compression (4-bit quantization reduces to ~4MB); tiered storage | AI/ML Engineer |
| L2 smart contract exploit | Low | Critical | Third-party audit; emergency pause function; cold storage for treasury | Security Engineer |
| DPDP Act interpretation more restrictive than expected | Medium | High | Privacy-by-architecture design means technical compliance is not dependent on legal interpretation | Legal + DevOps |
| LangGraph framework deprecated or major breaking change | Low | Medium | Agents are thin wrappers; core logic is portable to any orchestration framework in ~4 weeks | Agent Mesh Engineer |
| VR motion sickness at high PIM refresh rates | Medium | Medium | Refresh rate decoupled from render rate; PIM runs at 30s cadence not per-frame | UE5 Developer |

---

## 19. Glossary

| Term | Definition |
|---|---|
| **Agent** | A specialized AI component with a bounded context window and a single responsibility. LOKA has six: Oracle, Weaver, Challenger, Smith, Hermès, Ancestor. |
| **Ancestor (Agent)** | The agent responsible for generational pattern matching — comparing new players' intent vectors to historical records and assigning ancestry. |
| **Ancestor NPC** | An AI-controlled character in a player's world whose personality and behaviour are derived from a real player's Elder AI. |
| **Artifact** | Any durable creation in LOKA: structure, music, text, map, tool, covenant, recipe. Every artifact has a creative signature (intent vector at creation), timeline provenance, and chronicle reference. |
| **Bound Companion** | The deepest social bond in LOKA. Two players write a Covenant — a declared shared intent — that the world holds as a living contract. |
| **Cassandra** | The distributed database used as LOKA's immutable event store. Chosen for its append-only write performance and timeline-partitioned data model. |
| **Challenger (Agent)** | The adversarial agent that finds the gap between a player's stated desire and actual behaviour, then introduces scenarios that expose that gap. Separate from all assist agents. |
| **Chronicle** | The ongoing narrative record of a player's journey in LOKA, generated by the Weaver agent. Published annually as *The Chronicle of the Common World*. |
| **ClickHouse** | A columnar analytics database used for CYS computation, World Impact scoring, and Pantheon rankings. |
| **Convergence Point** | Specific in-world locations where different timelines become accessible — where multiple timeline versions of the world thin and crossings occur. |
| **Covenant** | A written intent declaration between two Bound Companions. The world treats it as a living contract and generates scenarios to test and reward it. |
| **Creative Yield Score (CYS)** | A composite score (0–100) measuring the genuine creative value of an artifact. Computed from five signals: Originality, Intent Coherence, World Impact, Craft Depth, and Temporal Durability. |
| **Cross-Timeline Signature** | The composite intent vector created when a player holds multiple timeline passes and plays across timelines. Enables cross-timeline ancestry matching. |
| **Elder AI** | Tier 5 intelligence. A per-player AI model distilled from the player's complete LOKA history, deployed as an ancestor NPC in other players' worlds. Cannot be purchased — earned only. |
| **Encounter Zone** | The spatial region beyond 50km from a player's position where other players become visible and interaction becomes possible. |
| **Event Sourcing** | The architectural pattern of storing system state as a sequence of immutable events rather than as mutable records. LOKA's entire history is event-sourced. |
| **Expedition** | A formal game state where two or more players merge their Private Bubbles into a Shared Expedition, generating co-owned artifacts and a shared chronicle chapter. |
| **Feast** | The ML feature store used to provide consistent features for PIM training across the team. Ensures that the features used during training match those available at inference time. |
| **Generational Ancestry** | The system by which players with similar intent patterns across time are assigned ancestor/descendant relationships, regardless of whether they ever met. |
| **Hermès (Agent)** | The social agent responsible for managing encounters between players — detecting intent compatibility, generating encounter events, and presenting interaction options. |
| **Intent Coherence** | The cosine similarity between a player's PIM vector at creation time and the intent signature embedded in an artifact. The master anti-gaming signal. |
| **Intent Event** | The atomic data record in LOKA. Every meaningful player action generates an Intent Event stored in the World State Broker. |
| **Intent Vector** | A 256-dimensional float array representing what a specific player actually wants at a given moment. Produced by the Player Intent Model every 30 seconds. |
| **Kafka** | The distributed message streaming system used as the ingestion layer of the World State Broker. |
| **Ley Lines** | Invisible energy corridors in the Manifold Map implemented as vector similarity fields where intent embeddings influence procedural generation weights. |
| **Lineage Match** | The process run by the Ancestor agent when a new player joins: comparing their early intent vector against the historical lineage index to identify potential ancestors. |
| **LLM (Large Language Model)** | A type of AI model trained on large amounts of text that can generate, summarize, and reason about language. Powers all six LOKA agents. |
| **LoRA (Low-Rank Adaptation)** | A technique for fine-tuning large AI models with minimal compute by adding small adapter weight matrices to a frozen base model. Used for the per-player PIM. |
| **Memory Strata** | The visible geological layers in LOKA's terrain that represent real player history — ruins are genuine ruins, abandoned structures were genuinely built. |
| **Mythological Echoes** | Narrative features, artifacts, and landscape elements generated when a new player's cross-timeline signature matches a past player's signature, referencing the original without direct attribution. |
| **NPC (Non-Player Character)** | An AI-controlled character in the game world. In LOKA, NPCs are not scripted — they are instantiated with personality tensors drawn from world cultural history. |
| **Neo4j** | A graph database used for the Generational Ancestry system. Chosen because ancestry is inherently a graph traversal problem. |
| **OpenXR** | The hardware-agnostic VR runtime standard. LOKA's VR client targets OpenXR to support Meta Quest, Valve SteamVR, and PlayStation VR2 without separate codebases. |
| **Oracle (Agent)** | The intent-reading agent. Receives the current intent vector and recent action history, produces terrain weight adjustments and NPC parameters. |
| **Pantheon** | LOKA's multi-dimensional recognition system. Eight leaderboard categories measuring different forms of player value. The eighth — The Unnamed — has no published criteria. |
| **PIM (Player Intent Model)** | A per-player LoRA adapter applied to the Llama-3-8B base model. Trained on the player's behavioural signals, produces a 256-dimensional intent vector. |
| **Private Bubble** | The 5km radius around each player that is entirely their own. No one enters without an explicit invitation. |
| **Proof of Intent** | The eligibility check for chain registration: CYS ≥ 65, Intent Coherence ≥ 0.75, Temporal Durability ≥ 0.30, no anti-gaming flags, peer validation passed. |
| **Resonance Zone** | The 5–50km radius beyond the Private Bubble where other players' environmental impacts (structures, paths, resource depletion) are visible but players are not yet visible. |
| **Smith (Agent)** | The crafting agent. Assists with artifact and structure creation by suggesting material combinations, proportions, and creative directions aligned with the player's intent. |
| **SpatialOS** | Improbable's distributed spatial simulation platform. Manages zone-based world partitioning for MMO-scale multiplayer. |
| **Timeline** | A version-stamped instance of the Manifold that mirrors the cultural and technological context of a specific real-world era. Each has its own physics, cultural memory, and economy. |
| **TimescaleDB** | A time-series extension to PostgreSQL used for temporal queries across timeline eras — e.g., what did the world look like in a specific zone 3 months ago? |
| **Weaver (Agent)** | The narrative agent. Generates the player's chronicle, writes in-world documents, and creates the annual Chronicle of the Common World. |
| **World Impact** | One of the five CYS signals. Measures whether other players' downstream behaviour changed because of a specific artifact. Verified via graph analysis in Neo4j. |
| **World State Broker** | The central event store that is the single source of truth for all LOKA state. Built on Apache Kafka + Cassandra. All agents and components read and write through it. |
| **vLLM** | An open-source LLM serving framework that enables efficient serving of multiple LoRA adapters simultaneously on shared GPU hardware. Used in Phase 2+ self-hosted inference. |

---

*This document is maintained by the LOKA engineering team. For questions, corrections, or contributions, refer to the game-vault repository.*

*Document version: 1.0 | Last updated: April 2026*

*Related documents: [[business-proposal]] | [[game-concept]] | [[intent-engine]] | [[multi-agent-system]] | [[economy-and-earnings]] | [[social-and-ancestry]] | [[technical-architecture]] | [[budget-roadmap]]*
