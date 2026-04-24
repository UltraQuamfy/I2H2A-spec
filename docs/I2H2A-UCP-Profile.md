# I2H2A-UCP Integration Profile

## 1. Overview

This document defines a normative integration profile for using I2H2A SD-JWT+KB credentials in Universal Commerce Protocol (UCP) flows.

I2H2A and UCP address different trust layers:

- OAuth 2.0 (in UCP) proves the calling platform identity and platform authorization.
- I2H2A proves that a specific human delegated authority to a specific agent key for a constrained scope.

I2H2A is complementary to UCP, not a replacement for UCP authentication or payment rails.

In the UCP actor model:

- Platform/Agent: presents OAuth credentials and an I2H2A presentation.
- Business (Merchant): verifies both OAuth context and I2H2A delegation proof.
- Credential Provider (CP): remains responsible for payment/identity credentials unrelated to I2H2A delegation proof.
- Payment Service Provider (PSP): remains responsible for payment processing.

This profile applies across UCP transports (REST, MCP, A2A, Embedded).

---

## 2. Credential Requirements

### 2.1 Required I2H2A fields for UCP use

Implementations using this profile MUST require an I2H2A SD-JWT VC that conforms to I2H2A v0.2, including:

- `vct` = `https://rotavera.io/credentials/I2H2A`
- `iss`, `sub`, `iat`, `nbf` (if present), `exp`
- `cnf.jwk` with `kty=EC`, `crv=P-256`, `x`, `y`
- `_sd_alg` = `sha-256`
- `credentialStatus` (`type=BitstringStatusListEntry`, `statusListIndex`, `statusListCredential`)
- KB-JWT with `aud`, `nonce`, `sd_hash`, signed by key corresponding to `cnf.jwk`

Verifiers MUST enforce:

- issuer signature verification (ES256/P-256)
- holder binding via KB-JWT signature and `cnf.jwk`
- KB binding checks (`aud`, `nonce`, `sd_hash`)
- temporal validity
- status check when `credentialStatus` is present
- disclosure hash checks against `_sd`
- delegation checks (`delegationDepth`, `parentCredential`, required scope disclosure)

### 2.2 Scope mapping to UCP operations

Current I2H2A v0.2 fields are `scope.mcpServers` and `scope.taskType`.

For UCP usage:

- `scope.taskType` SHOULD represent the operation class (for example: checkout, discovery, order).
- `scope.mcpServers` SHOULD be interpreted as verifier/service identifiers in the current implementation, including UCP merchant verifier identifiers where used.

Because UCP capability identifiers are namespaced (for example `dev.ucp.shopping.checkout`), implementers SHOULD define a shared mapping policy from UCP operation/capability to `scope.taskType` values.

### 2.3 Authorization object for UCP-specific constraints

The `authorization` disclosure is opaque to generic verifiers and MAY carry UCP-specific policy constraints.  
When used in UCP, `authorization` SHOULD contain only constraints already agreed by participating parties (for example amount limits, merchant binding, narrow expiry windows, operation limitations), while preserving backward compatibility with generic I2H2A verification.

Generic I2H2A verifiers MUST NOT require `authorization` semantics beyond structural presence unless platform policy explicitly requires it.

### 2.4 Credential lifetime guidance

I2H2A v0.2 requires temporal checks (`nbf`, `exp`).  
For checkout-grade risk posture, issuers SHOULD use short validity windows appropriate for session-bound delegation and SHOULD minimize credential reuse windows.

---

## 3. Transport Carriage (Normative)

### 3.1 Core rule

Implementations MUST NOT overload OAuth `Authorization: Bearer` with I2H2A presentation data.

OAuth and I2H2A MUST be carried as separate artifacts.

### 3.2 REST/HTTP

- OAuth token MUST be carried in `Authorization: Bearer <oauth-token>`.
- I2H2A presentation MUST be carried in `X-I2H2A-Presentation: <sd-jwt+kb>`.

### 3.3 MCP

- I2H2A presentation MUST be carried in request metadata at `meta.i2h2a.presentation`.

### 3.4 A2A

- I2H2A presentation MUST be carried in equivalent protocol metadata associated with the request/session envelope.

### 3.5 Embedded Protocol

- I2H2A presentation MUST be carried in session context metadata provided to embedded flow handlers.

---

## 4. Nonce and Audience Binding

### 4.1 Nonce requirements

Verifiers MUST issue nonce values that are:

- unique per verification attempt
- single-use
- bound to the verifier session/challenge context

Per transport:

- REST: nonce SHOULD be bound to merchant checkout session/challenge state.
- MCP: nonce SHOULD be bound to MCP session/tool-call challenge context.
- A2A: nonce SHOULD be bound to A2A request/session context.
- Embedded: nonce SHOULD be bound to embedded session context.

### 4.2 Audience requirements

KB-JWT `aud` MUST match the verifier audience configured by the receiving merchant/verifier integration.

