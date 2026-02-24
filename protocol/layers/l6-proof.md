# L6 — Proof Engine Layer

## Purpose
Generate cryptographic proofs of on-chain events using Merkle trees and hybrid signatures. Proofs are publicly verifiable and designed to remain valid for 20+ years.

## Proof Generation Flow

```
On-Chain Events (trades, liquidations, oracle updates...)
          │
          ▼
    ┌─── Hash each event ──────── Keccak-256 (EVM-compatible)
    │
    ▼
    ┌─── Build Merkle Tree ────── Sorted leaves, deterministic
    │
    ▼
    ┌─── Sign Root ────────────── Ed25519 (fast, daily verification)
    │                              + SPHINCS+-256s (PQ, 20+ year validity)
    │
    ▼
    ┌─── Generate Proofs ──────── Individual Merkle branch per event
    │
    ▼
    ┌─── Store ────────────────── Encrypted in Supabase (L5)
    │                              + Public verification endpoint
    ▼
    [FUTURE: Anchor on K-PPC]
```

## Event Hashing

Events are serialized deterministically (sorted keys, canonical JSON) before hashing:

```typescript
function hashEvent(event: OnChainEvent): string {
  const canonical = JSON.stringify(event, Object.keys(event).sort());
  return keccak256(canonical);
}
```

Using Keccak-256 ensures compatibility with the EVM ecosystem. Any Ethereum client can independently verify the hash.

## Merkle Tree Construction

```
                    Root (signed)
                   /             \
              H(0,1)             H(2,3)
             /      \           /      \
          H(0)     H(1)     H(2)     H(3)
           │        │        │        │
        Event_0  Event_1  Event_2  Event_3
```

- Leaves are sorted by hash value for deterministic tree construction
- Same events always produce the same root
- Batches are generated every 60 seconds (configurable)

## Why SPHINCS+ for Proofs

| Algorithm | Based On | Key Assumption | Signature Size | Best For |
|-----------|----------|---------------|----------------|----------|
| Ed25519 | Elliptic curves | DLP is hard | 64 bytes | Fast daily verification |
| ML-DSA-65 | Lattices | MLWE is hard | 3,309 bytes | General PQ signatures |
| **SPHINCS+-256s** | **Hash functions only** | **Hash collision resistance** | **29,792 bytes** | **Long-term proofs** |

SPHINCS+ has **zero mathematical assumptions** beyond hash function security. Even if a flaw is found in lattice-based cryptography (breaking ML-DSA), SPHINCS+ signatures remain valid. For proofs that must be verifiable in 2046, this is the only responsible choice.

The trade-off is signature size (~30 KB). Acceptable for Merkle roots generated once per minute.

## Public Verification

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
