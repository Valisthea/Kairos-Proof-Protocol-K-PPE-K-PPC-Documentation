  },
  "batch": {
    "id": "batch-2026-02-24-1200",
    "total_events": 847,
    "timestamp": 1708776000
  }
}
```

Anyone can verify. No account needed. No API key needed. This is the core value proposition of Kairos Analytics.

---

# L7 — Key Rotation & Lifecycle

## Purpose
Automate cryptographic key rotation to minimize the impact of key compromise. Different keys rotate on different schedules based on their risk exposure.

## Rotation Schedule

| Key Type | Rotation | Grace Period | Method |
|----------|----------|-------------|--------|
| TLS certificates | 90 days | 7 days | ACME (Let's Encrypt) |
| ML-KEM encapsulation keys | 24 hours | 1 hour | Re-encapsulation |
| JWT signing (Ed25519) | 30 days | 48 hours | Dual-valid period |
| JWT signing (ML-DSA) | 30 days | 48 hours | Synced with Ed25519 |
| API HMAC secrets | 90 days | 7 days | Encrypted distribution |
| API KMAC secrets | 90 days | 7 days | Synced with HMAC |
| Data encryption DEKs | 7 days | N/A | New DEK per record anyway |
| User PQ keys (ML-DSA) | 12 months | 30 days | Re-sign with wallet |
| Proof signing (Ed25519) | 6 months | 30 days | Dual-signature transition |
| Proof signing (SPHINCS+) | 24 months | 60 days | Longest cycle (stability) |

## Grace Period Behavior

During a grace period, both the old and new keys are valid:

```
Day 0:    Generate new key pair
Day 0-2:  Both old and new keys accepted for JWT verification
Day 2:    Old key deactivated, new key is sole active key
```

This prevents service disruption for clients with cached tokens.

## Key Storage

```
Active keys:     Railway environment variables (in-memory at runtime)
Key metadata:    Supabase kpe_signing_keys table (encrypted)
Key backups:     Encrypted with master key, stored separately
Old keys:        Retained encrypted for 2x rotation period (audit trail)
```

## Forward Secrecy

ML-KEM encapsulation keys rotate every 24 hours. This means:
- Traffic captured today cannot be decrypted with tomorrow's keys
- Even if a key is compromised, only 24 hours of traffic is exposed
- Combined with TLS 1.3's ephemeral key exchange, forward secrecy is maximized

## Automated Rotation Process

```
1. Cron job checks key ages every hour
2. If key age > (rotation_period - grace_period):
   a. Generate new key pair
   b. Encrypt private key with master KEK
   c. Store in kpe_signing_keys (active = true)
   d. Mark old key as grace period
3. If key age > rotation_period:
   a. Mark old key as inactive
   b. Log rotation event to audit trail
4. If key age > 2x rotation_period:
   a. Archive key (keep encrypted, mark archived)
```
