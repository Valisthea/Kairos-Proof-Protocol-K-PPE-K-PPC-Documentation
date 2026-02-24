# What is Kairos Analytics?

Kairos Analytics is a real-time dApp analytics platform built on top of the Kairos Proof Protocol (K-PPE). It is the first application to demonstrate the full capabilities of K-PPE in production.

## Core Concept

Traditional analytics platforms (Dune, Nansen, Arkham) collect on-chain data and display it on dashboards. You trust their data pipeline, their infrastructure, their security.

Kairos Analytics does the same — but with a fundamental difference: **every data point is backed by a cryptographic proof that anyone can verify independently.**

```
Traditional Analytics              Kairos Analytics
════════════════════               ════════════════

Blockchain → Indexer → DB → UI     Blockchain → Indexer → K-PPE → DB → UI
                                                   │
You trust the platform             K-PPE generates:
to show correct data               • Merkle proof per event
                                   • Dual signatures (classical + PQ)
                                   • Encrypted storage
                                   • Public verification endpoint
                                   
                                   You VERIFY the data, not trust it
```

## Key Features

### 1. Real-Time dApp Monitoring
Live feeds of on-chain events across supported chains: trades, liquidations, oracle updates, governance actions, token transfers, contract deployments.

### 2. Verifiable Data Points
Every metric displayed on the dashboard (volume, TVL, active users, etc.) links back to Merkle proofs that anyone can verify via the public API.

### 3. Post-Quantum Security
All data is protected with hybrid classical + post-quantum encryption. Authentication uses dual-signed JWTs. Proofs are signed with SPHINCS+ for 20+ year validity.

### 4. Zero-Knowledge Storage
The database (Supabase) stores only ciphertext. A database compromise yields nothing readable. Encryption and decryption happen at the application layer (Railway).

### 5. Public Proof Verification
The `/proofs/verify` endpoint requires no authentication. Anyone — auditors, regulators, users, competitors — can verify any proof at any time.

### 6. API Access
Full API for programmatic access to analytics data and proof verification. SDK available for JavaScript/TypeScript integration.
