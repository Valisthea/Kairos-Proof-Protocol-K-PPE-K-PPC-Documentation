# Roadmap Overview

The Kairos Proof Protocol roadmap spans from initial PoC to a fully decentralized, cross-chain proof network. Seven phases, from months to years, each building on the previous.

```
2026 Q1-Q2          2026 Q3-Q4          2027 Q1-Q2          2027 Q3+
━━━━━━━━━━          ━━━━━━━━━━          ━━━━━━━━━━          ━━━━━━━━
PHASE 1              PHASE 2             PHASE 3              PHASE 4
Foundation           Proof Engine        PQ Production         K-PPC Launch
                                                               
2028 Q1-Q2          2028 Q3+            2029+                 2030+
━━━━━━━━━━          ━━━━━━━━            ━━━━━                 ━━━━━
PHASE 5              PHASE 6             PHASE 7              ULTIMATE
ZK Proofs            Cross-Chain         Decentralized        VISION
                                         Proof Network
```

## Progress Tracker

| Phase | Name | Status | Timeline |
|-------|------|--------|----------|
| 1 | Foundation | ✅ **Core Deployed** | Q1-Q2 2026 |
| 2 | Proof Engine Live | 🔨 **In Progress** | Q3-Q4 2026 |
| 3 | Post-Quantum Production | 📋 Planned | Q1-Q2 2027 |
| 4 | K-PPC Chain Launch | 🔬 Research — **Asterchain L1 Mainnet** | Q3 2027+ |
| 5 | ZK Proofs | 🔬 Research | Q1-Q2 2028 |
| 6 | Cross-Chain | 💡 Concept | Q3 2028+ |
| 7 | Decentralized Proof Network | 💡 Concept | 2029+ |
| ∞ | Ultimate Vision | 🌐 North Star | 2030+ |

---

# Phase 1 — Foundation

**Timeline**: Q1-Q2 2026 (12-16 weeks)
**Status**: ✅ Core Deployed (Base Mainnet)
**Goal**: Ship a working K-PPE with hybrid auth and basic proof generation

### Phase 1 — Deployed Components

| Component | Status | Details |
|-----------|--------|---------|
| KPPEAnchor smart contract | ✅ Deployed | `0x3B53F7044E47766769156bF210c2661F03Df45dd` — Base Mainnet, verified Basescan |
| App Registry contract | ✅ Deployed | `0xcbf3e71fb2E09929682b8448442d184f8E1E37B8` — Base Mainnet |
| Local Merkle tree builder | ✅ Deployed | `src/kpe/merkle.ts` — sorted-pair Keccak-256, max 256 leaves |
| OnChainSender.anchorMerkleRoot() | ✅ Deployed | Only root hash (32 bytes) on-chain |
| BatchManager.flushKPPE() | ✅ Deployed | Replaces deprecated sendBatch() |
| KPPE_MODE routing | ✅ Deployed | kppe / legacy / disabled |
| Chain of trust (prevRoot) | ✅ Deployed | Immutable linked batch chain |
| Supabase schema | ✅ Deployed | merkle_batches, event_receipts, events with client_hash |
| KA-HAP 7-layer security | ✅ Deployed | Database tables + relayer integration |
| Test batch #1 | ✅ Confirmed | TX: `0x6506230...e401c5e5bef` |

## Deliverables (Original Checklist)

### 1.1 — Project Scaffold (Week 1-2)
- [x] Repository structure defined
- [x] Claude Code prompts generated
- [x] TypeScript project initialized
- [x] Dependencies installed
- [ ] CI/CD pipeline (GitHub Actions)
- [x] README + CLAUDE.md

### 1.2 — Classical Cryptography Module (Week 2-3)
- [ ] Ed25519 key generation, sign, verify
- [ ] HMAC-SHA256
- [ ] AES-256-GCM encrypt/decrypt
- [ ] HKDF key derivation
- [ ] Test suite with round-trip validation