Audience SHOULD be a stable verifier identifier (for example merchant verifier endpoint identifier, service identifier, or equivalent verifier audience value configured in deployment).

### 4.3 Anti-replay

Verifiers MUST reject presentations when:

- nonce mismatch
- audience mismatch
- `sd_hash` mismatch
- expired or not-yet-valid credential

Verifiers SHOULD enforce narrow nonce lifetimes and replay cache windows.

---

## 5. Verification Flow

### 5.1 UCP merchant verification sequence

A UCP merchant integration SHOULD execute verification in this order:

1. Validate transport-level request integrity per UCP.
2. Validate OAuth/platform authentication per UCP policy.
3. Extract I2H2A presentation from transport-specific location.
4. Call middleware verification (`verifyI2H2APresentation`) with:
   - presentation token
   - verifier audience
   - verifier nonce
   - resolver URL when issuer DID is non-`did:key`
5. Enforce business policy on returned claims.
6. Continue UCP operation only if verification is valid and policy checks pass.

### 5.2 Middleware response shape

Current middleware response shape:

```json
{
  "valid": true,
  "error": "optional string when invalid",
  "claims": {
    "agentDid": "did:key:...",
    "delegatedBy": "did:...",
    "scope": {
      "mcpServers": ["..."],
      "taskType": "..."
    },
    "authorization": {}
  }
}
```

### 5.3 Mapping failures to UCP responses

UCP response code mapping is transport-specific. Implementations SHOULD map I2H2A verification failures to UCP authentication/authorization error handling conventions without changing UCP protocol semantics.

Recommended baseline:

- Missing/invalid I2H2A presentation: authorization/authentication failure path
- Signature/holder binding/replay failure: authorization/authentication failure path
- Scope violation: authorization failure path
- Resolver/status infrastructure failure: transient/failure path with retry semantics where appropriate

### 5.4 Latency guidance

For UCP checkout latency sensitivity, deployments SHOULD:

- cache DID resolution results
- cache status list fetches with bounded TTL
- set resolver and status fetch timeout budgets
- pre-warm verifier dependencies where possible
- fail predictably under timeout/dependency outage conditions

---

## 6. OAuth Coexistence (Normative)

1. UCP OAuth processing and I2H2A verification are both REQUIRED in this profile for delegated-agent commerce flows.
2. OAuth and I2H2A MUST be processed as separate credentials.
3. Implementations MUST process OAuth first, then I2H2A.
4. OAuth success MUST NOT imply delegation authority.
5. I2H2A success MUST NOT replace platform authentication requirements.

---

## 7. Conformance Examples

### 7.1 Example 1: REST checkout request with OAuth + I2H2A

```http
POST /checkout-sessions/abc/complete HTTP/1.1
Authorization: Bearer eyJhbGciOi...
UCP-Agent: profile="https://platform.example/.well-known/ucp-profile.json"
X-I2H2A-Presentation: <issuer-jwt>~<disclosure>~...~<kb-jwt>
Content-Type: application/json

{
  "payment": {
    "instruments": [
      {
        "handler_id": "processor_tokenizer",
        "credential": { "token": "tok_123" }
      }
    ]
  }
}
```

### 7.2 Example 2: MCP tool call with metadata-carried I2H2A

```json
{
  "jsonrpc": "2.0",
  "method": "tools/call",
  "params": {
    "name": "complete_checkout",
    "arguments": {
      "meta": {
        "ucp-agent": {
          "profile": "https://platform.example/.well-known/ucp-profile.json"
        },
        "i2h2a": {
          "presentation": "<issuer-jwt>~<disclosure>~...~<kb-jwt>"
        }
      },
      "checkout": {
        "id": "checkout_123"
      }
    }
  },
  "id": 1
}
```

### 7.3 Example 3: Verification failure mapping

REST example:

```json
{
  "code": "authorization_failed",
  "content": "I2H2A verification failed: KB-JWT nonce mismatch"
}
```

MCP example:

```json
{
  "jsonrpc": "2.0",
  "id": 1,
  "error": {
    "code": -32000,
    "message": "Unauthorized",
    "data": {
      "reason": "I2H2A verification failed: Issuer JWT signature invalid"
    }
  }
}
```

---

## 8. What Is Out of Scope

This profile does not:

- replace or redefine UCP payment credential flows
- replace Credential Providers (for example wallet/token providers)
- define UCP checkout state machines or capability negotiation protocol
- define payment instrument/tokenization semantics

I2H2A in this profile is strictly the human delegation proof layer used alongside existing UCP mechanisms.

---

## 9. Conformance

An implementation claiming conformance to this profile:

- MUST use I2H2A v0.2 SD-JWT+KB semantics and field requirements
- MUST carry I2H2A separately from OAuth bearer tokens
- MUST enforce nonce and audience binding checks
- MUST enforce scope and delegation constraints from disclosures
- SHOULD provide transport-specific error mapping consistent with UCP handling conventions
