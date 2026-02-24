# L2 — Identity Layer (Wallet + PQ Backup)

## Purpose
Authenticate users via their Web3 wallet (SIWE/EIP-4361) and generate a post-quantum backup identity for future-proof authentication.

## How It Works

### First Login (Registration)
```
1. User connects wallet (MetaMask, WalletConnect, etc.)
2. Server generates SIWE message + cryptographic nonce
3. User signs with ECDSA (secp256k1) via wallet
4. Server verifies ECDSA signature
5. Server generates ML-DSA-65 key pair for the user
6. ML-DSA private key encrypted with key derived from wallet signature
7. ML-DSA public key stored in database (Supabase: kpe_users table)
```

### Subsequent Logins
```
1. User connects wallet
2. Server generates SIWE message + nonce
3. User signs with wallet (ECDSA)
4. Server verifies ECDSA + loads user's ML-DSA public key
5. Server issues dual-signed JWT (→ L3)
```

## SIWE Message Format (EIP-4361)

```
kairos-analytics.io wants you to sign in with your Ethereum account:
0x742d35Cc6634C0532925a3b844Bc9e7595f2bD18

Sign in to Kairos Analytics with hybrid post-quantum security

URI: https://kairos-analytics.io
Version: 1
Chain ID: 56
Nonce: a3f8b2c1d4e56789
Issued At: 2026-02-24T12:00:00Z
Expiration Time: 2026-02-24T12:05:00Z
Resources:
- https://kairos-analytics.io/tos
```

## PQ Key Storage

The user's ML-DSA-65 private key is never stored in plaintext:

```
wallet_signature = wallet.sign(SIWE_message)
salt = random(32 bytes)
kek = HKDF-SHA256(wallet_signature ║ server_salt, salt, "kpe-user-kek-v1", 32)
encrypted_sk = AES-256-GCM(kek, ml_dsa_private_key)
→ Store: encrypted_sk + salt in Supabase
```

To recover the PQ key, the user must re-sign with their wallet. This means:
- Supabase compromise → attacker gets encrypted blob, useless without wallet signature
- Server compromise → attacker needs wallet to derive KEK
- Only the wallet holder can unlock their PQ identity

## Database Schema

```sql
CREATE TABLE kpe_users (
  id               UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  wallet_address   TEXT NOT NULL UNIQUE,
  pq_public_key    BYTEA NOT NULL,          -- ML-DSA-65 public key (1952 bytes)
  pq_encrypted_sk  BYTEA NOT NULL,          -- Encrypted ML-DSA-65 private key
  pq_salt          BYTEA NOT NULL,          -- Salt for KEK derivation
  key_version      INTEGER DEFAULT 1,
  created_at       TIMESTAMPTZ DEFAULT now(),
  last_auth_at     TIMESTAMPTZ,
  rotated_at       TIMESTAMPTZ
);
```

## Supported Wallets

Any EIP-4361 compatible wallet:
- MetaMask
- WalletConnect v2
- Coinbase Wallet
- Rainbow
- Rabby
- Safe (multi-sig — with threshold signatures)