### 1.3 — Post-Quantum Module (Simulated) (Week 3-4)
- [ ] ML-DSA-65 interface + simulated implementation
- [ ] ML-KEM-768 interface + simulated implementation
- [ ] SPHINCS+-256s interface + simulated implementation
- [ ] KMAC-256 implementation
- [ ] Correct NIST key/signature sizes
- [ ] Clear `PQ_STATUS: SIMULATED` flag everywhere
- [ ] Test suite validating interfaces

### 1.4 — Hybrid Crypto Combiner (Week 4-5)
- [ ] hybridSign / hybridVerify (strict + degraded modes)
- [ ] hybridEncrypt / hybridDecrypt (single + double)
- [ ] hybridKeyExchange
- [ ] Tests: one-side failure scenarios

### 1.5 — SIWE Authentication (Week 5-6)
- [ ] SIWE message generation (EIP-4361)
- [ ] Signature verification
- [ ] Nonce management (generate, store, validate, expire)
- [ ] Wallet address validation + normalization
- [ ] Auth routes: `/auth/challenge`, `/auth/verify`

### 1.6 — Dual-Signed JWT (Week 6-7)
- [ ] KPP-HYBRID-1 JWT format (4-part token)
- [ ] Sign with Ed25519 + ML-DSA-65
- [ ] Verify both signatures (strict mode)
- [ ] Refresh token flow
- [ ] Auth middleware for Express
- [ ] Supabase JWT bridge

### 1.7 — API Security (Week 7-8)
- [ ] Double HMAC middleware (SHA-256 + KMAC-256)
- [ ] Anti-replay (nonce + timestamp + memory store)
- [ ] Rate limiting (per-wallet + global)
- [ ] Security pipeline (rate → replay → HMAC → JWT)

### 1.8 — Proof Engine v1 (Week 8-10)
- [x] Event hashing (Keccak-256, deterministic) — client-side via SDK
- [x] Merkle tree construction — local builder `src/kpe/merkle.ts`
- [ ] Hybrid proof signing (Ed25519 + SPHINCS+ simulated)
- [x] Individual proof generation (Merkle branches)
- [x] Proof verification — on-chain via `verifyProofView()` + off-chain API
- [x] On-chain anchoring — `KPPEAnchor.anchorBatch()` on Base Mainnet
- [ ] Routes: `/proofs/batch`, `/proofs/:id`, `/proofs/verify/:batchId/:eventHash`

### 1.9 — Encrypted Storage (Week 10-12)
- [ ] Envelope encryption (DEK/KEK pattern)
- [ ] Double envelope encryption (for proofs)
- [ ] Supabase schema + migrations
- [ ] Zero-knowledge storage (encrypt before insert, decrypt after read)
- [ ] RLS on all tables

### 1.10 — Assembly + Deploy (Week 12-14)
- [ ] Express app assembly
- [ ] Health endpoints (`/health`, `/health/crypto`)
- [ ] Environment validation (Zod)
- [ ] Key generation script
- [ ] Integration tests (full flow)
- [ ] Railway deployment config
- [ ] Documentation (ARCHITECTURE.md, API.md, THREAT-MODEL.md, DEPLOYMENT.md)

### 1.11 — External Testing (Week 14-16)
- [ ] Internal testing with Kairos Analytics
- [ ] Invite 5-10 beta testers
- [ ] Bug fixes + performance tuning
- [ ] Phase 1 complete ✅

## Phase 1 Success Criteria
- All crypto module tests pass
- Full auth flow works (SIWE → JWT → API call → proof generation)
- Public proof verification endpoint works without auth
- Zero plaintext data in Supabase
- Health endpoints report all-green
- Deployed on Railway and accessible

---

# Phase 2 — Proof Engine Live

**Timeline**: Q3-Q4 2026 (12-16 weeks)
**Status**: 📋 Planned
**Goal**: Production-ready proof engine with real-time indexing and Kairos Analytics integration

## Deliverables

### 2.1 — Real-Time On-Chain Indexing
- [ ] WebSocket connections to BNB Chain RPC nodes
- [ ] AsterDex event listener (trades, liquidations, funding rates)
- [ ] Generic EVM event listener (configurable per contract/event)
- [ ] Event queue (Redis) for reliable processing
- [ ] Automatic reconnection + missed block detection

