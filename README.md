# I2H2A — Issuer to Holder to Agent Delegation Credential

**A W3C Verifiable Credential type for cryptographic human-to-agent delegation**

I2H2A defines a standard credential type so a human holder can cryptographically delegate authority to an autonomous agent (e.g. for MCP servers) without platform lock-in. It fills the gap between “user consented in a UI” and “verifier can prove delegation, scope, and revocation” using interoperable W3C Verifiable Credentials. Implementers who issue or verify delegated agent access—wallet vendors, identity providers, MCP hosts, and standards bodies—should adopt it to enable a common verification and interoperability story across ecosystems.

The normative specification is **[I2H2A-v1.0.md](./I2H2A-v1.0.md)**.

## Examples

See `examples/` directory for complete I2H2A credential samples:

- [H2A with did:cheqd](examples/h2a-cheqd-example.json) — On-chain anchored holder DID
- [H2A with did:web](examples/h2a-didweb-example.json) — Web-hosted holder DID
- [H2A with did:key](examples/h2a-didkey-example.json) — Ephemeral holder and agent DIDs

Each example shows the full JWT-VC structure with decoded header and payload.

## License

This repository is licensed under the [MIT License](./LICENSE).

## Contributing

This is an open specification. Submit issues or PRs to propose changes.
