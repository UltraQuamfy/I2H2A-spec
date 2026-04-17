# I2H2A Vocabulary

**URI:** https://ultraquamfy.github.io/I2H2A-spec/vocab
**Status:** Draft
**Version:** 0.2

This document defines the vocabulary terms used by the I2H2A credential type and the Human Identity VC. All terms are selectively disclosable via SD-JWT unless marked as always-visible.

---

## Credential type terms

### I2H2A

URI: `https://ultraquamfy.github.io/I2H2A-spec/vocab#I2H2A`
The I2H2A delegation credential type. Used as the `vct` claim value in SD-JWT VC. Always visible.

### HumanIdentityCredential

URI: `https://ultraquamfy.github.io/I2H2A-spec/vocab#HumanIdentityCredential`
The Human Identity VC credential type. Used as the `vct` claim value in SD-JWT VC. Always visible.

---

## I2H2A credential terms

### delegatedBy

URI: `https://ultraquamfy.github.io/I2H2A-spec/vocab#delegatedBy`
The DID of the verified human holder who delegated authority to the agent. Selectively disclosable. MUST equal the `sub` of the Human Identity VC issued to the same human.

### parentCredential

URI: `https://ultraquamfy.github.io/I2H2A-spec/vocab#parentCredential`
Reference to a parent I2H2A credential in an H2A2A chain. MUST be `null` for V1 (H2A). Selectively disclosable.

### delegationDepth

URI: `https://ultraquamfy.github.io/I2H2A-spec/vocab#delegationDepth`
Non-negative integer indicating delegation chain depth. MUST be `0` for V1. Selectively disclosable.

### scope

URI: `https://ultraquamfy.github.io/I2H2A-spec/vocab#scope`
Object defining the permitted operations for the delegated agent. Contains `mcpServers` and `taskType`. Selectively disclosable.

### mcpServers

URI: `https://ultraquamfy.github.io/I2H2A-spec/vocab#mcpServers`
Array of MCP server identifiers the agent is permitted to interact with. Selectively disclosable. Verifiers MUST enforce this list.

### taskType

URI: `https://ultraquamfy.github.io/I2H2A-spec/vocab#taskType`
String identifying the class of task the agent is permitted to perform. Selectively disclosable. Verifiers MUST enforce this value.

### constraints

URI: `https://ultraquamfy.github.io/I2H2A-spec/vocab#constraints`
Optional object carrying additional platform-defined constraints on agent behaviour. Selectively disclosable.

### authorization

URI: `https://ultraquamfy.github.io/I2H2A-spec/vocab#authorization`
Opaque platform-specific payload. Contains the A2AUAS payload in Ultra Quamfy deployments. Generic I2H2A verifiers MUST NOT interpret its contents. Selectively disclosable.

---

## Human Identity VC terms

### kycAssuranceLevel

URI: `https://ultraquamfy.github.io/I2H2A-spec/vocab#kycAssuranceLevel`
KYC assurance level string (e.g. `substantial`, `high`). Always visible.

### kycProvider

URI: `https://ultraquamfy.github.io/I2H2A-spec/vocab#kycProvider`
Identifier of the KYC provider that performed verification (e.g. `regula`). Selectively disclosable.

### biometricBindingRef

URI: `https://ultraquamfy.github.io/I2H2A-spec/vocab#biometricBindingRef`
Reference to the biometric binding record created during KYC liveness check. Selectively disclosable.

### documentTypeVerified

URI: `https://ultraquamfy.github.io/I2H2A-spec/vocab#documentTypeVerified`
The type of identity document verified during KYC (e.g. `drivers_licence`, `passport`). Selectively disclosable.

### documentIssuingCountry

URI: `https://ultraquamfy.github.io/I2H2A-spec/vocab#documentIssuingCountry`
ISO 3166-1 alpha-2 country code of the document issuing authority. Selectively disclosable.

---

## SD-JWT structural terms (informative)

These terms are defined by RFC 9901 and used in I2H2A credentials. They are not defined in this vocabulary but are referenced for completeness.

| Term | Source | Description |
|------|--------|-------------|
| `_sd` | RFC 9901 | Array of SHA-256 hashes of disclosure objects |
| `_sd_alg` | RFC 9901 | Hash algorithm — MUST be `sha-256` |
| `cnf.jwk` | RFC 7800 | Agent P-256 public key for KB-JWT holder binding |
| `vct` | SD-JWT VC draft | Verifiable credential type string |

---

## Algorithm

I2H2A v0.2 mandates **ES256 / P-256** for all signatures (issuer JWT, KB-JWT). EdDSA/Ed25519 was used in v0.1 and is superseded.
