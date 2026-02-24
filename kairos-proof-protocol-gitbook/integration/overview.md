# Integration Overview

K-PPE is designed as **infrastructure**, not a monolithic product. Any application can integrate K-PPE to add verifiable, post-quantum secure proofs to their data.

## Integration Methods

### 1. REST API (Direct)
Call K-PPE endpoints directly from your backend. Best for: custom integrations, backend services, scripts.

### 2. JavaScript/TypeScript SDK
Import the K-PPE client library. Best for: Node.js applications, dApps, web apps.

```typescript
import { KairosProofClient } from '@kairos/proof-sdk';

const client = new KairosProofClient({
  endpoint: 'https://kpe.kairos-analytics.io',
  apiKey: process.env.KPE_API_KEY,
});

// Submit events for proof generation
const batch = await client.proofs.createBatch({
  events: myOnChainEvents,
});

// Verify a proof (no API key needed)
const verification = await client.proofs.verify(
  batch.batchId,
  eventHash
);
```

### 3. Webhook Listeners
K-PPE can push proof completion events to your endpoint. Best for: async workflows, monitoring systems.

### 4. K-PPC Direct (Future)
Read and verify proofs directly from K-PPC chain. Best for: fully decentralized verification without trusting K-PPE servers.

## Quick Comparison

| Method | Auth Required | Real-time | Decentralized | Complexity |
|--------|:------------:|:---------:|:-------------:|:----------:|
| REST API | ✅ | ✅ | ❌ | Low |
| SDK | ✅ | ✅ | ❌ | Low |
| Webhooks | ✅ | ⚠️ | ❌ | Medium |
| K-PPC Direct | ❌ | ⚠️ | ✅ | High |
| Public Verify | ❌ | ✅ | ❌ | Very Low |

---

# For dApps & Protocols

## Why Integrate

Your protocol probably displays metrics: TVL, volume, user count, yield rates. Users trust these numbers because they trust you. With K-PPE, users can **verify** these numbers independently.

## Integration Flow

```
Your Protocol
     │
     │ 1. Capture on-chain events (trades, deposits, votes...)
     │
     ▼
K-PPE SDK
     │
     │ 2. POST /proofs/batch → submit events
     │ 3. Receive proof batch (Merkle root + individual proofs)
     │
     ▼
Your Dashboard
     │
     │ 4. Display metrics with "Verified by Kairos" badge
     │ 5. Link each metric to its public verification URL
     │
     ▼
Your Users
     │
     │ 6. Click "Verify" → GET /proofs/verify/... (no auth)
     │ 7. See cryptographic proof that the data is accurate
```

## Example: Verifiable TVL Display

```typescript
// Your protocol's backend
const tvlEvents = await getDepositWithdrawEvents(last24h);

const batch = await kairos.proofs.createBatch({
  events: tvlEvents,
  metadata: { metric: 'tvl', period: '24h' }
});

// Your frontend
<div class="tvl-card">
  <span>TVL: $50,234,891</span>
  <a href={`https://kpe.kairos-analytics.io/proofs/verify/${batch.batchId}`}>
    🛡️ Verified by Kairos
  </a>
</div>
```

---

# For Exchanges & CEX

## The Problem

Centralized exchanges publish volume numbers that are widely suspected of being inflated. Proof of Reserves became a norm after FTX, but there's no equivalent for trade volume or execution quality.

## What K-PPE Offers

- **Verifiable Volume**: Submit trade events to K-PPE, receive Merkle proofs. Publish proofs for independent verification.
- **Execution Proofs**: Prove that trades were executed at the stated price and time.
- **Audit Trail**: Generate tamper-proof records for regulatory compliance.

## Integration Architecture

```
Exchange Matching Engine
     │
     │ trade events (via internal API)
     ▼
K-PPE Integration Service (your backend)
     │
     │ batch submission every 60 seconds
     ▼
K-PPE → Proof generation + storage
     │
     ▼
Public Verification Endpoint
     │
     ▼
Auditors / Regulators / Users can verify
```

---

# For Institutional Clients

## Audit-Grade Proofs

Institutional investors need:
1. Verifiable track records (not self-reported PnL)
2. Proof of execution (trades happened at stated prices)
3. Long-term data integrity (records valid for 10+ years)
4. Regulatory compliance (MiCA, MiFID II, SEC reporting)

K-PPE provides all four, with post-quantum protection ensuring proofs remain valid for decades.

## Compliance Features

| Requirement | K-PPE Solution |
|-------------|---------------|
| Data retention (5-10 years) | SPHINCS+ proofs valid 20+ years |
| Independent verification | Public endpoint, no auth required |
| Tamper evidence | Merkle trees + dual signatures |
| Data integrity | Hash chain from raw event to proof |
| Audit trail | Every proof batch timestamped and signed |
| Data protection | App-level encryption, zero-knowledge storage |

## How to Present K-PPE Proofs to Auditors

```
1. Generate proofs for all relevant events via K-PPE
2. Provide auditor with batch IDs
3. Auditor independently verifies via public endpoint
4. Auditor receives: event hash, Merkle proof, dual signatures
5. Auditor can mathematically verify without trusting any party
```

No access to your K-PPE account needed. No API key. No trust. Just math.

---

# SDK Reference

## Installation

```bash
npm install @kairos/proof-sdk
```

## Initialization

```typescript
import { KairosProofClient } from '@kairos/proof-sdk';

const client = new KairosProofClient({
  endpoint: 'https://kpe.kairos-analytics.io',
  walletAddress: '0x...',
  chainId: 56,
  // API credentials (for authenticated endpoints)
  hmacSecret: process.env.KPE_HMAC_SECRET,
  kmacSecret: process.env.KPE_KMAC_SECRET,
});
```

## Authentication

```typescript
// Get SIWE challenge
const { message, nonce } = await client.auth.challenge();

// Sign with wallet (browser)
const signature = await wallet.signMessage(message);

// Verify and get JWT
const { token } = await client.auth.verify(message, signature);
// Token is automatically stored and used for subsequent requests
```

## Proof Operations

```typescript
// Submit events for proof generation
const batch = await client.proofs.createBatch({
  events: [
    {
      txHash: '0xabc...',
      blockNumber: 12345678,
      chainId: 56,
      contractAddress: '0xdef...',
      eventName: 'Trade',
      args: { pair: 'BTC/USDT', price: 65000, size: 0.5 },
      timestamp: Date.now(),
      rawLog: '0x...',
    }
  ]
});

// Get batch details
const batchData = await client.proofs.getBatch(batch.batchId);

// Verify a proof (no auth required)
const result = await client.proofs.verify(batch.batchId, eventHash);
console.log(result.valid); // true
console.log(result.signatures.classical.valid); // true
console.log(result.signatures.postQuantum.valid); // true
```

## Webhook Registration (Future)

```typescript
await client.webhooks.register({
  url: 'https://your-app.com/webhook/kpe',
  events: ['proof.batch.created', 'proof.batch.anchored'],
  secret: 'your-webhook-secret',
});
```
