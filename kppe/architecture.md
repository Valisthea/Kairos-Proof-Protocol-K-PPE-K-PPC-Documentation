# K-PPE Architecture

> **Status: DEPLOYED on Base Mainnet** — KPPEAnchor contract verified on Basescan.

## Deployment Architecture

```
┌─────────────────────────────────────────────────────┐
│                   CLIENT LAYER                       │
│   Browser │ dApp │ Mobile │ SDK │ Direct API         │
│   Hash born here → keccak256(event) = client_hash   │
└──────────────────────┬──────────────────────────────┘
                       │ TLS 1.3 + ML-KEM-768 (hybrid)
                       │
┌──────────────────────▼──────────────────────────────┐
│                 RELAYER (Railway)                     │
│                                                      │
│  ┌──────────────┐  ┌──────────────┐  ┌───────────┐  │
│  │  Edge Proxy   │  │ Auth Service  │  │ API GW    │  │
│  │  TLS termina- │  │ SIWE verify   │  │ Rate limit│  │
│  │  tion +       │→ │ JWT dual-sign │→ │ Anti-repla│  │
│  │  PQ handshake │  │ PQ key mgmt   │  │ Dual HMAC │  │
│  └──────────────┘  └──────────────┘  └─────┬─────┘  │
│                                            │         │
│  ┌──────────────────┐  ┌──────────────┐    │         │
│  │  Local Merkle     │  │  Encryption  │    │         │
│  │  Builder          │  │  Envelope    │◄───┘         │
│  │  src/kpe/merkle.ts│  │  AES + XCha  │              │
│  │  Sorted-pair hash │  │  Zero-know.  │              │
│  │  Max 256 leaves   │  └──────┬───────┘              │
│  └──────┬───────────┘          │                      │
│         │ Merkle root          │                      │
│         ▼                      │                      │
│  ┌──────────────────┐          │                      │
│  │  OnChainSender    │          │                      │
│  │  anchorMerkle-    │          │                      │
│  │  Root()           │          │                      │
│  └──────┬───────────┘          │                      │
└─────────┼──────────────────────┼──────────────────────┘
          │                      │
          │ 32 bytes (root)      │ proofs + receipts
          ▼                      ▼
┌───────────────────┐  ┌─────────────────────────────┐
│  BASE MAINNET      │  │         SUPABASE              │
│  Chain 8453        │  │                               │
│                    │  │  ┌──────────────┐             │
│  KPPEAnchor        │  │  │ merkle_      │             │
│  0x3B53...45dd     │  │  │ batches      │             │
│                    │  │  └──────────────┘             │
│  anchorBatch(      │  │  ┌──────────────┐             │
│    appId,          │  │  │ event_       │             │
│    merkleRoot,     │  │  │ receipts     │             │
│    eventCount      │  │  └──────────────┘             │
│  )                 │  │  ┌──────────────┐             │
│                    │  │  │ events       │             │
│  verifyProofView(  │  │  │ (client_hash,│             │
│    batchId,        │  │  │  batch_id,   │             │
│    eventHash,      │  │  │  batched)    │             │
│    proof[],        │  │  └──────────────┘             │
│    leafIndex       │  │                               │
│  )                 │  │  KA-HAP Tables:               │
│                    │  │  kpe_proof_batches,            │
│  Events:           │  │  kpe_event_proofs,             │
│  KPPEMerkle-       │  │  kpe_signing_keys,             │
│  Anchored(         │  │  kpe_verifications,             │
│    batchId,        │  │  kpe_status, kpe_nonces        │
│    appId,          │  │                               │
│    merkleRoot,     │  │  Row Level Security: ALL       │
│    eventCount,     │  │  Access: Service key only      │
│    prevRoot        │  └───────────────────────────────┘
│  )                 │
└───────────────────┘
```

## Data Flow: K-PPE Trustless Anchoring (Production — KPPE_MODE=kppe)

