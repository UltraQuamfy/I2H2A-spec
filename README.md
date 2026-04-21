# I2H2A — Issuer to Holder to Agent Delegation Credential

**A W3C-aligned SD-JWT VC credential type for cryptographic human-to-agent delegation**

I2H2A defines a standard credential type so a verified human holder can cryptographically delegate authority to an autonomous agent (e.g. for MCP servers) without platform lock-in. It fills the gap between "user consented in a UI" and "verifier can prove delegation, scope, and revocation" using SD-JWT VC (RFC 9901), ES256/P-256 signatures, and Bitstring Status List revocation (HTTPS-resolvable status lists).

Implementers who issue or verify delegated agent access — wallet vendors, identity providers, MCP server operators, and standards bodies — should adopt it to enable a common verification and interoperability story across ecosystems.

## Current version

The normative specification is **[I2H2A-v0.2-draft.md](./I2H2A-v0.2-draft.md)** — SD-JWT VC, ES256/P-256, field visibility map.

Previous version: [I2H2A-v0.1.md](./I2H2A-v0.1.md) — superseded. SD-JWT VC, ES256/P-256. Retained for reference only.

## Format

I2H2A v0.2 credentials are encoded as **SD-JWT VCs** (RFC 9901) with **ES256/P-256** signatures throughout. Selective disclosure allows verifiers to receive only the minimum claims required — delegation scope, task type, delegated-by — without exposing the full authorization payload.

## Key properties

- **Format:** SD-JWT VC (RFC 9901)
- **Algorithm:** ES256 / P-256
- **Issuer DID:** any DID method with a JsonWebKey2020 verification method (e.g. did:key, did:web, did:cheqd)
- **Agent DID:** `did:key` (P-256, session-scoped)
- **Holder binding:** KB-JWT signed by agent P-256 key (`cnf.jwk`)
- **Revocation:** Bitstring Status List (HTTPS-resolvable)
- **VP mechanics:** SD-JWT+KB presented at MCP session initiation via OID4VP

## Chain models

- **H2A** — Issuer → Holder → Agent → Verifier. Terminal delegation (`delegationDepth: 0`). Current V1 scope.
- **H2A2A** — Issuer → Holder → ParentAgent → ChildAgent → Verifier. Defined in spec, not implemented in V1.

## Root-of-trust invariant

Every credential in every chain MUST trace back to a verified human `delegatedBy` DID. Child agent credentials MUST be issuer-signed, never parent-agent-signed.

## Examples

See `examples/` for complete SD-JWT VC samples:

- [H2A with did:cheqd](./examples/h2a-cheqd-example.json) — illustrative did:cheqd holder and issuer (example JSON), P-256
- [H2A with did:key](./examples/h2a-didkey-example.json) — ephemeral holder and agent DIDs
- [H2A with did:web](./examples/h2a-didweb-example.json) — web-hosted holder DID

## JSON-LD context

Live context: `https://ultraquamfy.github.io/I2H2A-spec/contexts/v1.jsonld`

Source: [contexts/v1.jsonld](./contexts/v1.jsonld)

## Vocabulary

[vocab.md](./vocab.md)

## Middleware

Reference implementation of SD-JWT+KB verification for any integration point:

[UltraQuamfy/I2H2A-middleware](https://github.com/UltraQuamfy/I2H2A-middleware)

```
npm install @rotavera/verification-sdk
```

## License

MIT