---
tldr: Sidecar proxy pattern for adapting legacy resources to EdGrant — credential mapping, scope translation, access proxying
category: pattern
---

# Sidecar Adaptation Pattern

## Target

How resource owners adapt legacy resources (Jira, Grafana, APIs with their own auth) to the EdGrant protocol without modifying the resource itself. The sidecar is an adapter — it speaks EdGrant on one side and the resource's native API on the other.

## Behaviour

### What a Sidecar Does

- Sits between the requestor and the legacy resource as a reverse proxy
- Implements all four EdGrant phases on the requestor-facing side (discovery, request handling, grant issuance, access verification)
- Holds the resource's native credentials (API token, service account, OAuth client) with broad permissions
- Translates EdGrant scope checks into native API filtering — proxies only operations within the granted scopes
- Rejects operations outside the granted scopes before they reach the resource
- Logs every proxied operation for the audit trail

### What a Sidecar Is Not

- Not a protocol element — the protocol is identical with or without a sidecar
- Not a shared service — each resource owner operates their own sidecar for their own resources
- Not an authorization server — the sidecar IS the verifier, applying the resource owner's policy directly
- Not required for EdGrant-native resources — only for legacy systems that can't verify capability tokens themselves

### Credential Mapping

- The sidecar holds a single privileged credential for the legacy resource (e.g. Jira API token with project admin access)
- Requestors never see or receive the native credential — they interact only via EdGrant
- The sidecar's native credential should have the broadest permissions any requestor might need — scope restriction happens at the sidecar level, not the native credential level
- If the native credential is compromised, only the sidecar is affected — requestors' EdGrant tokens remain valid but unusable without the sidecar

### Scope Translation

- The sidecar defines the scope vocabulary for its resource (advertised in discovery)
- When a requestor presents a capability token with specific scopes, the sidecar maps those to native API operations
- Example: `read` on a Jira board → sidecar allows GET requests to board/issue endpoints, rejects POST/PUT/DELETE
- Example: `write:filter` → sidecar allows filter creation endpoints, rejects task modification endpoints
- Example: `read:dashboard` on Grafana → sidecar proxies dashboard GET requests via Grafana service account
- Scope translation is resource-type-specific — each sidecar implementation defines its own mapping

### Rate Limiting and Constraints

- The sidecar enforces constraints from the capability token (e.g. `max_queries_per_minute: 60`)
- The sidecar MAY impose additional constraints beyond what the token specifies (as part of resource owner policy)
- Rate limits protect both the legacy resource and the sidecar's native credential from abuse

### Audit Trail

- The sidecar logs at two levels:
  - **Grant lifecycle** — request received, policy evaluated, grant issued, grant expired/revoked
  - **Access operations** — every proxied API call with: who (requestor fingerprint), what (operation), when, result
- The sidecar's audit log is the resource owner's record — it shows what actually happened at the native API level, not just what was granted

## Design

### The Sidecar is the Verifier

In EdGrant/EdProof terms, the sidecar occupies the verifier role. It:
- Verifies the requestor's EdProof credential (identity)
- Verifies the capability token (permission)
- Applies its own policy (Layer 4 sovereignty)
- Owns the decision to grant or deny each operation

This is consistent with EdProof's "the verifier owns the decision" principle — the sidecar is the resource owner's agent.

### Deployment Models

- **Reverse proxy** — sidecar runs as a standalone service in front of the resource, all traffic passes through it
- **API gateway plugin** — sidecar logic embedded in an existing API gateway (e.g. Envoy, Kong, Nginx)
- **Library/middleware** — sidecar logic embedded in the resource's application layer (if the resource owner controls the code)

The protocol doesn't prescribe a deployment model. The sidecar is an implementation concern.

### Security Boundary

- The sidecar's native credential is the highest-value secret — if compromised, an attacker has the sidecar's full access to the legacy resource
- The sidecar SHOULD run in an isolated environment with minimal attack surface
- The native credential SHOULD be rotated regularly and scoped as narrowly as practical
- {>> for high-security resources, consider a sidecar that requests short-lived native credentials on demand rather than holding a long-lived token}

## Interactions

- **Implements** the legacy resource adaptation described in [[eidos/spec - edgrant protocol.md]] §Legacy Resource Adaptation
- **Instantiated by** the use cases in [[eidos/spec - permission grant use cases.md]] (Jira/Gyros, Grafana, API gateways)
- **Requires** EdProof credential verification and EdGrant token verification
- **Holds** native resource credentials (API tokens, service accounts)

## Mapping

> [[RFC-EDGRANT.md]] §12
> [[eidos/spec - edgrant protocol.md]]
> [[eidos/spec - permission grant use cases.md]]
