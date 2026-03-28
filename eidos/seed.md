# EdGrant — Permission Grant Protocol for EdProof Identities

Scoped, time-limited, auditable access grants. Built on EdProof identity.

## Core Idea

EdProof is the passport — long-lived identity. EdGrant is the visa — temporal permission.

A requestor discovers what a resource accepts, requests specific permissions for a specific duration, and the resource owner decides. The requestor carries the grant as a capability token — verified locally, no issuer contacted.

## Protocol Entities

Two entities only: **requestor** and **resource owner**. Any additional approval chain (bot owner, risk engine) is the resource owner's policy concern, not protocol.

## Protocol Phases

0. **Discovery** — requestor approaches with identity only, resource describes what it accepts
1. **Request** — requestor signs a request for specific scopes and TTL
2. **Grant** — resource owner evaluates policy, issues capability token
3. **Access** — requestor presents identity + token, resource verifies locally

## Key Design Decisions

- Scopes are domain-specific and opaque to the protocol
- No permanent grants — everything has a TTL
- Revocation is a registry concern (same as EdProof identity revocation)
- Namespace-separated signatures prevent cross-protocol replay
- Formally verified in Tamarin (7 lemmas, Dolev-Yao adversary)

## Specs

- `spec - edgrant protocol.md` — core protocol specification
- `spec - permission grant use cases.md` — concrete scenarios (Jira, Grafana, CI/CD, data stores)

## Status

RFC v0.1 complete. Formal model verified. No reference implementation yet.
