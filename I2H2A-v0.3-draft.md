# I2H2A Specification v0.3 (Draft)

**Issuer-to-Holder-to-Agent (I2H2A) Delegation Protocol**

**Version:** 0.3-draft  
**Date:** April 29, 2026  
**Status:** Draft Specification

---

## Abstract

This specification defines the **Issuer-to-Holder-to-Agent (I2H2A)** protocol, a cryptographic delegation framework enabling humans to authorize autonomous agents to act on their behalf. I2H2A provides verifiable, revocable, scope-constrained delegation credentials using W3C Verifiable Credentials 2.0 and Decentralized Identifiers (DIDs).

The protocol is **transport-agnostic**, **DID-method-agnostic**, and **domain-agnostic**, supporting any use case requiring delegated authority: file access, API authorization, service management, resource operations, and beyond.

---

## 1. Introduction

### 1.1 Problem Statement

Autonomous agents increasingly act on behalf of humans across digital systems. However, no standardized mechanism exists for:

- Cryptographically proving an agent is authorized by a specific human
- Constraining agent authority with verifiable scope limitations  
- Revoking agent authority in real-time
- Maintaining chain-of-custody when agents delegate to other agents
- Enabling offline verification without centralized infrastructure

Existing solutions (API keys, OAuth tokens, session credentials) are insufficient:
- **Bearer tokens** are vulnerable to theft and replay attacks
- **Centralized validation** creates single points of failure
- **No cryptographic binding** between human and agent
- **Limited revocation** mechanisms (TTL-based, not real-time)
- **Platform lock-in** prevents cross-domain delegation

### 1.2 Motivating Use Cases

**Use Case 1: Document Access Delegation**  
Alice authorizes her AI research assistant to read and summarize academic papers from her university document repository. The agent operates autonomously, accessing papers as needed without requiring Alice's real-time approval for each document.

**Use Case 2: API Authorization**  
Bob delegates his calendar API access to an automation agent that schedules meetings based on email requests. The agent can create calendar entries within specified time windows but cannot delete existing events.

**Use Case 3: Service Subscription Management**  
Carol authorizes an agent to manage her streaming service subscriptions, allowing it to pause, resume, or modify plans within a monthly budget constraint.

**Use Case 4: Cloud Resource Operations**  
David delegates cloud storage management to a backup agent that can write files to designated folders but cannot delete or modify existing archives.

### 1.3 Design Goals

1. **Cryptographic Verification** - Mathematical proof of delegation, not institutional attestation alone
2. **Revocability** - Real-time credential revocation without requiring credential expiry
3. **Scope Constraints** - Verifiable limits on agent authority
4. **Chain of Custody** - Traceable delegation chains when agents sub-delegate
5. **Offline Verification** - No dependency on issuer availability for validation
6. **Interoperability** - Works across organizational and technical boundaries
7. **DID Method Agnostic** - Supports any DID method (web, key, ion, cheqd, etc.)
8. **Transport Agnostic** - Independent of presentation protocol (OID4VP, DIDComm, HTTP, etc.)
9. **Domain Agnostic** - Applicable to any delegation scenario

---

## 2. Architecture Overview

### 2.1 Actors

**Issuer**  
Entity that issues delegation credentials after verifying human authorization. Issues credentials as W3C Verifiable Credentials signed with the issuer's DID. Maintains status lists for revocation.

**Holder (Human)**  
Individual who authorizes an agent to act on their behalf. Authenticates to the issuer to request credential issuance. Controls revocation of issued credentials.

**Agent (Subject)**  
Autonomous system authorized to perform actions within defined scope. Presents verifiable presentations to verifiers. May be identified by `did:key`, `did:web`, or other DID methods.

**Verifier (Service Provider)**  
Entity that validates agent credentials before granting access. Verifies issuer signature, holder binding, scope constraints, and revocation status. Examples: API endpoints, file servers, service platforms.

### 2.2 Protocol Flow

```
1. Holder authenticates to Issuer
2. Holder requests delegation credential for Agent
3. Issuer verifies Holder identity and authorization
4. Issuer creates Verifiable Credential (VC):
   - credentialSubject.id = Agent DID
   - Scope and constraints defined
   - Signed by Issuer DID
   - Status list reference included
5. Credential delivered to Holder's wallet or storage
6. Agent retrieves credential (via wallet API, secure storage, etc.)
7. Agent presents Verifiable Presentation (VP) to Verifier:
   - Contains the VC
   - Signed by Agent DID (KB-JWT binding)
   - Includes challenge/nonce from Verifier
8. Verifier validates:
   - Issuer signature on VC
   - Agent DID matches credentialSubject.id
   - Credential not revoked (status list check)
   - Scope permits requested action
   - KB-JWT binds VP to Agent
9. Access granted or denied based on validation
```