```
1. SDK hashes event client-side → keccak256(event) = client_hash
2. Client sends event + client_hash → POST /events (with JWT + HMAC headers)
3. API Gateway validates: rate limit → anti-replay → dual HMAC → JWT
4. Relayer stores event in Supabase (events table, with client_hash)
5. BatchManager cron (every 60s):
   a. Fetch unbatched events for each app
   b. Build Merkle tree locally (src/kpe/merkle.ts):
      - Leaves = client_hash values (or server-generated fallback)
      - Sorted-pair hashing: keccak256(abi.encodePacked(min(a,b), max(a,b)))
      - Pad to power-of-2 with zero hashes
      - Max 256 leaves per batch
   c. Verify all proofs locally (self-test)
   d. Call OnChainSender.anchorMerkleRoot():
      - KPPEAnchor.anchorBatch(appId, merkleRoot, eventCount)
      - Only 32 bytes (root) + uint256 goes on-chain
      - prevRoot automatically tracked in contract
   e. Save to Supabase: merkle_batches + event_receipts
   f. Mark events as batched
6. Response: batch_id + merkle_root + tx_hash + individual proofs
```

## Data Flow: On-Chain Verification (Public)

```
1. Anyone calls → KPPEAnchor.verifyProofView(batchId, eventHash, proof[], leafIndex)
2. Contract reconstructs root from eventHash + proof using sorted-pair hashing
3. Contract compares reconstructed root with stored root
4. Returns: bool (valid or not)
   → No auth required, fully trustless, works from any chain via RPC
```

## Data Flow: Off-Chain Verification (API)

```
1. Anyone sends → GET /proofs/verify/{batchId}/{eventHash} (no auth)
2. Relayer fetches proof from Supabase (event_receipts + merkle_batches)
3. Relayer verifies Merkle branch against stored root
4. Relayer optionally verifies hybrid signatures (Ed25519 + SPHINCS+)
5. Response: verification result + proof data + tx_hash for on-chain check
```

---

# Tech Stack

## Runtime

| Component | Technology | Version |
|-----------|-----------|---------|
| Language | TypeScript | 5.x |
| Runtime | Node.js | 20 LTS |
| Framework | Express.js | 4.x |
| Process Manager | Railway (native) | — |

## Cryptography

| Function | Library | Notes |
|----------|---------|-------|
| Ed25519 | `@noble/ed25519` | Audited, pure JS |
| ECDSA verification | `ethers` v6 | For SIWE signature verification |
| SHA-256, HMAC | `@noble/hashes` | Audited, pure JS |
| Keccak-256 | `keccak256` npm | EVM-compatible hashing |
| AES-256-GCM | Node.js `crypto` | Native, hardware-accelerated |
| XChaCha20-Poly1305 | `libsodium-wrappers` | Audited, WASM |
| HKDF | `@noble/hashes/hkdf` | Key derivation |
| Merkle Trees | Local builder (`src/kpe/merkle.ts`) | Sorted-pair Keccak-256, EVM-compatible |
| KMAC-256 | `js-sha3` | Keccak-based MAC |
| ML-DSA-65 | Simulated (Phase 1) → `liboqs` (Phase 3) | NIST FIPS 204 |
| ML-KEM-768 | Simulated (Phase 1) → `liboqs` (Phase 3) | NIST FIPS 203 |
| SPHINCS+-256s | Simulated (Phase 1) → `liboqs` (Phase 3) | NIST FIPS 205 |

## Infrastructure

| Service | Provider | Role |
|---------|----------|------|
| Compute | Railway | Auth, API, Proof Engine, Encryption |
| Database | Supabase (PostgreSQL) | Encrypted data storage |
| Cache | Redis (Railway addon) | Nonce store, rate limiting |
| TLS | Let's Encrypt + Railway | Certificate management |
| CI/CD | GitHub Actions | Build, test, deploy |

## Authentication

| Component | Library |
|-----------|---------|
| SIWE (EIP-4361) | `siwe` npm |
| Ethereum utils | `ethers` v6 |
| JWT base | `jose` npm |
| Validation | `zod` |

## Monitoring

| Tool | Purpose |
|------|---------|
| Pino | Structured logging |
| Railway Metrics | Infrastructure monitoring |
| `/health` endpoint | Service health check |
| `/health/crypto` | Crypto module self-test |
