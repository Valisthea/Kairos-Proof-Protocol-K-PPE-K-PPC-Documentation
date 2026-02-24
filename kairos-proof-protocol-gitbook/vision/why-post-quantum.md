# Why Post-Quantum Matters Now

## The Timeline

The common misconception: "Quantum computers are decades away, we have time."

The reality:

| Year | Milestone |
|------|-----------|
| 2019 | Google achieves quantum supremacy (53 qubits) |
| 2022 | IBM unveils 433-qubit Osprey processor |
| 2023 | IBM announces 1,121-qubit Condor processor |
| 2024 | NIST finalizes ML-KEM (FIPS 203) and ML-DSA (FIPS 204) standards |
| 2024 | Google's Willow chip: 105 qubits with error correction breakthroughs |
| 2025 | Microsoft announces Majorana 1 topological qubit chip |
| 2025+ | Multiple governments funding billion-dollar quantum programs |
| 2030-2035 | **Estimated: cryptographically relevant quantum computer** |

The number of logical qubits needed to break ECDSA (secp256k1): approximately **2,500**. Current systems have 1,000+ physical qubits. Error correction is the bottleneck — and it's being solved rapidly.

## Why "Later" Is Too Late

### The HNDL Problem

**Harvest Now, Decrypt Later** means the attack is happening today:

```
2026: Adversary captures encrypted analytics data
      (trading patterns, wallet correlations, protocol metrics)
                    │
                    │  stores for later
                    ▼
2032: Quantum computer breaks encryption
      Adversary now has 6 years of trading intelligence
      Adversary can forge 6 years of ECDSA signatures
      Adversary can de-anonymize 6 years of wallet activity
```

For data with long-term value — financial records, legal proofs, audit trails — protection must begin **before** the threat materializes, not after.

### The Migration Problem

Migrating cryptographic infrastructure is extremely slow:

- The transition from SHA-1 to SHA-256 took **over a decade** (deprecated 2011, still found in 2023)
- The TLS 1.0 → 1.3 migration took **7 years** (2011-2018)
- NIST started the PQ standardization process in **2016** and finalized in **2024** (8 years)

Web3 protocols that wait for quantum urgency will face a chaotic, emergency migration while adversaries are already exploiting the vulnerability window.

### The Data Retention Problem

Analytics data has a long shelf life:

- **Regulatory compliance**: MiCA, MiFID II, SEC require years of trade records
- **Legal proceedings**: Proofs of on-chain events may be needed years after the fact
- **Institutional audits**: Fund performance records must be verifiable long-term
- **Historical research**: Protocol governance decisions, DAO votes, market events

Data encrypted today with ECDH/AES and signed with ECDSA will be **unverifiable and readable** within 10-15 years. That's within the retention period of most regulatory frameworks.

## NIST Post-Quantum Standards

In August 2024, NIST finalized the first three post-quantum cryptographic standards:

### FIPS 203: ML-KEM (Module-Lattice Key Encapsulation)
- **Replaces**: ECDH, RSA key exchange
- **Based on**: Module Learning With Errors (MLWE) problem
- **Used in KPP**: L1 Transport Layer (hybrid with X25519)
- **Key sizes**: 1,184 bytes (public), 2,400 bytes (private) for ML-KEM-768

### FIPS 204: ML-DSA (Module-Lattice Digital Signature)
- **Replaces**: ECDSA, EdDSA for general signatures
- **Based on**: Module Learning With Errors + Short Integer Solution
- **Used in KPP**: L2 Identity, L3 Session (dual-signed JWT)
- **Signature size**: 3,309 bytes for ML-DSA-65

### FIPS 205: SLH-DSA (Stateless Hash-Based Digital Signature) / SPHINCS+
- **Replaces**: Nothing — it's a backup, zero mathematical assumptions
- **Based on**: Hash functions only (SHA-256, SHAKE-256)
- **Used in KPP**: L6 Proof Engine (long-term proof signatures)
- **Signature size**: 29,792 bytes for SPHINCS+-256s
- **Why**: Even if lattice-based crypto (ML-DSA) is somehow broken, SPHINCS+ remains secure because it relies only on hash function security

## KPP's Approach: Hybrid, Not Replacement

KPP doesn't replace classical cryptography — it **layers** post-quantum on top:

```
Classical (today)          +    Post-Quantum (future-proof)
═══════════════════              ═════════════════════════
Ed25519 signatures               ML-DSA-65 signatures
X25519 key exchange              ML-KEM-768 encapsulation
HMAC-SHA256 API auth             KMAC-256 API auth
AES-256-GCM encryption           XChaCha20-Poly1305 backup
SHA-256 Merkle trees             SPHINCS+ proof signing
```

**Both must validate.** If quantum computers break Ed25519, ML-DSA-65 still holds. If a flaw is found in ML-DSA-65, Ed25519 still holds. The system is secure as long as **at least one** side is unbroken.

This is defense in depth applied to cryptography.
