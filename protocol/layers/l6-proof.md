# L6 — Proof Engine Layer

> **Status: DEPLOYED** — KPPEAnchor on Base Mainnet, local Merkle builder in production.

## Purpose
Generate cryptographic proofs of on-chain events using Merkle trees and hybrid signatures. Only the Merkle root is anchored on-chain — event data stays private. Proofs are publicly verifiable on-chain and designed to remain valid for 20+ years.

## Proof Generation Flow (Production — KPPE_MODE=kppe)

```
SDK (Browser/dApp)
    │
    ├─── Hash event client-side ── keccak256(event) = client_hash
    │    (hash born in browser, engine NEVER re-hashes)
    │
    ▼
Relayer (Railway)
    │
    ├─── Store event + client_hash ── Supabase (events table)
    │
    ▼
BatchManager cron (every 60s)
    │
    ├─── Collect unbatched events per app
    │
    ├─── Build Merkle Tree locally ── src/kpe/merkle.ts
    │    Sorted-pair Keccak-256, padded to power-of-2
    │    Max 256 leaves per batch
    │
    ├─── Verify all proofs locally (self-test)
    │
    ├─── Anchor on-chain ──────────── KPPEAnchor.anchorBatch()
    │    Only 32 bytes (root) + eventCount
    │    prevRoot links to previous batch (chain of trust)
    │
    ├─── Save proofs ──────────────── Supabase (merkle_batches + event_receipts)
    │
    ├─── [FUTURE] Sign root ──────── Ed25519 + SPHINCS+-256s (PQ)
    │
    ▼
    [FUTURE: Bridge to K-PPC on Asterchain L1]
```

## Event Hashing

Events are hashed **client-side** by the SDK before being sent to the relayer. The engine never re-hashes — it uses the `client_hash` as-is for Merkle leaf construction:

```typescript
// Client-side (SDK) — hash born in the browser
function hashEvent(event: OnChainEvent): string {
  const canonical = JSON.stringify(event, Object.keys(event).sort());
  return keccak256(canonical);
}
// → This hash becomes client_hash, used as Merkle leaf
```

For events without SDK client hashing (legacy), the relayer generates a fallback hash server-side.

Using Keccak-256 ensures compatibility with the EVM ecosystem. Any Ethereum client can independently verify the hash.

## Merkle Tree Construction — Sorted-Pair Hashing

The K-PPE Merkle tree uses **sorted-pair hashing** for deterministic, order-independent construction:

```typescript
// Sorted-pair hash function (EVM-compatible)
function sortedPairHash(a: string, b: string): string {
  const [left, right] = BigInt(a) < BigInt(b) ? [a, b] : [b, a];
  return keccak256(abi.encodePacked(left, right));
}
```

```
                    Root (anchored on-chain)
                   /             \
          sortedPairHash    sortedPairHash
             /      \           /      \
          H(0)     H(1)     H(2)     H(3)    ← client_hash values
           │        │        │        │
        Event_0  Event_1  Event_2  Event_3
```

**Key properties:**
- **Sorted-pair**: `H(a,b) = keccak256(min(a,b) ‖ max(a,b))` — same result regardless of sibling order
- **Padded to power-of-2**: Empty slots filled with zero hashes (`0x0000...`)
- **Max 256 leaves** per batch (configurable via `KPPE_MAX_BATCH_SIZE`)
- **Deterministic**: Same events always produce the same root
- **Batches** generated every 60 seconds (configurable)

## Chain of Trust (prevRoot)

Each batch on-chain stores a `prevRoot` reference to the previous batch's Merkle root:

```
Batch #1          Batch #2          Batch #3
┌───────────┐     ┌───────────┐     ┌───────────┐
│ root: 0xAB│◄────│ prevRoot: │◄────│ prevRoot: │
│ prevRoot:0│     │   0xAB    │     │   0xCD    │
│ events: 50│     │ root: 0xCD│     │ root: 0xEF│
└───────────┘     │ events: 80│     │ events: 42│
                  └───────────┘     └───────────┘
```

This creates an **immutable linked chain** of proof batches. Any tampering breaks the chain.

## Why SPHINCS+ for Proofs

| Algorithm | Based On | Key Assumption | Signature Size | Best For |
|-----------|----------|---------------|----------------|----------|
| Ed25519 | Elliptic curves | DLP is hard | 64 bytes | Fast daily verification |
| ML-DSA-65 | Lattices | MLWE is hard | 3,309 bytes | General PQ signatures |
| **SPHINCS+-256s** | **Hash functions only** | **Hash collision resistance** | **29,792 bytes** | **Long-term proofs** |

SPHINCS+ has **zero mathematical assumptions** beyond hash function security. Even if a flaw is found in lattice-based cryptography (breaking ML-DSA), SPHINCS+ signatures remain valid. For proofs that must be verifiable in 2046, this is the only responsible choice.

The trade-off is signature size (~30 KB). Acceptable for Merkle roots generated once per minute.

## On-Chain Verification (Trustless)

Anyone can verify a proof directly on-chain — no API, no auth, no trust required:

```solidity
// KPPEAnchor.sol — Base Mainnet
// 0x3B53F7044E47766769156bF210c2661F03Df45dd

function verifyProofView(
  uint256 _batchId,
  bytes32 _eventHash,
  bytes32[] calldata _proof,
  uint256 _leafIndex
) external view returns (bool);
```

The contract reconstructs the Merkle root from `_eventHash` + `_proof` using sorted-pair hashing, and compares it against the stored root for `_batchId`.

## Off-Chain Verification (API)

```http
GET /proofs/verify/{batchId}/{eventHash}

# No authentication required

Response:
{
  "valid": true,
  "event_hash": "0xabc...",
  "merkle_root": "0xdef...",
  "merkle_proof": ["0x123...", "0x456...", "0x789..."],
  "leaf_index": 42,
  "tx_hash": "0x650...",
  "chain": "base",
  "contract": "0x3B53...45dd"
