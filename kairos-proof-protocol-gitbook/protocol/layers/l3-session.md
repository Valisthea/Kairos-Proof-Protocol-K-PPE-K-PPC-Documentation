# L3 — Session Layer (Dual-Signed JWT)

## Purpose
Issue session tokens that are signed by two independent algorithms. A future quantum attack on Ed25519 does not compromise active sessions.

## JWT Structure (KPP-HYBRID-1)

```json
{
  "header": {
    "alg": "KPP-HYBRID-1",
    "typ": "JWT",
    "kid": "kpe-2026-02",
    "pq_status": "SIMULATED"
  },
  "payload": {
    "sub": "0x742d35Cc6634C0532925a3b844Bc9e7595f2bD18",
    "iss": "kairos-proof-engine",
    "iat": 1708700000,
    "exp": 1708703600,
    "nonce": "a3f8b2c1d4e5",
    "pq_level": 3,
    "data_scope": ["realtime_dapp", "onchain_proofs"],
    "key_version": 1
  },
  "signatures": {
    "classical": "<Ed25519_base64url>",
    "post_quantum": "<ML-DSA-65_base64url>"
  }
}
```

## Token Format

```
base64url(header).base64url(payload).base64url(ed25519_sig).base64url(mldsa_sig)
```

Four parts separated by dots (vs. standard JWT's three). The signed message for both algorithms is the same: `base64url(header).base64url(payload)`.

## Verification Rules

**Strict Mode (default):**
- Both Ed25519 AND ML-DSA-65 signatures must be valid
- Token must not be expired
- Nonce must be unique (not replayed)
- Key version must match active signing keys

**Degraded Mode (future feature flag):**
- Only ML-DSA-65 signature required
- Activated if Ed25519 is compromised by quantum attack
- Controlled via server configuration, not client request

## Token Lifetimes

| Token Type | Expiry | Refresh |
|-----------|--------|---------|
| Access Token | 1 hour | Via refresh token |
| Refresh Token | 7 days | Via new SIWE signature |

## Supabase Integration

K-PPE JWT is verified at the Railway layer. After verification, Railway issues a standard Supabase JWT for database access via RLS. This avoids modifying Supabase's auth core.

```
Client → Railway: KPP-HYBRID-1 JWT
Railway: Verify dual signatures ✓
Railway → Supabase: Standard JWT (for RLS)
Supabase: Applies Row Level Security as normal
```
