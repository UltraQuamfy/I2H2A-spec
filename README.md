# I2H2A — Issuer to Holder to Agent Delegation Credential

**A W3C-aligned SD-JWT VC credential type for cryptographic human-to-agent delegation**

I2H2A defines a standard credential type so a verified human holder can cryptographically delegate authority to an autonomous agent (e.g. for MCP servers) without platform lock-in. It fills the gap between "user consented in a UI" and "verifier can prove delegation, scope, and revocation" using SD-JWT VC (RFC 9901), ES256/P-256 signatures, and Bitstring Status List revocation (HTTPS-resolvable status lists).

Implementers who issue or verify delegated agent access — wallet vendors, identity providers, MCP server operators, and standards bodies — should adopt it to enable a common verification and interoperability story across ecosystems.

## Current versions

The latest specification draft is **[I2H2A-v0.3-draft.md](./I2H2A-v0.3-draft.md)** — protocol overview (transport‑, DID‑, and domain‑agnostic delegation), credential semantics, VP/KB‑JWT flows, Bitstring revocation, and H2A2A delegation chains.

The **concrete SD‑JWT VC profile** used by **`examples/`**, the JSON‑LD context, the **[UCP integration profile](./docs/I2H2A-UCP-Profile.md)**, and the reference **[`@i2h2a/verification-sdk`](https://github.com/i2h2a-org/I2H2A-middleware)** verifier remains **[I2H2A-v0.2-draft.md](./I2H2A-v0.2-draft.md)** (MCVI‑aligned byte layout, field visibility map, unchanged).

Historical: **[I2H2A-v0.1.md](./I2H2A-v0.1.md)** — superseded; retained for reference only.

## Format

**v0.3 draft** describes delegation at the VC / protocol layer (including SD‑JWT and KB‑JWT requirements in prose).

**v0.2 draft** codifies interoperable payloads: **SD‑JWT VC** (RFC 9901) with **ES256/P‑256**, `vct` `https://i2h2a.org/credentials/I2H2A`, selective disclosure, and KB‑JWT holder binding to `cnf.jwk` — the shape the middleware validates today.

## Key properties

- **Format:** SD-JWT VC (RFC 9901)
- **Algorithm:** ES256 / P-256
- **Issuer DID:** any DID method with a JsonWebKey2020 verification method (e.g. did:key, did:web, did:example)
- **Agent DID:** SHOULD use `did:key` for ephemeral sessions; MAY use any DID method
- **Holder binding:** KB-JWT signed by agent P-256 key (`cnf.jwk`)
- **Revocation:** Bitstring Status List (HTTPS-resolvable)
- **VP mechanics:** SD‑JWT+KB presentations (v0.2 deployments commonly use OID4VP; MCP is one optional transport among many)

## Chain models

- **H2A** — Issuer → Holder → Agent → Verifier. Terminal delegation (`delegationDepth: 0`). Current V1 scope.
- **H2A2A** — Issuer → Holder → ParentAgent → ChildAgent → Verifier. Defined in spec, not implemented in V1.

## Root-of-trust invariant

Every credential in every chain MUST trace back to a verified human `delegatedBy` DID. Child agent credentials MUST be issuer-signed, never parent-agent-signed.

## Examples

See `examples/` for complete SD-JWT VC samples:

- [H2A with did:example](./examples/h2a-cheqd-example.json) — illustrative did:example holder and issuer (example JSON), P-256
- [H2A with did:key](./examples/h2a-didkey-example.json) — ephemeral holder and agent DIDs
- [H2A with did:web](./examples/h2a-didweb-example.json) — web-hosted holder DID

## JSON-LD context

Live context: `https://i2h2a.org/contexts/v1.jsonld`

Source: [contexts/v1.jsonld](./contexts/v1.jsonld)

## Vocabulary

[vocab.md](./vocab.md)

## Middleware

Reference implementation of SD-JWT+KB verification for any integration point:

[I2H2A-middleware](https://github.com/i2h2a-org/I2H2A-middleware)

```
npm install @i2h2a/verification-sdk
```

## License

MIT