### 2.3 Credential Lifecycle

**Issuance** → **Storage** → **Presentation** → **Verification** → **Revocation**

- Credentials issued with validity period and scope
- Holder controls credential storage and agent access
- Agent presents credentials when accessing resources
- Verifiers validate independently without issuer callback
- Holder can revoke at any time via status list update

---

## 3. Credential Structure

### 3.1 Format

I2H2A credentials MUST use **SD-JWT** (Selective Disclosure JWT, RFC 9901) with **ES256** (ECDSA P-256) signatures.

**Rationale:**
- Selective disclosure enables privacy-preserving presentations
- ES256 provides strong cryptographic security with broad library support
- JWT structure is widely understood and interoperable
- Compatible with W3C VC 2.0 data model

### 3.2 Required Fields

All I2H2A credentials MUST include:

```json
{
  "@context": [
    "https://www.w3.org/ns/credentials/v2",
    "https://i2h2a.org/contexts/v1"
  ],
  "type": ["VerifiableCredential", "I2H2ADelegationCredential"],
  "issuer": "did:example:issuer123",
  "issuanceDate": "2026-04-29T10:00:00Z",
  "expirationDate": "2027-04-29T10:00:00Z",
  "credentialSubject": {
    "id": "did:key:agent456",
    "scope": ["action:read", "action:write"],
    "constraints": {
      "validFrom": "2026-04-29T10:00:00Z",
      "validUntil": "2027-04-29T10:00:00Z"
    }
  },
  "credentialStatus": {
    "id": "https://example.com/status/list#94567",
    "type": "BitstringStatusListEntry",
    "statusListIndex": "94567",
    "statusListCredential": "https://example.com/status/list"
  }
}
```

### 3.3 Field Definitions

#### 3.3.1 Credential Subject

**`id`** (REQUIRED)  
DID of the agent being authorized. MUST match the DID used to sign the Verifiable Presentation.

**`scope`** (REQUIRED)  
Array of permitted actions. Vocabulary is domain-specific but SHOULD follow the pattern `action:operation` or `resource:type`.

Examples:
- `["action:read", "action:write"]`
- `["api:calendar.create", "api:calendar.read"]`
- `["file:read", "file:write"]`
- `["service:pause", "service:resume"]`

**`constraints`** (OPTIONAL)  
Object containing additional limitations on agent authority.

Common constraint fields:
- `validFrom` / `validUntil` - Time boundaries for credential use
- `maxOperations` - Maximum number of operations allowed
- `resourceType` - Type of resource agent can access
- `regionRestriction` - Geographic or network limitations

Domain-specific constraints MAY be added as needed.

#### 3.3.2 Credential Status

I2H2A credentials MUST include revocation status using **BitstringStatusListEntry** (W3C Bitstring Status List v1.0).

**`statusListCredential`**  
URL of the status list credential containing revocation bits.

**`statusListIndex`**  
Index position of this credential's status bit in the list.

Status list MAY be deployed on:
- Public blockchains or other distributed anchors (implementations MUST follow **[StatusList]** and applicable security practices for the deployment)
- IPFS or similar content-addressed storage
- Centralized registries with cryptographic integrity

#### 3.3.3 Optional Extensions

**`resourceReference`** (OPTIONAL)  
URI identifying the specific resource this credential authorizes access to.

Examples:
- `https://api.example.com/v1/documents/abc123`
- `gs1:01:09506000134352` (GS1 Digital Link for products)
- `did:example:resource789`

**`parentCredential`** (OPTIONAL)  
Reference to parent credential in H2A2A delegation chains (see Section 7).

---

## 4. Complete Examples

### 4.1 Example 1: File Access Delegation

