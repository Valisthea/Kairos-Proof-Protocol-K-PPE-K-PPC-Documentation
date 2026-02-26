# Glossary

| Term | Definition |
|------|-----------|
| **AES-256-GCM** | Advanced Encryption Standard with 256-bit key in Galois/Counter Mode. Symmetric encryption algorithm used for data at rest. |
| **Arweave** | Permanent decentralized storage network. Pay once, store forever. |
| **anchorBatch()** | KPPEAnchor contract method — anchors a Merkle root on-chain. Params: `appId`, `merkleRoot`, `eventCount`. |
| **Asterchain** | Layer-1 blockchain — planned deployment target for K-PPC (Kairos Proof Protocol Chain). |
| **Base Mainnet** | Layer-2 blockchain (Coinbase, Chain 8453) where K-PPE Merkle roots are anchored via KPPEAnchor contract. |
| **BNB Chain** | Blockchain network (formerly Binance Smart Chain). No longer the primary L1 for K-PPE (replaced by Base Mainnet). |
| **DEK** | Data Encryption Key. Random key used to encrypt a single data record. |
| **Dual-Sign** | Signing data with two independent algorithms (e.g., Ed25519 + ML-DSA). Both must validate. |
| **ECDSA** | Elliptic Curve Digital Signature Algorithm. Used by Bitcoin and Ethereum for transaction signing. Quantum-vulnerable. |
| **Ed25519** | Edwards-curve Digital Signature Algorithm. Fast, constant-time signatures. Used in K-PPE for classical signing. |
| **EIP-4361** | Ethereum Improvement Proposal defining Sign-In with Ethereum (SIWE). |
| **Envelope Encryption** | Pattern where data is encrypted with a DEK, and the DEK is encrypted with a KEK. Enables key rotation without re-encrypting data. |
| **HKDF** | HMAC-based Key Derivation Function. Derives multiple keys from a single master key. |
| **HMAC** | Hash-based Message Authentication Code. Provides both authentication and integrity. |
| **HNDL** | Harvest Now, Decrypt Later. Attack where adversaries capture encrypted data today to decrypt with future quantum computers. |
| **HSM** | Hardware Security Module. Physical device that safeguards cryptographic keys. |
| **Hybrid Cryptography** | Using both classical and post-quantum algorithms simultaneously. Security holds if either one is unbroken. |
| **K-PPC** | Kairos Proof Protocol Chain. Dedicated blockchain for permanent proof storage. |
| **K-PPE** | Kairos Proof Protocol Engine. TypeScript engine implementing the protocol. |
| **KA-HAP** | Kairos Analytics Hybrid Authentication Protocol. The 7-layer security specification. |
| **Keccak-256** | SHA-3 hash function variant used natively in Ethereum/EVM. |
| **KEK** | Key Encryption Key. Used to encrypt DEKs in the envelope encryption pattern. |
| **KMAC-256** | Keccak-based MAC. Message authentication code based on SHA-3. Quantum-resistant by design. |
| **KPP** | Kairos Proof Protocol. The overall cryptographic specification. |
| **KPPEAnchor** | Smart contract deployed on Base Mainnet (`0x3B53F7044E47766769156bF210c2661F03Df45dd`) that stores Merkle roots on-chain. Core of the K-PPE trustless model. |
| **KPPE_MODE** | Environment variable controlling the relayer's anchoring mode: `kppe` (production, root-only on-chain), `legacy` (deprecated, full data on-chain), `disabled` (DB only). |
| **Merkle Tree** | Hash tree structure where each leaf is a hash of data and each node is the hash of its children. Enables efficient proof of inclusion. |
| **MiCA** | Markets in Crypto-Assets Regulation. EU regulatory framework for crypto. |
| **ML-DSA-65** | Module-Lattice Digital Signature Algorithm (NIST FIPS 204). Post-quantum replacement for ECDSA/EdDSA. |
| **ML-KEM-768** | Module-Lattice Key Encapsulation Mechanism (NIST FIPS 203). Post-quantum replacement for ECDH/RSA key exchange. |
| **MLWE** | Module Learning With Errors. Mathematical problem underlying ML-KEM and ML-DSA security. |
| **Post-Quantum** | Cryptographic algorithms designed to be secure against quantum computer attacks. |
| **prevRoot** | Previous batch's Merkle root, stored on-chain in each new batch. Creates an immutable chain of trust linking all batches chronologically. |
| **RLS** | Row Level Security. Supabase/PostgreSQL feature restricting data access at the row level. |
| **Session DNA** | Unique identifier combining wallet, user agent, and timestamp to create a session fingerprint. |
| **Sorted-Pair Hashing** | Merkle hashing method: `keccak256(abi.encodePacked(min(a,b), max(a,b)))`. Order-independent — same result regardless of sibling position. Used by KPPEAnchor. |
| **Shor's Algorithm** | Quantum algorithm that efficiently factors large numbers and computes discrete logarithms. Breaks RSA, ECDSA, ECDH. |
| **SIWE** | Sign-In with Ethereum. Standard for wallet-based authentication (EIP-4361). |
| **SPHINCS+** | Stateless Hash-Based Digital Signature. Post-quantum algorithm based solely on hash functions. Zero mathematical assumptions. Used for long-term proof signatures. |
| **verifyProofView()** | KPPEAnchor contract read method — verifies a Merkle proof on-chain. Params: `batchId`, `eventHash`, `proof[]`, `leafIndex`. Returns `bool`. |
| **XChaCha20-Poly1305** | Symmetric encryption algorithm. Alternative to AES-GCM with larger nonce (192-bit). Used as second layer in double encryption. |
| **Zero-Knowledge Storage** | Architecture where the database stores only ciphertext and has no access to plaintext data or encryption keys. |
| **ZK Proof** | Zero-Knowledge Proof. Cryptographic method to prove a statement is true without revealing the underlying data. |
| **ZK Rollup** | Layer 2 scaling solution that posts transaction data on-chain and uses ZK proofs for validity. |

