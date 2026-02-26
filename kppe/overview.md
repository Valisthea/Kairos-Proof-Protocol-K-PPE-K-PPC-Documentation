# What is K-PPE?

K-PPE (Kairos Proof Protocol Engine) is the core trustless anchoring engine of the Kairos Proof Protocol. It is a TypeScript/Node.js application that:

- **Builds Merkle trees locally** from client-hashed events (`src/kpe/merkle.ts`)
- **Anchors only the Merkle root** on-chain via `KPPEAnchor.anchorBatch()` — event data **never** goes on-chain
- **Stores proofs** in Supabase (merkle_batches, event_receipts) for off-chain verification
- **Maintains chain of trust** via `prevRoot` linking batches chronologically
- Provides hybrid authentication (SIWE + post-quantum) via KA-HAP 7-layer security

**Repository**: `kairos-proof-protocol-engine` (public, all rights reserved)

## Live Deployment

| Item | Value |
|------|-------|
| **Chain** | Base Mainnet (Chain 8453) |
| **KPPEAnchor** | `0x3B53F7044E47766769156bF210c2661F03Df45dd` |
| **App Registry** | `0xcbf3e71fb2E09929682b8448442d184f8E1E37B8` |
| **Protocol** | K-PPE-v1 |
| **Merkle spec** | Sorted-pair Keccak-256, padded to power-of-2, max 256 leaves |
| **On-chain cost** | 32 bytes (root) + uint256 (eventCount) per batch |
| **KPPE_MODE** | `kppe` (production), `legacy` (deprecated), `disabled` |

## KPPE_MODE

The relayer supports three modes controlled by the `KPPE_MODE` environment variable:

| Mode | Behavior | Data on-chain | Privacy |
|------|----------|---------------|---------|
| `kppe` | Local Merkle tree → `anchorMerkleRoot()` → Supabase | Root hash only (32 bytes) | **Private** |
| `legacy` | Old `sendBatch()` → full event data on-chain | ALL event data | **Public** — DEPRECATED |
| `disabled` | Database only, no on-chain anchoring | Nothing | N/A |

## Who Uses K-PPE

K-PPE is infrastructure. It can be consumed by:
- **Kairos Analytics** (first-party consumer — deployed)
- **Any dApp** via REST API or SDK
- **Any protocol** that needs verifiable event proofs
- **Any developer** building on top of proof infrastructure

## Current Status

| Component | Status |
|-----------|--------|
| Project scaffold | ✅ Deployed |
| Local Merkle tree builder (`src/kpe/merkle.ts`) | ✅ Deployed |
| KPPEAnchor smart contract (Base Mainnet) | ✅ Deployed & Verified |
| `anchorMerkleRoot()` in OnChainSender | ✅ Deployed |
| `flushKPPE()` in BatchManager | ✅ Deployed |
| KPPE_MODE routing (kppe / legacy / disabled) | ✅ Deployed |
| Chain of trust (prevRoot linking) | ✅ Deployed |
| Supabase persistence (merkle_batches, event_receipts) | ✅ Deployed |
| KA-HAP 7-layer security | ✅ Deployed |
| Classical crypto (Ed25519, AES, HMAC) | ✅ Deployed |
| PQ crypto (simulated ML-DSA, ML-KEM, SPHINCS+) | 🔨 Phase 1 |
| SIWE authentication | 🔨 Phase 1 |
| Production PQ (liboqs) | 📋 Phase 3 |
| K-PPC on-chain anchoring | 📋 Planned — Asterchain L1 Mainnet |
| ZK proof circuits | 📋 Phase 5 |
