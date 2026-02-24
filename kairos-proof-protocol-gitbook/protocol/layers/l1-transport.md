# L1 — Hybrid Transport Layer

## Purpose
Encrypt all data in transit between clients and the K-PPE server using a combination of classical and post-quantum key exchange.

## How It Works

Standard TLS 1.3 uses X25519 (Elliptic Curve Diffie-Hellman) for key exchange. This is quantum-vulnerable. KPP modifies the handshake to include ML-KEM-768 alongside X25519.

```
Client                                          Server (Railway)
  │                                                │
  │─── ClientHello ────────────────────────────────►│
  │    + X25519 public key                         │
  │    + ML-KEM-768 encapsulation key              │
  │                                                │
  │◄── ServerHello ────────────────────────────────│
  │    + X25519 public key                         │
  │    + ML-KEM-768 ciphertext                     │
  │                                                │
  │  shared_secret = HKDF(X25519_ss ║ ML-KEM_ss)  │
  │  session_keys = HKDF-Expand(shared_secret)     │
  │                                                │
  │◄═══════ Encrypted Channel (AES-256-GCM) ══════►│
```

## Key Details

| Parameter | Value |
|-----------|-------|
| Classical KE | X25519 (Curve25519) |
| Post-Quantum KE | ML-KEM-768 (NIST Level 3) |
| Key Derivation | HKDF-SHA384 |
| Symmetric Cipher | AES-256-GCM |
| Combined Secret | 64 bytes (32 X25519 + 32 ML-KEM) |

## Fallback Behavior

If a client doesn't support ML-KEM (older browsers, limited environments):
- Handshake falls back to X25519-only TLS 1.3
- Server adds header: `X-KPP-PQ-Downgrade: true`
- Sensitive data (proofs) gets additional L5 encryption as compensation
- Client is warned via API response header

## Implementation

Server-side: `oqs-provider` for OpenSSL 3.x on Railway
Client-side: `liboqs` compiled to WASM, loaded asynchronously
