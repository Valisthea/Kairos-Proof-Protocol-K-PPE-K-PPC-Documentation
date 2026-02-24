# K-PPE Architecture

## Deployment Architecture

```
┌─────────────────────────────────────────────────────┐
│                   CLIENT LAYER                       │
│   Browser │ dApp │ Mobile │ SDK │ Direct API         │
└──────────────────────┬──────────────────────────────┘
                       │ TLS 1.3 + ML-KEM-768 (hybrid)
                       │
┌──────────────────────▼──────────────────────────────┐
│                 RAILWAY COMPUTE                       │
│                                                      │
│  ┌──────────────┐  ┌──────────────┐  ┌───────────┐  │
│  │  Edge Proxy   │  │ Auth Service  │  │ API GW    │  │
│  │  TLS termina- │  │ SIWE verify   │  │ Rate limit│  │
│  │  tion +       │→ │ JWT dual-sign │→ │ Anti-repla│  │
│  │  PQ handshake │  │ PQ key mgmt   │  │ Dual HMAC │  │
│  └──────────────┘  └──────────────┘  └─────┬─────┘  │
│                                            │         │
│  ┌──────────────┐  ┌──────────────┐        │         │
│  │  Proof Engine │  │  Encryption  │        │         │
│  │  Merkle trees │  │  Envelope    │◄───────┘         │
│  │  SPHINCS+ sig │  │  AES + XCha  │                  │
│  │  Batch proc.  │  │  Zero-know.  │                  │
│  └──────┬───────┘  └──────┬───────┘                  │
│         │                  │                          │
└─────────┼──────────────────┼──────────────────────────┘
          │                  │
          │  ciphertext only │
          ▼                  ▼
┌─────────────────────────────────────────────────────┐
│                    SUPABASE                          │
│                                                      │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐  │
│  │ kpe_users    │  │ kpe_proof_  │  │ kpe_event_  │  │
│  │ (encrypted)  │  │ batches     │  │ proofs      │  │
│  │              │  │ (encrypted) │  │ (encrypted) │  │
│  └─────────────┘  └─────────────┘  └─────────────┘  │
│                                                      │
│  ┌─────────────┐  ┌─────────────┐                    │
│  │ kpe_nonces   │  │ kpe_signing │                    │
│  │              │  │ _keys       │                    │
│  └─────────────┘  └─────────────┘                    │
│                                                      │
│  Row Level Security: ALL tables locked               │
│  Access: Service key only (from Railway)             │
└─────────────────────────────────────────────────────┘
```

## Data Flow: Proof Creation

```
1. Client sends events → POST /proofs/batch (with JWT + HMAC headers)
2. API Gateway validates: rate limit → anti-replay → dual HMAC → JWT
3. Proof Engine: hash events → build Merkle tree → sign root (Ed25519 + SPHINCS+)
4. Encryption: envelope encrypt each proof + batch
5. Storage: write ciphertext to Supabase
6. Response: return batch ID + Merkle root + individual proofs to client
```

## Data Flow: Proof Verification (Public)

```
1. Anyone sends → GET /proofs/verify/{batchId}/{eventHash} (no auth)
2. K-PPE fetches encrypted proof from Supabase
3. K-PPE decrypts proof
4. K-PPE verifies Merkle branch against root
5. K-PPE verifies both signatures (Ed25519 + SPHINCS+)
6. Response: verification result + proof data
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
| Merkle Trees | `merkletreejs` | Deterministic construction |
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
