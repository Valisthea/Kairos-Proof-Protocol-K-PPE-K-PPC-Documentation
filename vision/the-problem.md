# The Problem We Solve

## The Web3 Trust Paradox

Web3 was built on a simple promise: **don't trust, verify**. Yet the entire ecosystem relies on cryptographic primitives that have an expiration date.

Every signature, every proof, every encrypted message in blockchain today uses one of these algorithms:

| Algorithm | Used For | Quantum Vulnerable? |
|-----------|----------|-------------------|
| ECDSA (secp256k1) | Wallet signatures, transaction signing | ✅ **Broken by Shor's algorithm** |
| RSA | TLS certificates, API auth | ✅ **Broken by Shor's algorithm** |
| ECDH (X25519) | Key exchange, TLS handshake | ✅ **Broken by Shor's algorithm** |
| SHA-256 | Hashing, Merkle trees | ⚠️ Weakened by Grover (256→128 bit) |
| AES-256 | Data encryption | ⚠️ Weakened by Grover (256→128 bit) |

When a sufficiently powerful quantum computer arrives — estimated between 2030 and 2040 — **every wallet signature ever made becomes forgeable**. Every encrypted communication becomes readable. Every proof becomes unverifiable.

## The "Harvest Now, Decrypt Later" Threat

This isn't a future problem. It's a **present** problem.

Sophisticated adversaries (nation-states, intelligence agencies, well-funded criminal organizations) are already executing **HNDL attacks**: capturing and storing encrypted data today, waiting for quantum computers to decrypt it later.

For analytics platforms that handle:
- Trading data (competitive intelligence)
- Liquidation events (market manipulation signals)
- Wallet activity patterns (de-anonymization)
- Protocol metrics (insider trading opportunities)

...the data captured today has enormous value if decrypted in 5-10 years.

## The Analytics Platform Gap

The current analytics ecosystem has a critical security gap:

| Platform | Authentication | Data Encryption | Proof Integrity | PQ Protection |
|----------|---------------|-----------------|-----------------|---------------|
| Dune Analytics | OAuth2 | At-rest (provider) | None | ❌ |
| DefiLlama | API Key | At-rest (provider) | None | ❌ |
| Nansen | JWT | At-rest (provider) | None | ❌ |
| Arkham Intelligence | JWT + 2FA | At-rest (provider) | None | ❌ |
| Token Terminal | API Key | At-rest (provider) | None | ❌ |
| **Kairos Analytics** | **SIWE + PQ Hybrid** | **App-level + PQ** | **Merkle + SPHINCS+** | **✅ Native** |

No major analytics platform offers:
1. **Application-level encryption** (they trust their cloud provider entirely)
2. **Verifiable proofs** of on-chain events (you trust their data pipeline)
3. **Post-quantum protection** of any kind

Kairos Proof Protocol fills all three gaps simultaneously.

## Three Problems, One Protocol

### Problem 1: Proofs That Expire
When you query Dune or Nansen for "the volume on Uniswap yesterday," you're trusting their data pipeline. There's no cryptographic proof that the data is accurate. And even if there were, the signatures would be quantum-vulnerable.

**KPP Solution**: Every data point is backed by a Merkle proof, dual-signed with classical (Ed25519) and post-quantum (SPHINCS+) signatures. Verifiable by anyone, forever.

### Problem 2: Data You Can't Protect
Analytics platforms encrypt data at the infrastructure level (AWS, GCP disk encryption). If the provider is compromised, everything is readable. There's no application-level encryption, and certainly no protection against future quantum decryption.

**KPP Solution**: Zero-knowledge architecture where the database (Supabase) only stores ciphertext. Double encryption with independent algorithms. Quantum-resistant key exchange.

### Problem 3: Authentication That Will Break
Every Web3 login (SIWE, wallet connect, API keys) relies on ECDSA or HMAC-SHA256 — both quantum-vulnerable. A future attacker could forge authentication tokens and access historical data.

**KPP Solution**: Hybrid authentication combining classical (ECDSA/Ed25519) with post-quantum (ML-DSA-65) signatures. Both must validate. If one breaks, the other holds.