### 2.2 — Automated Batch Processing
- [ ] Cron-based batch generation (every 60 seconds)
- [ ] Configurable batch intervals per client
- [ ] Batch queuing + retry logic
- [ ] Monitoring + alerting on batch failures

### 2.3 — Key Rotation Automation
- [ ] Automated key age checking (hourly cron)
- [ ] Grace period management (dual-valid keys)
- [ ] JWT signing key rotation (30 days)
- [ ] API secret rotation (90 days)
- [ ] Proof signing key rotation (6-24 months)
- [ ] Rotation audit log

### 2.4 — Redis Integration (Production)
- [ ] Redis nonce store (replace in-memory)
- [ ] Redis rate limiter (replace in-memory)
- [ ] Redis event queue
- [ ] Redis session cache

### 2.5 — Kairos Analytics Dashboard
- [ ] Real-time analytics dashboard (React frontend)
- [ ] Live event feed
- [ ] Proof verification widget ("Verified by Kairos" badge)
- [ ] API key management for clients
- [ ] Usage analytics

### 2.6 — Client SDK v1
- [ ] `@kairos/proof-sdk` npm package
- [ ] TypeScript/JavaScript SDK
- [ ] Authentication helpers
- [ ] Proof submission + verification
- [ ] Webhook registration
- [ ] Documentation + examples

### 2.7 — Permanent Storage (Arweave)
- [ ] Arweave upload integration
- [ ] Proof batch → Arweave transaction
- [ ] Arweave TX ID stored alongside Supabase proof
- [ ] Fallback retrieval: if Supabase down, fetch from Arweave
- [ ] Cost monitoring + budget alerts

### 2.8 — On-Chain Anchoring v1
- [x] ~~BNB Chain~~ **Base Mainnet** smart contract for Merkle root anchoring — KPPEAnchor deployed
- [x] Batch root + eventCount + prevRoot published on-chain
- [x] Verification contract — `verifyProofView()` (anyone can verify on-chain)
- [ ] Gas optimization (batch multiple roots per transaction)

### 2.9 — Multi-Client Support
- [ ] API key system (multiple clients, isolated proofs)
- [ ] Per-client rate limits + quotas
- [ ] Client dashboard for proof management
- [ ] Billing infrastructure (usage-based pricing)

## Phase 2 Success Criteria
- Real-time event indexing running 24/7
- Automated batch processing every 60 seconds
- Key rotation fully automated
- Arweave backup for all proofs
- Merkle roots anchored on BNB Chain
- SDK published on npm
- At least 3 external clients integrated
- Kairos Analytics dashboard live

---

# Phase 3 — Post-Quantum Production

**Timeline**: Q1-Q2 2027 (12-16 weeks)
**Status**: 📋 Planned
**Goal**: Replace simulated PQ algorithms with production liboqs implementations

## Deliverables

### 3.1 — liboqs Integration
- [ ] Compile liboqs for Node.js (native bindings or WASM)
- [ ] ML-DSA-65 production implementation
- [ ] ML-KEM-768 production implementation
- [ ] SPHINCS+-256s production implementation
- [ ] NIST test vector validation (FIPS 203, 204, 205)
- [ ] Performance benchmarks (sign/verify latency)

### 3.2 — Client-Side PQ (WASM)
- [ ] liboqs compiled to WebAssembly
- [ ] Browser-compatible PQ key operations
- [ ] Client-side ML-KEM for hybrid TLS handshake
- [ ] Size optimization (<500KB WASM bundle)
- [ ] Lazy loading (PQ module loaded on demand)

### 3.3 — Hybrid TLS Production
- [ ] oqs-provider for OpenSSL 3.x on Railway
- [ ] ML-KEM-768 + X25519 hybrid key exchange
- [ ] Fallback handling for non-PQ clients
- [ ] Performance testing (handshake latency)

