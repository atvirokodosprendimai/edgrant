---
tldr: Protocol review identifying 5 precision gaps for agent-first consumption — canonical serialization, replay resistance, discovery boundaries, scope semantics, enforcement profile
category: core
---

# Review: Agent-First Precision Gaps

**Date:** 2026-03-28
**Verdict:** Good idea, right abstraction, real problem — but the spec needs sharper edges before agents can be trusted to eat it safely.

## What EdGrant Gets Right

- **Addresses a real failure mode**: fine-grained enterprise permissions are so tedious that users fall back to broad, long-lived credentials. EdGrant replaces that with request-driven, resource-scoped, time-bounded delegation
- **The model is right**: agent discovers → requests → owner decides. Closer to sharing a Google Doc than minting a service credential
- **Correct abstraction boundary**: backend-specific enforcement belongs on the resource-owner side. Protocol stays generic and portable
- **Sidecar is transitional**: a compatibility crutch, not core architecture. If platforms natively supported EdGrant-style delegation, sidecars disappear. That's a strength

## Five Findings

### 1. Canonical Serialization — HIGH

If signed content is not canonically defined, implementations will drift. Different SDKs and wrappers will serialize the same intent differently, and agent ecosystems multiply that problem fast. Exact canonical serialization needs to be defined, not implied.

**Resolved:** Added RFC §5.3 Canonical Serialization — sorted keys, no whitespace, UTF-8, no trailing newline, array order preserved. Defined as strict subset of JCS (RFC 8785). Updated eidos protocol spec.

### 2. Replay and Retry Resistance — HIGH

Agents and tool chains retry by default. Access requests need built-in replay resistance: request IDs, timestamps, and tight expiration semantics. Otherwise approval flows and grant issuance will get weird fast.

**Resolved:** Added `request_id` (UUID v4) and `created_at` to request format. Added RFC §10.4 Replay Protection — request ID deduplication, 5-minute freshness window, idempotent grant issuance for retries. Both fields covered by signature. Updated eidos protocol spec.

### 3. Discovery Boundaries — MEDIUM

Discovery is a great usability feature, but for agents it is also an escalation slope. If the system tells an agent what scopes are available and how to request them, the agent may keep asking until something works. Discovery needs policy boundaries, not just convenience.

**Status:** Open
**Action:** Define rate limiting on discovery, scope visibility controls (resource owner may hide scopes from certain requestors), and request attempt limits.

### 4. Scope Semantics — MEDIUM

Scope names and capabilities need to be tight and machine-friendly. If they are vague, agents will guess. And guessed permissions are how systems quietly rot.

**Status:** Open
**Action:** Define scope naming conventions, recommend machine-parseable scope structures (not just opaque strings), provide guidance on scope granularity.

### 5. Enforcement Profile — MEDIUM

The core RFC should stay small, but there should be a companion document defining what a resource-owner adapter must do: default deny, exact scope-to-operation mapping, no broad fallback, strict expiry propagation, clear handling of unsupported scopes, and full audit trace from request to backend action. Without that, the protocol can be elegant while deployments turn sloppy.

**Status:** Open
**Action:** Write a companion spec (or eidos spec) for the enforcement profile — the minimum security contract an adapter must satisfy.

## Summary

| # | Finding | Severity | Status |
|---|---------|----------|--------|
| 1 | Canonical serialization | HIGH | **Resolved** |
| 2 | Replay/retry resistance | HIGH | **Resolved** |
| 3 | Discovery boundaries | MEDIUM | Open |
| 4 | Scope semantics | MEDIUM | Open |
| 5 | Enforcement profile | MEDIUM | Open |
