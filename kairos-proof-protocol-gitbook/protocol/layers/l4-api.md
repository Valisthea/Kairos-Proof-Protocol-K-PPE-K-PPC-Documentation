# L4 — API Security Layer

## Purpose
Protect every API request with double authentication (HMAC-SHA256 + KMAC-256) and prevent replay attacks via nonce + timestamp validation.

## Request Headers

Every authenticated API request must include:

```http
POST /proofs/batch HTTP/1.1
Authorization: Bearer <KPP-HYBRID-1 JWT>
X-KPE-Timestamp: 1708700000
X-KPE-Nonce: a3f8b2c1d4e56789abcd1234
X-KPE-Sig-Classic: 9f86d081884c7d659a2feaa0c55ad015...
X-KPE-Sig-PQ: 3c7a2b9e1d4f8a6c5b0e7d2f9a1b3c5d...
Content-Type: application/json
```

## Signature Construction

```
message = timestamp + "|" + nonce + "|" + SHA-256(request_body)

X-KPE-Sig-Classic = HMAC-SHA256(api_secret_classic, message)
X-KPE-Sig-PQ      = KMAC-256(api_secret_pq, message, "kpe-api-v1")
```

Both signatures must be valid. If either fails, the request is rejected with a 401 response indicating which component failed.

## Anti-Replay Protection

```
1. Extract nonce + timestamp from headers
2. Check: |current_time - timestamp| ≤ 30 seconds?
   → No: Reject (clock drift too high)
3. Check: nonce exists in store?
   → Yes: Reject 409 "Replay detected"
   → No: Add nonce to store with 5-minute TTL
4. Proceed to HMAC verification
```

Nonce store: In-memory Map for development, Redis on Railway for production.

## Rate Limiting

| Scope | Limit | Window |
|-------|-------|--------|
| Per wallet address | 100 requests | 1 minute |
| Per wallet address | 1,000 requests | 1 hour |
| Global | 10,000 requests | 1 minute |

Exceeded: HTTP 429 with `Retry-After` header.

## Processing Order

```
Rate Limit (cheapest) → Anti-Replay → HMAC Verify → JWT Verify (most expensive)
```

Cheap checks run first to minimize compute waste on malicious requests.