### 3.4 — Migration: Simulated → Production
- [ ] Seamless algorithm swap (same interfaces)
- [ ] Re-sign existing proofs with production SPHINCS+ (optional)
- [ ] Re-generate user PQ keys with production ML-DSA
- [ ] Update `PQ_STATUS: SIMULATED` → `PQ_STATUS: PRODUCTION`
- [ ] Announcement + documentation update

### 3.5 — External Security Audit
- [ ] Engage auditing firm (Trail of Bits, OpenZeppelin, Cure53)
- [ ] Full protocol audit (crypto, auth, proof engine)
- [ ] Penetration testing
- [ ] Fix all findings
- [ ] Publish audit report (transparency)

### 3.6 — HSM Integration
- [ ] Hardware Security Module for signing keys
- [ ] AWS CloudHSM or YubiHSM integration
- [ ] Keys never leave HSM boundary
- [ ] Performance testing with HSM latency

## Phase 3 Success Criteria
- All PQ algorithms running with production liboqs
- NIST test vectors passing for all three algorithms
- External audit completed and published
- Hybrid TLS operational in production
- Zero simulated components remaining
- PQ_STATUS: PRODUCTION across all endpoints

---

# Phase 4 — K-PPC Chain Launch

**Timeline**: Q3 2027+ (6-12 months)
**Status**: 🔬 Research — **Target: Asterchain L1 Mainnet**
**Goal**: Launch the dedicated Kairos Proof Protocol Chain on Asterchain L1

## Deliverables

### 4.1 — Chain Architecture Finalization
- [ ] OP Stack vs Cosmos SDK vs custom evaluation (complete)
- [ ] Sequencer design
- [ ] State transition function (proof-specific)
- [ ] Data availability layer selection (EigenDA, Celestia, or BNB DA)

### 4.2 — Core Chain Development
- [ ] Genesis configuration
- [ ] Proof storage module (native, not smart contract)
- [ ] Proof query module (by hash, batch, time, source)
- [ ] Verification module (Merkle + signature check at protocol level)
- [ ] Cross-chain anchor module (publish roots to BNB Chain)

### 4.3 — Sequencer Launch
- [ ] Kairos-operated sequencer (centralized Phase 1)
- [ ] Batch poster to BNB Chain
- [ ] State root publication
- [ ] Explorer / block viewer

### 4.4 — Bridge from K-PPE (Base Mainnet → Asterchain L1)
- [ ] K-PPE → K-PPC proof submission pipeline (bridge Base → Asterchain)
- [ ] Automatic anchoring of all proof batches
- [ ] Dual storage: Supabase + K-PPC + Arweave (triple redundancy)
- [ ] Fallback chain: if K-PPC down, Arweave + Base Mainnet anchoring continues

### 4.5 — Verification Contract on BNB Chain
- [ ] Smart contract for K-PPC state root verification
- [ ] Cross-chain proof verification (verify K-PPC proof from BNB Chain)
- [ ] Gas-optimized batch verification

### 4.6 — Testnet
- [ ] Public testnet launch
- [ ] Testnet faucet
- [ ] Integration testing with K-PPE
- [ ] Community testing program
- [ ] Bug bounty

### 4.7 — Mainnet Launch
- [ ] Genesis block
- [ ] K-PPE migration to dual-write (Supabase + K-PPC)
- [ ] Public announcement
- [ ] Documentation update

## Phase 4 Success Criteria
- K-PPC chain running with centralized sequencer
- All K-PPE proofs automatically anchored on K-PPC
- Cross-chain verification from BNB Chain operational
- Triple redundancy: Supabase + K-PPC + Arweave
- Public explorer available
- Testnet → Mainnet migration complete

---

# Phase 5 — ZK Proofs

**Timeline**: Q1-Q2 2028 (6-9 months)
**Status**: 🔬 Research
**Goal**: Add zero-knowledge proof circuits to K-PPC for privacy-preserving verification

## Deliverables

### 5.1 — ZK Toolchain Setup
- [ ] SP1 or Risc0 development environment
- [ ] Rust circuit development framework
- [ ] Testing infrastructure (proof generation + verification)
- [ ] Benchmark suite (proving time, verification cost, proof size)

