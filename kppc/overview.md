# What is K-PPC?

K-PPC (Kairos Proof Protocol Chain) is a dedicated blockchain designed for one purpose: **storing and verifying cryptographic proofs permanently.**

K-PPC is not a general-purpose blockchain. It doesn't host DeFi protocols, NFTs, or smart contracts. It is a purpose-built proof ledger — an immutable, decentralized record of on-chain events verified by the Kairos Proof Protocol.

## Status: PLANNED — Asterchain L1 Mainnet

> **K-PPC will be deployed on Asterchain L1 Mainnet.** Architecture and integration details are being defined. K-PPE (the engine) is already deployed on Base Mainnet and will bridge proofs to K-PPC when ready.

K-PPC is planned for Phase 4 of the roadmap (2027+). The current focus is on K-PPE (the engine), which is live on Base Mainnet.

---

# Why a Dedicated Chain?

## The Deletion Problem

K-PPE stores proofs in Supabase and anchors Merkle roots on Base Mainnet via KPPEAnchor. Both are centralized services. If they go down — or if data is deleted — the proofs are lost.

The Merkle root is anchored on Base Mainnet via `KPPEAnchor.anchorBatch()`, but without the underlying proof data, the root alone cannot verify individual events. It's like having the seal of a court document without the document itself.

## Why Not Just Use Arweave / IPFS?

| Solution | Permanent? | Verifiable? | Structured? | Decentralized? |
|----------|-----------|------------|------------|---------------|
| Arweave | ✅ | ❌ (raw data) | ❌ | ✅ |
| IPFS/Filecoin | ⚠️ (pinning) | ❌ | ❌ | ✅ |
| Base Mainnet | ✅ | ✅ | ❌ (expensive) | ✅ |
| **K-PPC** | **✅** | **✅ (native)** | **✅ (purpose-built)** | **✅** |

K-PPC is designed from the ground up for proof storage and verification. It understands proof structures natively. Verification is a first-class operation, not an afterthought. Query proofs by event hash, by batch, by time range, by source chain — all at the protocol level.

## The Trust Equation

```
WITH K-PPE only:
  Trust = Railway uptime × Supabase integrity
  Risk = Single provider failure → data loss

WITH K-PPE + K-PPC:
  Trust = At least 1 validator out of N is honest
  Risk = Must compromise >2/3 of validators simultaneously
```

---

# K-PPC Architecture

## Chain Design: ZK Rollup on Asterchain L1 Mainnet

> **Target L1: Asterchain L1 Mainnet** — Architecture details being finalized.
> K-PPE is currently live on **Base Mainnet** via KPPEAnchor contract.

```
┌─────────────────────────────────────────────┐
│              K-PPC (ZK Rollup)               │
│                                             │
│  Sequencer (Kairos-operated initially)      │
│       │                                     │
│       ▼                                     │
│  Proof Batches → ZK Circuit → ZK Proof      │
│       │                                     │
│       ▼                                     │
│  State Root + ZK Proof published to L1      │
│                                             │
│  Native operations:                         │
│  • storeProof(batchId, merkleRoot, sigs)    │
│  • verifyProof(batchId, eventHash)          │
│  • queryProofs(filter)                      │
│  • anchorExternal(sourceChain, rootHash)    │
│  • bridgeFromKPPE(batchId, merkleRoot)      │
└──────────────────┬──────────────────────────┘
                   │ state roots + ZK proofs
                   ▼
┌─────────────────────────────────────────────┐
│         Asterchain L1 Mainnet                │
│                                             │
│  Verification contract:                     │
│  • Verifies K-PPC ZK proofs                 │
│  • Stores state roots                       │
│  • Enables cross-chain verification         │
│  • Settlement layer                         │
│  • Bridge from Base Mainnet K-PPE batches   │
└─────────────────────────────────────────────┘
```

## Why ZK Rollup (Not Optimistic)

- **Optimistic Rollup**: 7-day dispute window. Proofs would need a week to finalize.
- **ZK Rollup**: Mathematical proof of correctness. Proofs finalize in minutes.

For a proof system, mathematical certainty is non-negotiable. Optimistic dispute windows introduce a period where proofs are "probably correct" — unacceptable for audit-grade data.

## Estimated Specifications

