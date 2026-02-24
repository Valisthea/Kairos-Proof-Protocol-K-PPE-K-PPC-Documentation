# The 7 Security Layers

The KA-HAP protocol organizes security into 7 independent layers. Each layer combines classical and post-quantum cryptography. Each layer can be upgraded independently without affecting others.

```
REQUEST FLOW
════════════

Client (Browser/dApp)
    │
    ▼
L1 — TRANSPORT ─────── TLS 1.3 + ML-KEM-768 hybrid handshake
    │
    ▼
L2 — IDENTITY ──────── SIWE wallet auth + ML-DSA-65 backup key
    │
    ▼
L3 — SESSION ───────── Dual-signed JWT (Ed25519 + ML-DSA-65)
    │
    ▼
L4 — API ───────────── Double HMAC (SHA-256 + KMAC-256) + anti-replay
    │
    ▼
L5 — DATA ──────────── Envelope encryption (AES-256 + XChaCha20)
    │
    ▼
L6 — PROOF ─────────── Merkle trees + Ed25519 + SPHINCS+ signatures
    │
    ▼
L7 — ROTATION ──────── Automated key lifecycle (24h to 24 months)
```

Each layer is detailed in its own page. Click through for implementation details, code examples, and security rationale.

| Layer | Primary Function | Classical | Post-Quantum | Failure Impact |
|-------|-----------------|-----------|-------------|---------------|
| L1 | Channel encryption | X25519 ECDH | ML-KEM-768 | Data exposed in transit |
| L2 | User identity | ECDSA (wallet) | ML-DSA-65 backup | Identity forgery |
| L3 | Session tokens | Ed25519 JWT | ML-DSA-65 JWT | Session hijacking |
| L4 | Request auth | HMAC-SHA256 | KMAC-256 | API abuse |
| L5 | Data at rest | AES-256-GCM | XChaCha20-Poly1305 | Data breach |
| L6 | Proof integrity | Ed25519 + Merkle | SPHINCS+ | Proof forgery |
| L7 | Key freshness | HKDF rotation | ML-KEM re-encap | Long-term compromise |