### 5.2 — Circuit: Proof of Event Validity
- [ ] "This event exists in the Merkle tree" without revealing event data
- [ ] Input: event data (private), Merkle root (public)
- [ ] Output: boolean (valid) + ZK proof
- [ ] On-chain verifier contract on BNB Chain

### 5.3 — Circuit: Proof of Aggregate
- [ ] "The sum of values across N events equals X"
- [ ] Used for: verifiable volume, TVL, active users
- [ ] Privacy: individual events not revealed
- [ ] On-chain verifier

### 5.4 — Circuit: Proof of Solvency
- [ ] "Assets > Liabilities for this entity"
- [ ] Used for: exchange proof-of-reserves, fund solvency
- [ ] Privacy: individual positions not revealed
- [ ] On-chain verifier

### 5.5 — Circuit: Proof of Compliance
- [ ] "All transactions met criteria X" (e.g., no sanctioned addresses)
- [ ] Used for: regulatory compliance without revealing all transactions
- [ ] Privacy: transaction details not revealed, only compliance status
- [ ] On-chain verifier

### 5.6 — K-PPC ZK Rollup Transition
- [ ] Replace optimistic/centralized proof posting with ZK proofs
- [ ] K-PPC state transitions verified by ZK proof on BNB Chain
- [ ] Proof generation pipeline (recursive STARKs → SNARK for verification)
- [ ] Gas cost optimization for on-chain verification

### 5.7 — SDK Update
- [ ] `@kairos/proof-sdk` v2 with ZK proof verification
- [ ] Client-side ZK proof generation (for privacy-preserving queries)
- [ ] Documentation + examples

## Phase 5 Success Criteria
- At least 4 ZK circuits operational
- On-chain verification contracts deployed on BNB Chain
- K-PPC running as ZK rollup (not optimistic)
- SDK v2 with ZK support published
- External audit of ZK circuits
- Proving time < 30 seconds for event validity proofs

---

# Phase 6 — Cross-Chain

**Timeline**: Q3 2028+ (6-12 months)
**Status**: 💡 Concept
**Goal**: Extend K-PPE and K-PPC to support multiple source chains and cross-chain proof verification

## Deliverables

### 6.1 — Multi-Chain Indexing
- [ ] Ethereum mainnet indexer
- [ ] Arbitrum indexer
- [ ] Base indexer
- [ ] Polygon indexer
- [ ] Solana adapter (non-EVM)
- [ ] Configurable chain addition (plug-and-play indexer)

### 6.2 — Cross-Chain Proof Format
- [ ] Universal proof format (chain-agnostic)
- [ ] Source chain attestation (prove which chain the event came from)
- [ ] Cross-chain Merkle aggregation (events from multiple chains in one tree)

### 6.3 — Cross-Chain Verification Bridges
- [ ] Verify K-PPC proofs from Ethereum
- [ ] Verify K-PPC proofs from Arbitrum
- [ ] Verify K-PPC proofs from Base
- [ ] Light client verification (no full node required)

### 6.4 — Multi-Chain Kairos Analytics
- [ ] Dashboard: unified view across all supported chains
- [ ] Cross-chain event correlation
- [ ] Multi-chain proof generation in single batches

### 6.5 — Interoperability Standards
- [ ] EIP proposal for standardized proof format
- [ ] Collaboration with other proof protocols
- [ ] Open specification document

## Phase 6 Success Criteria
- K-PPE indexing events from 5+ chains
- Cross-chain proof verification operational
- Universal proof format documented and implemented
- Multi-chain dashboard live

---

# Phase 7 — Decentralized Proof Network

**Timeline**: 2029+
**Status**: 💡 Concept
**Goal**: Transform K-PPC from a Kairos-operated chain into a fully decentralized, permissionless proof network

## Deliverables

### 7.1 — Decentralized Sequencer
- [ ] Transition from Kairos-operated to shared/elected sequencer
- [ ] Integration with shared sequencer network (Espresso/Astria)
- [ ] MEV protection for proof ordering

