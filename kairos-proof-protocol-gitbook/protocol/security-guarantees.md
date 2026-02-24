# Security Guarantees

## What KPP Guarantees vs. Standard Web3

| Property | KPP Guarantee | Standard Web3 |
|----------|--------------|---------------|
| **Confidentiality (classical)** | AES-256-GCM + TLS 1.3 | ≈ Equivalent |
| **Confidentiality (quantum)** | ML-KEM-768 + XChaCha20 | None |
| **Session integrity** | Dual-sign JWT (Ed25519 + ML-DSA) | Single JWT |
| **Non-repudiation** | ECDSA + ML-DSA + SPHINCS+ | ECDSA only |
| **Forward secrecy** | X25519 + ML-KEM (24h rotation) | X25519 only |
| **HNDL resistance** | Double encryption on proofs | None |
| **Proof validity** | 20+ years (SPHINCS+ hash-based) | None |
| **Anti-replay** | Nonce + timestamp + Redis cache | Variable |
| **Zero-knowledge storage** | App-level encryption, DB sees ciphertext | Provider-level |
| **Public verifiability** | Open endpoint, no auth required | None |

## The Core Guarantee

**If either classical OR post-quantum cryptography breaks (but not both simultaneously), the system remains secure.**

This is not a theoretical claim. It's architecturally enforced:
- Every signature requires two independent algorithms to validate
- Every key exchange combines two independent mechanisms
- Every encryption uses independent key derivation paths

The probability of both classical AND post-quantum cryptography breaking simultaneously is negligible — they rely on fundamentally different mathematical problems.

## What KPP Does NOT Guarantee

Transparency about limitations is a security feature:

1. **Client-side security**: If a user's wallet private key is compromised, authentication can be forged. K-PPE protects server-side; client-side security is the user's responsibility.

2. **Real-time key compromise**: If an attacker gains real-time access to Railway's memory, active session keys are exposed. Mitigated in future phases with HSM integration.

3. **Data availability**: If both Supabase and Railway are down, the service is unavailable. Mitigated by K-PPC (decentralized proof storage).

4. **PQ production strength (Phase 1)**: Post-quantum algorithms are currently simulated. They provide the correct interfaces and key sizes but use classical crypto internally. Full PQ security arrives in Phase 3 with liboqs integration.
