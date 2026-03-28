---
tldr: Permission grant protocol for EdProof identities — scoped, time-limited, auditable access via capability tokens
category: core
---

# EdGrant Protocol

## Target

A permission grant protocol that composes with EdProof identity. Any entity with an EdProof credential can request scoped, time-limited access to any resource. The resource owner decides whether to grant.

EdProof is the passport. EdGrant is the visa.

## Behaviour

### Discovery-First Access

- A requestor approaches a resource with only its EdProof identity — no pre-knowledge of the resource's permission model
- The resource rejects the attempt and describes what it accepts: available scopes, maximum TTL, where to submit requests
- The requestor constructs an informed request from this description
- Discovery is optional — a requestor that already knows a resource's requirements may skip it

### Request-Grant-Access Lifecycle

- The requestor signs a request specifying the resource, desired scopes, desired TTL, and a reason
- The resource owner evaluates the request against its own policy and decides
- If granted, the resource owner issues a capability token — a signed, time-limited, scope-bound document
- The requestor carries the token and presents it alongside its EdProof credential to access the resource
- The resource verifies both locally — the resource owner is not contacted at access time

### Capability Token

- Self-contained signed document — verified locally, not a session
- Bound to one requestor (by fingerprint), one resource, specific scopes, and a validity window
- The resource owner's signature is the authority — the token IS the permission
- Format-agnostic: the protocol defines required fields, not the encoding
  - {>> JSON for simplicity, Biscuit for embedded policy, SSH certificates for ecosystem fit}

### Scope Model

- Scopes are domain-specific — each resource defines its own vocabulary
- No central scope registry, no universal scope language
- The resource advertises what it offers; the requestor chooses from that set
- Scopes carry structure: resource + action + optional constraints
- The protocol treats scopes as opaque — interpretation belongs to the resource

### Temporal Grants

- Every grant has a TTL — no permanent permissions exist in the protocol
- Short TTLs (minutes to hours) are the expected pattern
- Long-lived grants are an explicit anti-pattern — permission is inherently temporal
- When a grant expires, the requestor re-requests — this forces re-evaluation

### Revocation

- Pre-TTL revocation is a registry concern, following the same model as EdProof identity revocation
- The resource consults zero or more revocation registries at verification time
- Registry unavailability is a policy input, not a verification failure
- Short TTLs reduce the revocation window — most grants expire before revocation is needed

### Resource Owner Sovereignty

- The resource owner owns every decision about granting access
- What evidence the resource owner requires is policy, not protocol:
  - May require bot owner co-approval
  - May require human review for certain scopes
  - May auto-approve low-risk scopes below a TTL threshold
  - May reject unknown requestors entirely
  - May require any other evidence or condition
- The protocol has two entities: requestor and resource owner. Any additional approval chain is the resource owner's policy concern

### Legacy Resource Adaptation

- Resources that don't support EdGrant natively use a sidecar proxy
- The sidecar is the verifier — it holds the real resource credentials and proxies authorized operations
- The sidecar is the resource owner's implementation concern, not a protocol element
- The grant model is identical whether the verifier is native or a sidecar

## Design

### Composition, Not Extension

EdGrant composes with EdProof. Neither protocol knows the other's internals. Identity (EdProof) + permission (EdGrant) = authenticated, authorized access.

### Canonical Serialization

- All signed payloads MUST be serialized in a canonical form: sorted keys, no whitespace, UTF-8, no trailing newline
- Strict subset of JCS (RFC 8785) — implementations MAY use a JCS library
- Without this, different SDKs produce different byte sequences for the same logical payload, breaking cross-implementation signature verification
- {>> especially critical in agent ecosystems where multiple tool chains serialize the same intent}

### Replay Protection

- Every request carries a `request_id` (UUID v4) and `created_at` timestamp, both covered by the signature
- Resource owner rejects duplicate request IDs and requests older than a freshness window (recommended: 5 minutes)
- Duplicate requests within the freshness window return the same grant (idempotent issuance) — making agent retries safe
- {>> agents and HTTP clients retry by default — without this, intercepted or retried requests cause duplicate grants}

### Signature Namespaces

- Signatures use the sshsig wire format with protocol-specific namespaces
- `edgrant-request` for request signatures, `edgrant-grant` for grant signatures
- Namespace separation prevents cross-phase and cross-protocol signature replay
- {>> same convention as EdProof's `edproof` namespace — consistent ecosystem}

### Formal Verification

- Grant exchange modeled in Tamarin prover under Dolev-Yao adversary
- Seven machine-checked properties: request authenticity, no scope escalation, bot binding, no request forgery, grant secrecy, grant authorization, executability (non-vacuity)
- Phase 0 (discovery) not modeled — no security-critical state
- Bot owner approval not modeled — it's a policy concern, not a protocol invariant

### Prior Art Positioning

- Combines request-initiation (from UMA), decentralisation (from UCAN), key-bound tokens (from GNAP), structured scopes (from OAuth RAR)
- Novel contributions: discovery-first flow, clean identity/permission separation, formally verified grant exchange, git-native audit trail option
- Attenuation principle borrowed from UCAN/Biscuit: permissions can only narrow, never widen

## Interactions

- **Requires** EdProof identity — every requestor and resource owner MUST have an EdProof credential
- **Requires** sshsig — signing convention for requests and grants
- **Requires** TLS — grant delivery encryption (Phase 2 → requestor)
- **Produces** capability tokens — consumed by resources at access time
- **Consumes** revocation registries — optional, at verification time
- **Enables** use-case-specific scope vocabularies (Jira, Grafana, CI/CD, data stores, APIs)

## Mapping

> [[RFC-EDGRANT.md]]
> [[formal/edgrant.spthy]]
> [[eidos/spec - formal verification model.md]]
> [[eidos/spec - permission grant use cases.md]]
> [[eidos/spec - sidecar adaptation pattern.md]]
> [[brainstorm - 2603271651 - edproof permission grants for bots and services.md]]
> [[research - 2603271704 - prior art for permission grant protocol.md]]