| Parameter | Target |
|-----------|--------|
| Block time | 2 seconds |
| Proof finality on L1 | ~10 minutes (batch ZK proof) |
| Throughput | 1,000+ proofs/second |
| Storage cost | ~$0.001 per proof |
| Gas token | Asterchain native token (TBD) |
| No native token | Simplifies regulatory compliance |

---

# ZK Proofs Integration

## What ZK Adds

Zero-Knowledge proofs enable Kairos Analytics to **prove statements about data without revealing the data itself**.

| Without ZK | With ZK |
|-----------|---------|
| "Here are all 50,000 trades, verify them" | "The total volume was $10M. Here's a ZK proof. Verify without seeing individual trades." |
| "Here's the wallet's full trading history" | "This wallet was profitable. Here's a ZK proof. No strategy revealed." |
| "Here are all oracle prices for the day" | "No price deviated >0.5% from the median. Here's a ZK proof." |

## ZK Circuit Types (Planned)

### 1. Proof of Event Validity
"This on-chain event happened exactly as described"
- Input: raw event data (private)
- Output: event hash matches Merkle root (public)

### 2. Proof of Aggregate
"The sum/average/count of these events equals X"
- Input: individual events (private)
- Output: aggregate value + correctness proof (public)

### 3. Proof of Solvency
"This entity's assets exceed its liabilities"
- Input: wallet balances, positions (private)
- Output: boolean (solvent or not) + proof (public)

### 4. Proof of Compliance
"All transactions met regulatory criteria X"
- Input: transaction details (private)
- Output: compliance status + proof (public)

## ZK Tech Stack (Planned)

| Component | Technology | Why |
|-----------|-----------|-----|
| Circuit language | **Rust** (via SP1 or Risc0) | Write ZK proofs in Rust, not Circom |
| Proving system | **STARK → SNARK** (Plonky2 or SP1) | STARK for recursion, SNARK for on-chain verification |
| Verification | BNB Chain smart contract | On-chain verification of K-PPC proofs |

SP1 and Risc0 allow writing ZK circuits in standard Rust — dramatically more accessible than low-level Circom/Halo2. This is why we chose to wait (Phase 5) rather than build ZK circuits now with less mature tooling.

---

# AsterDex Integration

## Native DeFi Proof Generation

AsterDex (perpetual DEX) is a primary data source for Kairos Analytics. The integration enables:

1. **Real-time event capture**: Every trade, liquidation, funding rate update, and oracle price on AsterDex is captured by K-PPE indexers via WebSocket
2. **Proof generation**: Events are batched into Merkle trees every 60 seconds
3. **Hybrid signing**: Roots are signed with Ed25519 + SPHINCS+
4. **Public verification**: Anyone can verify any AsterDex event through K-PPE's public endpoint

## AsterDex Data Points

| Event Type | Data Captured | Proof Value |
|-----------|--------------|-------------|
| Trade execution | Pair, price, size, timestamp, trader | Volume verification |
| Liquidation | Position, liquidation price, margin | Risk monitoring |
| Funding rate | Rate, timestamp, pair | Cost verification |
| Oracle update | Price, source, block, deviation | Feed accuracy |
| Open Interest | Per-pair OI snapshots | Market structure |

## Future: AsterDex on K-PPC

When K-PPC launches, AsterDex events can be anchored directly on the proof chain. This creates a permanent, decentralized record of every event on AsterDex — verifiable by anyone, forever, without trusting AsterDex or Kairos.

---

# Consensus & Validators

## Phase 1: Centralized Sequencer

Initially, K-PPC runs with a Kairos-operated sequencer. This is standard for new rollups (Optimism, Base, zkSync all launched with centralized sequencers).

Proofs are still ZK-verified on BNB Chain, so the sequencer cannot falsify data — it can only censor or delay, which is detectable.

## Phase 2: Shared Sequencer

Integrate with shared sequencer networks (Espresso, Astria) to decentralize ordering without bootstrapping a dedicated validator set.

## Phase 3: Decentralized Validation

Full validator set with:
- Geographic distribution (minimum 3 continents)
- Stake-weighted consensus
- Slashing for data withholding
- Rotating leader selection

## Target Validator Economics

| Parameter | Target |
|-----------|--------|
| Minimum validators | 10 |
| Target validators | 50+ |
| Stake requirement | TBD (BNB-denominated) |
| Revenue source | Proof storage fees |
| No token | Gas in BNB, no governance token |