---

# FAQ

## General

**Q: Is Kairos Proof Protocol only for Kairos Analytics / Kairos Lab?**
A: No. K-PPE is infrastructure designed for any dApp, protocol, exchange, or platform that needs verifiable on-chain proofs. Kairos Analytics is the first consumer, but the protocol is open to all.

**Q: Is the code open source?**
A: The code is publicly viewable on GitHub for transparency and auditability. However, all rights are reserved — it is not licensed for use, modification, or distribution by third parties.

**Q: Why not fully open source?**
A: We believe in transparency (public code) but also in protecting the protocol's development. This may change in the future as the ecosystem matures. Uniswap v3 followed a similar approach (Business Source License).

**Q: Do I need to understand quantum computing to use K-PPE?**
A: No. The post-quantum protection is built into the protocol. You integrate via a standard REST API or SDK — the hybrid crypto is handled automatically.

## Technical

**Q: Are the post-quantum algorithms really production-ready?**
A: In Phase 1, PQ algorithms are simulated (correct interfaces, correct key sizes, classical crypto internally). Production liboqs integration comes in Phase 3. The simulated mode is clearly labeled everywhere (`PQ_STATUS: SIMULATED`).

**Q: Why simulate PQ instead of waiting?**
A: Building the interfaces now means the entire codebase is PQ-ready. When liboqs is production-stable for Node.js, it's a drop-in replacement — no refactoring needed. The classical crypto layer provides full security in the meantime.

**Q: What happens if both classical AND post-quantum crypto are broken?**
A: This would require simultaneously breaking elliptic curve cryptography AND lattice-based cryptography (or hash-based, for SPHINCS+). These rely on fundamentally different mathematical problems. The probability of simultaneous breakthrough is negligible.

**Q: Why use SPHINCS+ for proofs instead of ML-DSA?**
A: SPHINCS+ is based solely on hash function security — zero mathematical assumptions. For proofs that must remain valid for 20+ years, this is the most conservative choice. ML-DSA relies on the MLWE problem, which is newer and less studied long-term.

**Q: Can I verify proofs without an account?**
A: Yes. The `/proofs/verify/:batchId/:eventHash` endpoint requires no authentication. Anyone can verify any proof, anytime.

## Business

**Q: How much does K-PPE cost?**
A: Pricing will be usage-based (per proof batch). Details in Phase 2 when multi-client support launches. For early integrators, contact us directly.

