# L5 — Data Encryption Layer

## Purpose
Encrypt all sensitive data at the application level before it reaches the database. Supabase stores only ciphertext — a database compromise yields nothing readable.

## Zero-Knowledge Architecture

```
                    DATA FLOW
                    ═════════

Client → Railway (K-PPE) → Supabase

         ┌─────────────────────┐
         │   Railway (K-PPE)    │
         │                     │
         │  plaintext IN       │
         │       ↓             │
         │  envelope_encrypt() │
         │       ↓             │
         │  ciphertext OUT     │
         └─────────┬───────────┘
                   │
                   ▼
         ┌─────────────────────┐
         │   Supabase          │
         │                     │
         │  Stores ONLY:       │
         │  • Encrypted blobs  │
         │  • Encrypted DEKs   │
         │  • IVs and tags     │
         │                     │
         │  NEVER sees:        │
         │  • Plaintext data   │
         │  • Master keys      │
         │  • Derived keys     │
         └─────────────────────┘
```

## Envelope Encryption Pattern

Data is encrypted with a random DEK (Data Encryption Key). The DEK is encrypted with a KEK (Key Encryption Key) derived from the master key. This allows key rotation without re-encrypting all data.

```
plaintext
    │
    ▼
DEK = random(32 bytes)
encrypted_data = AES-256-GCM(DEK, plaintext)
    │
    ▼
KEK = HKDF(master_key, salt, "kpe-dek-v1")
encrypted_DEK = AES-256-GCM(KEK, DEK)
    │
    ▼
Store: { encrypted_data, encrypted_DEK, iv, dek_iv, tag, dek_tag }
```

## Double Encryption (Proofs)

On-chain proofs have long-term value and are the primary HNDL target. They receive double encryption:

```
plaintext proof
    │
    ├─ Layer 1: AES-256-GCM(DEK_1)
    │
    ├─ Layer 2: XChaCha20-Poly1305(DEK_2)
    │
    └─ Both DEKs encrypted with independent KEKs
```

Two different algorithms, two different keys. An attacker must break both to access the proof data.

## Data Classification

| Data Type | Encryption | Double? | Key Rotation |
|-----------|-----------|---------|-------------|
| Analytics metrics (real-time) | AES-256-GCM | No | Weekly DEK rotation |
| On-chain proofs (archived) | AES-256 + XChaCha20 | **Yes** | Weekly DEK rotation |
| User PQ private keys | AES-256-GCM | No | On wallet re-auth |
| Audit logs | AES-256-GCM | No | Weekly DEK rotation |
| Signing keys (encrypted) | AES-256-GCM | No | Per rotation cycle |

## Key Hierarchy

```
Master Key (environment variable, Railway)
    │
    ├─ HKDF → KEK for analytics data
    ├─ HKDF → KEK for proof data (layer 1)
    ├─ HKDF → KEK for proof data (layer 2)
    ├─ HKDF → KEK for user PQ keys
    └─ HKDF → KEK for audit logs

Each KEK encrypts random DEKs.
Each DEK encrypts one data record.
```