### 7.2 — Validator Network
- [ ] Open validator registration
- [ ] Stake-based validator selection
- [ ] Slashing for misbehavior (data withholding, invalid proofs)
- [ ] Geographic distribution requirements
- [ ] Minimum 50 validators across 3+ continents

### 7.3 — Permissionless Proof Submission
- [ ] Anyone can submit proofs to K-PPC (not just K-PPE)
- [ ] Proof validation at protocol level
- [ ] Fee market for proof storage
- [ ] Proof indexing by anyone

### 7.4 — Governance
- [ ] On-chain governance for protocol upgrades
- [ ] Community proposals for new proof types
- [ ] Parameter adjustment (batch size, fees, rotation schedules)
- [ ] No governance token (vote with stake, BNB-based)

### 7.5 — Proof Marketplace
- [ ] Provers offer proof generation services
- [ ] Clients request proofs
- [ ] Competitive pricing
- [ ] Reputation system for provers

### 7.6 — Developer Ecosystem
- [ ] Grant program for builders on K-PPC
- [ ] Hackathon series
- [ ] Developer documentation portal
- [ ] Reference implementations in multiple languages (Rust, Go, Python, JS)

## Phase 7 Success Criteria
- K-PPC operated by 50+ independent validators
- Permissionless proof submission live
- Governance active with community participation
- Multiple independent provers in the marketplace
- Developer grant program funding 10+ projects

---

# Ultimate Vision

## 2030 and Beyond

```
                    THE KAIROS VISION
                    ═════════════════

     TODAY                          THE FUTURE
     ═════                          ══════════

  "Trust us, our            "Here's the proof.
   data is correct"          Verify it yourself.
                             On any chain.
                             From any device.
                             In any country.
                             Today or in 50 years.
                             We don't ask you to trust us.
                             We give you math."
```

## The End State

### Proof as a Primitive
Just as blockchain gave us "trustless transactions," Kairos gives the world "trustless data." Proof generation becomes a standard building block — as common as HTTPS, as expected as SSL certificates.

### Universal Verifiability
Any event, on any blockchain, can be proven and verified by anyone, anywhere, without trusting any intermediary. The proofs are post-quantum secure, meaning they survive the cryptographic transition that will break today's standards.

### The Proof Layer of Web3
```
Application Layer:    dApps, DeFi, Gaming, Social, DAOs
                          │
Data Layer:           Oracles, Indexers, Analytics, APIs
                          │
PROOF LAYER:          ███ KAIROS PROOF PROTOCOL ███
                          │
Consensus Layer:      Ethereum, BNB Chain, Solana, etc.
                          │
Network Layer:        P2P, Gossip, Block propagation
```

Kairos Proof Protocol becomes the **proof layer** of the Web3 stack — the missing piece between raw blockchain data and application trust.

### Post-Quantum Standard
When quantum computers arrive and break ECDSA, every Web3 project will need to migrate their cryptography. Kairos will already be there. Projects that integrated K-PPE early will have a seamless transition. Those that didn't will scramble.

### Beyond Blockchain
The proof protocol isn't limited to blockchain events. Any verifiable data source can generate proofs:
- IoT sensor data (prove a temperature reading happened)
- Legal documents (prove a contract was signed at a specific time)
- Scientific data (prove an experiment produced these results)
- Election systems (prove a vote was counted correctly)
- Supply chain (prove a product passed through specific checkpoints)

The technology is chain-agnostic, data-agnostic, and future-proof.

## The Narrative

**"Verify Everything. Trust Nothing. Prove Forever."**

This is not a tagline. It's a technical specification.

- **Verify Everything**: Public verification endpoints, no auth required
- **Trust Nothing**: Zero-knowledge storage, hybrid crypto, decentralized validation
- **Prove Forever**: SPHINCS+ signatures valid 20+ years, K-PPC permanent storage

Kairos Proof Protocol is the infrastructure that makes this possible. Not for one product. Not for one chain. For everyone.
