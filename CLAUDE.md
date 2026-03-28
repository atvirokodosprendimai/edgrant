# EdGrant — Project Rules

## Protocol Invariants

These are non-negotiable design constraints. All contributions must respect them.

### Resource Owner Sovereignty

The resource owner owns every decision about granting access. What evidence the resource owner requires — bot owner co-approval, risk assessment, scope limits, time-of-day restrictions, or nothing at all — is the resource owner's policy. The protocol does not prescribe approval chains.

### Two-Entity Protocol

The protocol has exactly two entities: **requestor** and **resource owner**. Any additional participants (bot owner, risk engine, approval committee) are the resource owner's policy concern, not protocol entities. Never introduce a third protocol-level entity.

## Terminology

- Use **requestor** (not "bot", "client", or "agent") when referring to the protocol entity requesting access
- Use **resource owner** (not "server", "provider", or "issuer") when referring to the entity controlling the resource
- Use **capability token** (not "access token", "grant token", or "permission") for the signed artifact
- "Bot" is acceptable in use-case-specific contexts where the requestor is concretely a bot