**Q: What chains are supported?**
A: K-PPE is deployed on **Base Mainnet** (Chain 8453). K-PPC is planned for **Asterchain L1 Mainnet**. Phase 6: Ethereum, Arbitrum, Polygon, Solana indexers.

**Q: What's K-PPC and when does it launch?**
A: K-PPC (Kairos Proof Protocol Chain) is a dedicated blockchain for permanent proof storage, planned for **Asterchain L1 Mainnet**. Research phase now, target launch Q3 2027+.

**Q: What is KPPE_MODE?**
A: The `KPPE_MODE` environment variable controls how the relayer anchors batches: `kppe` (production — only Merkle root on-chain, data stays private), `legacy` (deprecated — full event data on-chain), `disabled` (database only).

**Q: Can I verify proofs on-chain?**
A: Yes. Call `KPPEAnchor.verifyProofView(batchId, eventHash, proof[], leafIndex)` on Base Mainnet. Fully trustless, no auth required, callable from any chain via RPC.

---

# Cryptographic References

## NIST Standards
- [FIPS 203: ML-KEM](https://csrc.nist.gov/pubs/fips/203/final) — Module-Lattice Key Encapsulation
- [FIPS 204: ML-DSA](https://csrc.nist.gov/pubs/fips/204/final) — Module-Lattice Digital Signature
- [FIPS 205: SLH-DSA (SPHINCS+)](https://csrc.nist.gov/pubs/fips/205/final) — Stateless Hash-Based Signature

## Classical Cryptography
- [RFC 8032: Ed25519](https://datatracker.ietf.org/doc/html/rfc8032) — Edwards-Curve Digital Signature Algorithm
- [RFC 5869: HKDF](https://datatracker.ietf.org/doc/html/rfc5869) — HMAC-based Extract-and-Expand Key Derivation
- [NIST SP 800-38D: AES-GCM](https://csrc.nist.gov/pubs/sp/800/38/d/final) — Galois/Counter Mode

## Web3 Standards
- [EIP-4361: SIWE](https://eips.ethereum.org/EIPS/eip-4361) — Sign-In with Ethereum
- [EIP-55: Mixed-case checksum](https://eips.ethereum.org/EIPS/eip-55) — Address encoding

## Libraries Used
- [Open Quantum Safe (liboqs)](https://openquantumsafe.org/) — PQ crypto library (Phase 3)
- [@noble/ed25519](https://github.com/paulmillr/noble-ed25519) — Audited Ed25519 implementation
- [@noble/hashes](https://github.com/paulmillr/noble-hashes) — Audited hash functions
- [libsodium](https://doc.libsodium.org/) — XChaCha20-Poly1305, key exchange
- [merkletreejs](https://github.com/merkletreejs/merkletreejs) — Merkle tree construction

---

# Brand & Legal

## Brand Names

| Name | Usage |
|------|-------|
| **Kairos Proof Protocol (KPP)** | The overall cryptographic specification |
| **K-PPE** | Kairos Proof Protocol Engine (the software) |
| **K-PPC** | Kairos Proof Protocol Chain (the blockchain) |
| **Kairos Analytics** | The analytics platform product |
| **Kairos Lab** | The competitive trading platform (separate product) |
| **KA-HAP** | Kairos Analytics Hybrid Authentication Protocol (the 7-layer spec) |

## Legal Notice

**All Rights Reserved.**

The source code in the `kairos-proof-protocol-engine` repository is publicly viewable for transparency and auditability purposes. No license is granted for use, modification, distribution, or creation of derivative works.

Viewing, studying, and auditing the code for security research purposes is permitted and encouraged. Any other use requires explicit written permission.

## Trademark

"Kairos," "Kairos Analytics," "Kairos Lab," "Kairos Proof Protocol," "K-PPE," and "K-PPC" are trademarks. Use in reference to this project is permitted; use implying endorsement or affiliation is not.

## Contact

For integration inquiries, security disclosures, or partnership proposals, contact the Kairos team.

## Responsible Disclosure

If you discover a security vulnerability in K-PPE, please report it responsibly. Do not open a public GitHub issue. Contact us directly. We will acknowledge receipt within 24 hours and provide a timeline for resolution.