```json
{
  "@context": [
    "https://www.w3.org/ns/credentials/v2",
    "https://i2h2a.org/contexts/v1"
  ],
  "type": ["VerifiableCredential", "I2H2ADelegationCredential"],
  "issuer": "did:web:issuer.example.com",
  "issuanceDate": "2026-04-29T10:00:00Z",
  "expirationDate": "2027-04-29T10:00:00Z",
  "credentialSubject": {
    "id": "did:key:z6MkhaXgBZDvotDkL5257faiztiGiC2QtKLGpbnnEGta2doK",
    "scope": ["file:read", "file:list"],
    "constraints": {
      "resourceType": "document",
      "pathPrefix": "/research/papers/",
      "maxFileSize": 10485760
    }
  },
  "credentialStatus": {
    "id": "https://status.example.com/list#12345",
    "type": "BitstringStatusListEntry",
    "statusListIndex": "12345",
    "statusListCredential": "https://status.example.com/list"
  }
}
```

### 4.2 Example 2: API Authorization

```json
{
  "@context": [
    "https://www.w3.org/ns/credentials/v2",
    "https://i2h2a.org/contexts/v1"
  ],
  "type": ["VerifiableCredential", "I2H2ADelegationCredential"],
  "issuer": "did:ion:EiD...abc",
  "issuanceDate": "2026-04-29T14:30:00Z",
  "expirationDate": "2026-05-29T14:30:00Z",
  "credentialSubject": {
    "id": "did:web:agent.automation.example",
    "scope": ["api:calendar.create", "api:calendar.read"],
    "constraints": {
      "validFrom": "2026-04-29T14:30:00Z",
      "validUntil": "2026-05-29T14:30:00Z",
      "timeWindowStart": "09:00",
      "timeWindowEnd": "17:00",
      "daysOfWeek": ["Mon", "Tue", "Wed", "Thu", "Fri"]
    }
  },
  "credentialStatus": {
    "id": "did:example:global-status-registry#67890",
    "type": "BitstringStatusListEntry",
    "statusListIndex": "67890",
    "statusListCredential": "did:example:global-status-registry"
  }
}
```

### 4.3 Example 3: Service Management

```json
{
  "@context": [
    "https://www.w3.org/ns/credentials/v2",
    "https://i2h2a.org/contexts/v1"
  ],
  "type": ["VerifiableCredential", "I2H2ADelegationCredential"],
  "issuer": "did:key:z6Mkf5rGMoatrSj1f4CyvuHBeXJELe9RPdzo2PKHorYPRJGb",
  "issuanceDate": "2026-04-29T08:00:00Z",
  "expirationDate": "2027-04-29T08:00:00Z",
  "credentialSubject": {
    "id": "did:key:z6MkhN7PBjWgSMQ2Cr9kF8PNhLK4UYvUNXN7FE67hzVKQgpF",
    "scope": ["service:pause", "service:resume", "service:modify"],
    "constraints": {
      "serviceCategory": "streaming",
      "maxMonthlySpend": 50.00,
      "currency": "USD"
    }
  },
  "credentialStatus": {
    "id": "https://registry.example.com/revocation#11223",
    "type": "BitstringStatusListEntry",
    "statusListIndex": "11223",
    "statusListCredential": "https://registry.example.com/revocation"
  }
}
```

---

## 5. Presentation and Verification

### 5.1 Verifiable Presentation Structure

Agents MUST present credentials using **Verifiable Presentations** (W3C VC 2.0) with **KB-JWT** (Key Binding JWT) proof.

```json
{
  "@context": ["https://www.w3.org/ns/credentials/v2"],
  "type": ["VerifiablePresentation"],
  "verifiableCredential": [
    { /* I2H2A credential */ }
  ],
  "proof": {
    "type": "JsonWebSignature2020",
    "created": "2026-04-29T15:00:00Z",
    "verificationMethod": "did:key:z6MkhaXgBZDvotDkL5257faiztiGiC2QtKLGpbnnEGta2doK#key-1",
    "proofPurpose": "authentication",
    "challenge": "nonce-from-verifier-abc123",
    "jws": "eyJhbGci...signature"
  }
}
```

**KB-JWT Requirements:**
- VP MUST be signed by the DID in `credentialSubject.id`
- Signature MUST include verifier-provided challenge/nonce
- Prevents replay attacks and binds presentation to specific verification session

### 5.2 Verification Process

Verifiers MUST perform the following checks in order:

#### Step 1: Structural Validation
- VP contains at least one VC
- VC conforms to I2H2A schema
- All required fields present

