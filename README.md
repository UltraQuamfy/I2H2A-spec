# I2H2A — Issuer → Holder → Agent Delegation Credential

**A W3C Verifiable Credential type for cryptographic human-to-agent delegation**

The normative specification is [I2H2A-v1.0.md](./I2H2A-v1.0.md).

---

## The Problem

AI agents act on behalf of humans — booking, searching, executing — but verifiers have no standard way to confirm an agent was actually authorised by a real human, for a specific task, with the ability to revoke that authority at any time.

OAuth was built for humans authorising apps. It was never designed for agents delegating across system boundaries without a human in the loop. I2H2A fills that gap.

---

## The Three Roles

**ISSUER = the platform**
The platform verifies the human's identity (via KYC, OID4VP presentation, or equivalent) and issues an I2H2A credential anchored to a public trust registry. The spec is silent on which verification method — that is a platform implementation decision.

**HOLDER = the human user**
The human receives the I2H2A credential, reviews the delegation scope, and consents. The human's DID is ledger-anchored (e.g. `did:cheqd`, `did:web`). Government wallets or commercial wallets are single-use for onboarding only — the platform credential is what the agent holds.

**AGENT = an ephemeral session**
The agent holds the I2H2A credential and its own ephemeral `did:key` private key. It constructs and signs a Verifiable Presentation autonomously and presents it to MCP servers or other verifiers. No round-trip to the issuer required.