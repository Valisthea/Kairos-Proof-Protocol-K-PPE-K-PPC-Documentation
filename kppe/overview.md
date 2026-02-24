# What is K-PPE?

K-PPE (Kairos Proof Protocol Engine) is the core software engine that implements the Kairos Proof Protocol. It's a TypeScript/Node.js application that runs on Railway and provides:

- Hybrid authentication (SIWE + post-quantum)
- Cryptographic proof generation and verification
- Encrypted data storage via Supabase
- Public API for proof verification

**Repository**: `kairos-proof-protocol-engine` (public, all rights reserved)

## Who Uses K-PPE

K-PPE is infrastructure. It can be consumed by:
- **Kairos Analytics** (first-party consumer)
- **Any dApp** via REST API or SDK
- **Any protocol** that needs verifiable event proofs
- **Any developer** building on top of proof infrastructure

## Current Status

| Component | Status |
|-----------|--------|
| Project scaffold | ✅ Ready |
| Classical crypto (Ed25519, AES, HMAC) | 🔨 Phase 1 |
| PQ crypto (simulated ML-DSA, ML-KEM, SPHINCS+) | 🔨 Phase 1 |
| SIWE authentication | 🔨 Phase 1 |
| Dual-signed JWT | 🔨 Phase 1 |
| Double HMAC + anti-replay | 🔨 Phase 1 |
| Merkle proof engine | 🔨 Phase 1 |
| Encrypted Supabase storage | 🔨 Phase 1 |
| Production PQ (liboqs) | 📋 Phase 3 |
| K-PPC on-chain anchoring | 📋 Phase 4 |
| ZK proof circuits | 📋 Phase 5 |