#### Step 2: Signature Verification
- Resolve issuer DID document
- Verify VC signature using issuer's public key
- Verify VP signature using agent's public key

#### Step 3: Holder Binding
- Extract `credentialSubject.id` from VC
- Extract `verificationMethod` from VP proof
- MUST match: `credentialSubject.id` DID === VP signer DID
- This proves the agent presenting the VP is the agent authorized by the credential

#### Step 4: Challenge Validation
- Verify VP proof includes expected challenge/nonce
- Nonce MUST match value provided by verifier
- Prevents presentation replay

#### Step 5: Status Check
- Fetch status list from `credentialStatus.statusListCredential`
- Check bit at `credentialStatus.statusListIndex`
- If bit = 1, credential is REVOKED (reject)
- If bit = 0, credential is VALID (continue)

#### Step 6: Temporal Validity
- Current time MUST be >= `issuanceDate`
- Current time MUST be < `expirationDate`
- If `constraints.validFrom` exists, current time MUST be >= `validFrom`
- If `constraints.validUntil` exists, current time MUST be < `validUntil`

#### Step 7: Scope Validation
- Extract requested action from context (e.g., API endpoint, file operation)
- Check if action is permitted by `scope` array
- Check if action satisfies all `constraints`
- If scope insufficient, DENY access

All checks MUST pass for verification to succeed.

---

## 6. Revocation

### 6.1 Status Lists

I2H2A uses **Bitstring Status Lists** (W3C Bitstring Status List v1.0) for credential revocation.

**Status List Structure:**
```json
{
  "@context": ["https://www.w3.org/ns/credentials/v2"],
  "type": ["VerifiableCredential", "BitstringStatusListCredential"],
  "issuer": "did:example:issuer123",
  "issuanceDate": "2026-04-29T00:00:00Z",
  "credentialSubject": {
    "type": "BitstringStatusList",
    "statusPurpose": "revocation",
    "encodedList": "H4sIAAAAAAAAA..." // Base64-encoded compressed bitstring
  }
}
```

**Bitstring Encoding:**
- Each credential assigned a unique index
- Bit value 0 = valid
- Bit value 1 = revoked
- List compressed with GZIP, Base64-encoded

### 6.2 Revocation Process

