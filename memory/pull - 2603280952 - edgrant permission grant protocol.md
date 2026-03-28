# Pull — EdGrant Permission Grant Protocol

**Date:** 2026-03-28
**Sources:** RFC-EDGRANT.md, formal/edgrant.spthy, brainstorm, research, spec - permission grant use cases.md, README.md

## Collected Material

### Core Protocol Flow (RFC-EDGRANT §3-7)

Four-phase protocol between requestor and resource owner:

- **Phase 0 — Discovery** (optional): Requestor approaches with EdProof credential only → resource rejects with 401 + requirements response (available scopes, max TTL, approval endpoint, grant format)
- **Phase 1 — Request**: Requestor constructs signed request (resource, scopes, TTL, reason) using sshsig namespace `edgrant-request`
- **Phase 2 — Grant**: Resource owner evaluates policy → issues capability token encrypted over TLS, signed with namespace `edgrant-grant`
- **Phase 3 — Access**: Requestor presents EdProof credential + capability token → resource verifies locally (identity, grant signature, requestor binding, time validity, scope match, optional revocation check)

### Capability Token (RFC-EDGRANT §6.3)

Self-contained signed document carrying: grant_id, requestor_fingerprint, resource, scopes, not_before/not_after, resource_owner_fingerprint, resource_owner_signature.

Format-agnostic: JSON, Biscuit, SSH certificates all valid. Required fields are protocol-level; format is implementation choice.

### Scope Model (RFC-EDGRANT §8)

Structure: resource + action + optional constraints. Scopes are domain-specific opaque strings. No central registry. Resource advertises available scopes in discovery. Requestor MUST choose from advertised set.

### Revocation (RFC-EDGRANT §9)

TTL-based expiry is primary mechanism. Pre-TTL revocation via registry (same model as EdProof identity revocation). Short TTLs are the expected pattern (minutes to hours). Long TTLs are explicitly anti-pattern.

### Security (RFC-EDGRANT §10, formal/edgrant.spthy)

Dolev-Yao adversary model. Namespace separation prevents cross-phase signature replay (`edgrant-request` vs `edgrant-grant` vs `edproof`). Scope escalation structurally prevented — both signatures cover the same (resource, scopes, ttl) tuple.

7 machine-checked lemmas in Tamarin (0.66s):
- request_authenticity, no_scope_escalation, bot_binding, no_request_forgery, grant_secrecy, grant_authorization, executability

### Legacy Adaptation (RFC-EDGRANT §12)

Sidecar pattern for non-EdGrant-native resources. Sidecar is the verifier, holds real resource credentials, proxies authorized operations. Resource owner's implementation concern.

### Design Philosophy (RFC-EDGRANT §1, brainstorm clusters)

"Visa model" — EdProof is passport (identity), EdGrant is visa (permission). Six principles: discovery-first, resource owner decides, least privilege, token-is-document, domain-specific scopes, revocation-is-registry.

Key architectural decision from brainstorm: bot owner approval is policy, not protocol. The RFC simplified from dual-signed (brainstorm A3) to resource-owner-signed only. Bot owner co-approval is one possible policy the resource owner MAY require.

### Prior Art Positioning (research doc)

Unique combination: request-initiated + decentralized + identity-separated + formally verified. Borrows: attenuation from UCAN/Biscuit, request-initiation from UMA, key-bound tokens from GNAP, scope structure from OAuth RAR. Novel: dual-approval option, git-native audit, clean identity/permission separation.

## Patterns

- **Composition over extension**: EdGrant composes with EdProof, doesn't extend it
- **Sovereignty**: Resource owner owns every decision — consistent with EdProof Layer 4
- **Document model**: Token is a document (verified locally), not a session (requires issuer contact)
- **Registry for mutable state**: Revocation follows same pattern as EdProof identity revocation
- **sshsig with namespaces**: Reuses EdProof's signing convention, prevents cross-protocol replay

## Dependencies

- EdProof identity (Ed25519 credential, five-layer model) — required foundation
- sshsig wire format — signing convention
- TLS — grant delivery encryption

## Intent Sketch

**What someone needs to know to re-implement EdGrant differently:**

- Any entity with an EdProof identity can request scoped, time-limited permission to any resource
- The resource teaches the requestor what to ask for (discovery-first)
- The resource owner alone decides whether to grant — what evidence they require is their policy
- The grant is a self-contained capability token that the requestor carries and presents — the resource owner is not contacted at access time
- Scopes are domain-specific and opaque to the protocol — each resource defines its own vocabulary
- Grants are inherently temporal — short TTLs are expected, permanent grants are anti-pattern
- Revocation before TTL expiry uses the same registry mechanism as EdProof identity revocation
- Namespace-separated signatures prevent cross-protocol and cross-phase replay
- Legacy resources use a sidecar proxy pattern — the sidecar is the verifier
- The protocol is formally verified under Dolev-Yao: request authenticity, no scope escalation, bot binding, no request forgery, grant secrecy, grant authorization
