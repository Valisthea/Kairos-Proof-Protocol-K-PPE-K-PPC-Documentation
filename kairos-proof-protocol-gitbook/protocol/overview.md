# Protocol Overview

## KA-HAP: Kairos Analytics Hybrid Authentication Protocol

The Kairos Proof Protocol (KPP) is built on a 7-layer security architecture called **KA-HAP** (Kairos Analytics Hybrid Authentication Protocol). Each layer addresses a specific threat vector, and every layer combines classical and post-quantum cryptography.

### Design Principles

1. **Hybrid Always**: Every cryptographic operation uses two independent algorithms. Both must succeed. One can break without compromising the system.

2. **Defense in Depth**: Seven independent layers. A breach in one layer does not cascade to others.

3. **Zero Trust Storage**: The database stores only ciphertext. Encryption keys never touch the database.

4. **Public Verifiability**: Proofs can be verified by anyone without authentication. Privacy and verifiability coexist.

5. **Forward-Compatible**: Interfaces are designed for easy algorithm swaps when production-grade PQ libraries mature.

### Architecture Overview

```
┌────────────────────────────────────────────────────────────┐
│  L7 — KEY ROTATION          Automated lifecycle management │
├────────────────────────────────────────────────────────────┤
│  L6 — PROOF ENGINE          Merkle + SPHINCS+ signatures   │
├────────────────────────────────────────────────────────────┤
│  L5 — DATA ENCRYPTION       AES-256 + XChaCha20 (double)  │
├────────────────────────────────────────────────────────────┤
│  L4 — API SECURITY          HMAC-SHA256 + KMAC-256         │
├────────────────────────────────────────────────────────────┤
│  L3 — SESSION               Dual-signed JWT (Ed25519+PQ)   │
├────────────────────────────────────────────────────────────┤
│  L2 — IDENTITY              SIWE + ML-DSA-65 backup        │
├────────────────────────────────────────────────────────────┤
│  L1 — TRANSPORT             TLS 1.3 + ML-KEM-768 hybrid    │
└────────────────────────────────────────────────────────────┘
```

Each layer is documented in detail in the [Seven Security Layers](seven-layers.md) section.

---

# Threat Model

## Adversary Classes

### Class A — Opportunistic Attacker (Today)
- Skills: Script kiddie to intermediate hacker
- Resources: Consumer hardware, public tools
- Goals: Session hijacking, data scraping, API abuse
- **Mitigated by**: L1 (TLS), L3 (JWT), L4 (HMAC + anti-replay)

### Class B — Sophisticated Attacker (Today)
- Skills: Advanced persistent threat, state-affiliated
- Resources: Significant compute, zero-day exploits
- Goals: Infrastructure compromise, data exfiltration, targeted attacks
- **Mitigated by**: L2 (wallet auth), L5 (app-level encryption), L7 (key rotation)

### Class C — Quantum Adversary (2030-2040)
- Skills: Operates a cryptographically relevant quantum computer
- Resources: Nation-state level
- Goals: Break ECDSA/RSA/ECDH, decrypt historical data, forge signatures
- **Mitigated by**: All PQ components across all layers

### Class D — HNDL Adversary (Today, exploits later)
- Skills: Network interception capability
- Resources: State-level passive surveillance
- Goals: Capture encrypted traffic now, decrypt with quantum computer later
- **Mitigated by**: L1 (PQ key exchange), L5 (double encryption), L6 (SPHINCS+ long-term proofs)

## Risk Matrix

| Threat | Adversary | Probability | Impact | KPP Mitigation |
|--------|-----------|-------------|--------|---------------|
| MITM on analytics stream | A, B | High | Critical | L1: TLS 1.3 + ML-KEM hybrid |
| JWT session hijacking | A, B | High | High | L3: Dual-signed JWT, 1h expiry |
| API replay attack | A | Medium | High | L4: Nonce + timestamp + dual HMAC |
| Database compromise | B | Medium | Critical | L5: Zero-knowledge storage |
| Wallet spoofing | A | Medium | High | L2: SIWE + server nonce |
| HNDL on proofs | D | Medium | Critical | L5: Double encryption + L6: SPHINCS+ |
| Shor's on ECDSA | C | Low (2030+) | Critical | L2: ML-DSA backup identity |
| Key compromise | B | Low | Critical | L7: Automated rotation, short lifetimes |
| Infrastructure total loss | B | Low | Critical | L6: On-chain anchoring (K-PPC future) |

## Security Assumptions

1. **Hash functions remain secure**: SHA-256, SHA-3/Keccak-256, SHAKE-256 are not broken by quantum computers (weakened by Grover but AES-256 equivalent remains sufficient)
2. **MLWE problem is hard**: ML-KEM and ML-DSA are based on the Module Learning With Errors problem, believed to be quantum-resistant
3. **At least one algorithm survives**: If classical OR post-quantum crypto breaks (but not both simultaneously), the system remains secure
4. **Client-side wallet is not compromised**: If the user's private wallet key is stolen, authentication can be forged regardless of server-side protection
5. **Railway compute is not compromised in real-time**: Keys exist in memory during computation. A real-time memory dump would expose session keys (mitigated by HSM in future phases)

## Known Limitations

| Limitation | Impact | Mitigation Timeline |
|-----------|--------|-------------------|
| PQ algorithms are simulated (not production liboqs) | Reduced PQ security in Phase 1 | Phase 3: liboqs integration |
| No HSM for key storage | Keys in memory/env variables | Phase 5: HSM integration |
| Single compute provider (Railway) | Single point of failure | K-PPC: Decentralized validation |
| No formal verification of protocol | Potential logic errors | Phase 5: External audit |
| Supabase as storage dependency | Availability risk | K-PPC: Redundant on-chain storage |