1. Holder requests revocation from issuer
2. Issuer authenticates holder
3. Issuer updates status list (flips bit at credential's index)
4. Updated status list published to designated location
5. Next verification check will detect revocation

**Deployment Options:**
- Blockchain ledgers (immutable, decentralized)
- IPFS (content-addressed, distributed)
- Centralized registries (performance, but single point of control)

### 6.3 Revocation Granularity

Issuers MAY implement:
- **Immediate revocation** - Status list updated in real-time
- **Batch revocation** - Status list updated periodically (e.g., hourly)
- **Permanent revocation** - Bit flip is irreversible
- **Temporary suspension** - Separate status list for suspension vs revocation

---

## 7. Agent-to-Agent Delegation (H2A2A)

### 7.1 Overview

**H2A2A (Holder-to-Agent-to-Agent)** enables authorized agents to sub-delegate authority to other agents, creating verifiable delegation chains.

**Use Case:**  
Alice authorizes Agent-A to manage her cloud storage. Agent-A delegates backup operations to Agent-B (specialized backup service). Agent-B can prove its authority traces back to Alice.

### 7.2 Chain Structure

```
Holder (Alice)
  |
  ├─ Issues I2H2A Credential-1
  |     credentialSubject.id = Agent-A DID
  |     scope = ["storage:read", "storage:write"]
  |
  └─ Agent-A receives Credential-1
       |
       ├─ Issues I2H2A Credential-2  
       |     credentialSubject.id = Agent-B DID
       |     scope = ["storage:write"]  // subset of Credential-1
       |     parentCredential = Credential-1 reference
       |
       └─ Agent-B receives Credential-2
```

### 7.3 Parent Credential Reference

Child credentials MUST include `parentCredential` field:

```json
{
  "credentialSubject": {
    "id": "did:key:agent-b-xyz",
    "scope": ["storage:write"],
    "parentCredential": {
      "id": "did:example:issuer/credentials/abc123",
      "issuer": "did:example:issuer",
      "holder": "did:key:agent-a"
    }
  }
}
```

### 7.4 Critical Security Constraint

**ROOT-OF-TRUST INVARIANT:**

ALL credentials in a delegation chain (including H2A2A child credentials) MUST be signed by the original issuer DID, never by a parent agent's DID.

**Why:**  
If Agent-A could sign credentials, it could forge arbitrary delegations beyond its authorized scope. The issuer remains the cryptographic root of trust.

**Implementation:**  
- Agent-A requests child credential from issuer
- Agent-A proves it holds valid parent credential
- Agent-A specifies sub-delegation scope (must be subset of parent scope)
- Issuer validates Agent-A's authority
- Issuer issues new credential signed by issuer DID
- Child credential references parent via `parentCredential` field

### 7.5 Verification of Chains

Verifiers MUST validate entire chain:

1. Verify leaf credential (Agent-B's credential)
2. Extract `parentCredential` reference
3. Fetch and verify parent credential (Agent-A's credential)
4. Continue recursively until root credential (no `parentCredential`)
5. All credentials must:
   - Be signed by same issuer
   - Not be revoked
   - Have valid temporal constraints
   - Have scope subsumption (child scope ⊆ parent scope)

---

## 8. Security Considerations

### 8.1 Private Key Protection

**Agent Private Keys:**
- MUST be stored in secure environments (hardware security modules, secure enclaves, encrypted keystores)
- SHOULD use key rotation mechanisms
- MUST NOT be transmitted in plaintext

**Issuer Private Keys:**
- MUST use hardware security modules (HSMs) or equivalent
- SHOULD implement multi-signature schemes for high-value issuers
- MUST have key recovery procedures

### 8.2 Replay Attack Prevention

- VP signatures MUST include verifier-provided nonce/challenge
- Nonces SHOULD be cryptographically random and single-use
- Verifiers MUST reject VPs with missing or invalid challenges

### 8.3 Scope Constraint Enforcement

- Verifiers MUST NOT grant access beyond credential scope
- Scope checks MUST be performed on every operation
- Failed scope checks SHOULD be logged for audit

### 8.4 Status List Integrity

- Status lists SHOULD be deployed on tamper-evident infrastructure
- Status list updates SHOULD be logged and auditable
- Verifiers SHOULD cache status lists with appropriate TTL

### 8.5 DID Resolution Security

- DID resolution SHOULD use multiple resolvers for redundancy
- DID documents SHOULD be verified against ledger/authoritative source
- Verifiers MUST validate DID document signatures

---

## 9. Privacy Considerations

### 9.1 Selective Disclosure

I2H2A credentials use SD-JWT to enable selective disclosure:
- Agents MAY disclose only required claims
- Unnecessary personal information SHOULD NOT be included in credentials
- Verifiers SHOULD request minimum necessary disclosures

### 9.2 Correlation Resistance

- Agents MAY use different DIDs per verifier to prevent correlation
- Status list design SHOULD minimize linkability between credential checks
- Issuers SHOULD avoid including unique identifiers that enable tracking

### 9.3 Minimal Data Exposure

- Credentials SHOULD contain only authorization data, not identity attributes
- Personal information SHOULD be in separate credentials (e.g., eKYC) not mixed into delegation credentials

---

## 10. Interoperability

### 10.1 DID Method Agnostic

I2H2A credentials MUST work with any DID method:
- `did:web` - Web-based DIDs
- `did:key` - Cryptographic key DIDs
- `did:ion` - Bitcoin-anchored DIDs
- `did:cheqd` — DIDs anchored per that method’s registered rules
- `did:ethr` - Ethereum DIDs
- Any W3C DID-compliant method

### 10.2 Transport Agnostic

VP presentation MUST be independent of transport protocol:
- **OID4VP** (OpenID for Verifiable Presentations)
- **DIDComm** (Decentralized Identity Communication)
- **HTTP/REST APIs**
- **Message queues or event streams**
- Any protocol supporting cryptographic message exchange

### 10.3 Standards Alignment

I2H2A aligns with:
- **W3C Verifiable Credentials 2.0**
- **W3C Decentralized Identifiers 1.0**
- **W3C Bitstring Status List 1.0**
- **RFC 9901** (SD-JWT)
- **RFC 7515/7517/7518** (JOSE/JWS/JWK)
- **OpenID for Verifiable Presentations**

---

## 11. Implementation Guidance

### 11.1 Issuer Implementation

Issuers MUST:
1. Authenticate holders before credential issuance
2. Generate credentials conforming to I2H2A schema
3. Sign credentials with issuer DID private key (ES256)
4. Maintain status lists for revocation
5. Provide revocation endpoints for holders
6. Publish DID documents with current public keys

Issuers SHOULD:
- Implement rate limiting to prevent abuse
- Log all credential issuance and revocation events
- Provide APIs for credential retrieval
- Support multiple DID methods for flexibility

### 11.2 Agent Implementation

Agents MUST:
1. Generate and securely store private keys
2. Retrieve credentials from holder's wallet or secure storage
3. Present VPs with KB-JWT binding
4. Include verifier-provided challenge in VP proof
5. Respect credential scope constraints

Agents SHOULD:
- Use hardware-backed keystores where available
- Implement credential caching with proper invalidation
- Monitor credential expiry and request renewal
- Support selective disclosure for privacy

### 11.3 Verifier Implementation

Verifiers MUST:
1. Generate fresh nonces for each verification session
2. Perform all verification steps in Section 5.2
3. Reject invalid or revoked credentials
4. Enforce scope constraints strictly
5. Log verification attempts for audit

Verifiers SHOULD:
- Cache issuer DID documents with TTL
- Cache status lists with appropriate refresh intervals
- Implement rate limiting on verification endpoints
- Provide clear error messages for failed verifications

---

## 12. Future Extensions

### 12.1 Potential Enhancements

**Agent Attestation:**  
Mechanism for agents to prove their capabilities or security properties (e.g., "runs in secure enclave", "audited code")

**Multi-Party Delegation:**  
Credentials requiring signatures from multiple holders (e.g., joint account authorization)

**Conditional Scope:**  
Scope that changes based on external conditions (time of day, location, resource state)

**Credential Aggregation:**  
Protocols for agents to present multiple credentials in a single VP efficiently

**Privacy-Preserving Revocation:**  
Zero-knowledge proofs for revocation status to prevent linkability

### 12.2 Standards Work

I2H2A is positioned for standardization through:
- **W3C Credentials Community Group**
- **Decentralized Identity Foundation (DIF)**
- **IETF OAuth Working Group** (complementary to OAuth agent extensions)
- **OpenID Foundation** (integration with OID4VP)

---

## 13. References

### 13.1 Normative References

- **[VC-DATA-MODEL-2.0]** W3C Verifiable Credentials Data Model 2.0  
  https://www.w3.org/TR/vc-data-model-2.0/

- **[DID-CORE]** W3C Decentralized Identifiers (DIDs) v1.0  
  https://www.w3.org/TR/did-core/

- **[StatusList]** W3C Bitstring Status List v1.0  
  https://www.w3.org/TR/vc-bitstring-status-list/

- **[RFC9901]** SD-JWT: Selective Disclosure for JWTs  
  https://datatracker.ietf.org/doc/html/rfc9901

- **[RFC7515]** JSON Web Signature (JWS)  
  https://datatracker.ietf.org/doc/html/rfc7515

- **[RFC7517]** JSON Web Key (JWK)  
  https://datatracker.ietf.org/doc/html/rfc7517

### 13.2 Informative References

- **[OID4VP]** OpenID for Verifiable Presentations  
  https://openid.net/specs/openid-4-verifiable-presentations-1_0.html

- **[DIDComm]** DIDComm Messaging Specification  
  https://identity.foundation/didcomm-messaging/spec/

---

## Appendix A: Terminology

**Agent**  
Autonomous software system authorized to act on behalf of a human holder.

**Delegation**  
Process of authorizing an agent to perform actions within defined scope.

**Holder**  
Human individual who controls credential issuance and revocation.

**Issuer**  
Trusted entity that issues delegation credentials after verifying holder authorization.

**KB-JWT (Key Binding JWT)**  
Cryptographic proof that binds a Verifiable Presentation to the presenter's private key.

**Scope**  
Set of permitted actions an agent is authorized to perform.

**Status List**  
Bitstring structure enabling efficient revocation checking.

**Verifier**  
Service provider that validates agent credentials before granting access.

**VP (Verifiable Presentation)**  
Signed package containing one or more Verifiable Credentials, proving presenter controls the subject DID.

---

## Appendix B: Full SD-JWT Example

```
Issuer JWT (signed with issuer DID private key):
eyJhbGciOiJFUzI1NiIsInR5cCI6InZjK3NkLWp3dCIsImtpZCI6ImRpZDpleGFtcGxlOmlzc3VlcjEyMyNrZXktMSJ9.eyJfc2QiOlsiYWJjMTIzIiwiZGVmNDU2Il0sImlzcyI6ImRpZDpleGFtcGxlOmlzc3VlcjEyMyIsImlhdCI6MTY1MDAwMDAwMCwiZXhwIjoxNjgxNTM2MDAwLCJ2YyI6eyJAY29udGV4dCI6WyJodHRwczovL3d3dy53My5vcmcvbnMvY3JlZGVudGlhbHMvdjIiLCJodHRwczovL2kyaDJhLm9yZy9jb250ZXh0cy92MSJdLCJ0eXBlIjpbIlZlcmlmaWFibGVDcmVkZW50aWFsIiwiSTJIMkFEZWxlZ2F0aW9uQ3JlZGVudGlhbCJdLCJjcmVkZW50aWFsU3ViamVjdCI6eyJpZCI6ImRpZDprZXk6ejZNa2hhWGdCWkR2b3REa0w1MjU3ZmFpenRpR2lDMlF0S0xHcGJubkVHdGEyZG9LIiwic2NvcGUiOlsiZmlsZTpyZWFkIiwiZmlsZTpsaXN0Il0sImNvbnN0cmFpbnRzIjp7InJlc291cmNlVHlwZSI6ImRvY3VtZW50IiwicGF0aFByZWZpeCI6Ii9yZXNlYXJjaC9wYXBlcnMvIn19LCJjcmVkZW50aWFsU3RhdHVzIjp7ImlkIjoiaHR0cHM6Ly9zdGF0dXMuZXhhbXBsZS5jb20vbGlzdCMxMjM0NSIsInR5cGUiOiJCaXRzdHJpbmdTdGF0dXNMaXN0RW50cnkiLCJzdGF0dXNMaXN0SW5kZXgiOiIxMjM0NSIsInN0YXR1c0xpc3RDcmVkZW50aWFsIjoiaHR0cHM6Ly9zdGF0dXMuZXhhbXBsZS5jb20vbGlzdCJ9fX0.signature-here

Disclosures (Base64-encoded):
WyJzYWx0MSIsICJyZXNvdXJjZVR5cGUiLCAiZG9jdW1lbnQiXQ
WyJzYWx0MiIsICJwYXRoUHJlZml4IiwgIi9yZXNlYXJjaC9wYXBlcnMvIl0

Holder Binding JWT (signed with agent DID private key):
eyJhbGciOiJFUzI1NiIsInR5cCI6ImtiK2p3dCIsImtpZCI6ImRpZDprZXk6ejZNa2hhWGdCWkR2b3REa0w1MjU3ZmFpenRpR2lDMlF0S0xHcGJubkVHdGEyZG9LI2tleS0xIn0.eyJub25jZSI6Im5vbmNlLWZyb20tdmVyaWZpZXItYWJjMTIzIiwiaWF0IjoxNjUwMDEwMDAwLCJhdWQiOiJodHRwczovL3ZlcmlmaWVyLmV4YW1wbGUuY29tIiwic2RfaGFzaCI6Imhhc2gtb2YtaXNzdWVyLWp3dCJ9.holder-binding-signature

Complete SD-JWT:
<Issuer JWT>~<Disclosure1>~<Disclosure2>~<Holder Binding JWT>
```

---

**End of Specification**

---

## Document Change Log

**v0.3-draft (April 29, 2026)**
- Removed all e-commerce-specific examples and terminology
- Removed transport coupling (MCP references)
- Removed DID method bias (single-network examples only)
- Added diverse use case examples (file access, API auth, service mgmt, resource ops)
- Added DID method diversity (did:example, did:web, did:key, did:ion, did:cheqd)
- Abstracted vocabulary (resource, action, service instead of product, purchase, merchant)
- Removed platform/vendor references (Shopify, Amazon, Rainforest, Rotavera)
- Removed people/org references (Alex Tweeddale)
- Clarified A2AUAS is NOT part of I2H2A spec
- Emphasized transport-agnostic, DID-agnostic, domain-agnostic design

**v0.2 (Previous Draft)**
- Initial draft with e-commerce focus
- MCP transport coupling
- Network-specific illustrative examples only

---

This specification defines ONLY the cryptographic delegation envelope.  
Intelligence layers, transport protocols, and domain applications are implementation details.
