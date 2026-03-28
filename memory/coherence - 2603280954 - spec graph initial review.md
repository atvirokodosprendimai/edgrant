---
tldr: Coherence report — initial spec graph review across edgrant protocol spec and use cases
category: core
---

# Coherence: Initial Spec Graph Review

## Contradictions

### 1. Entity naming: "bot" vs "requestor"

`spec - permission grant use cases.md` uses "bot" and "bot owner" throughout as primary entities. `eidos/spec - edgrant protocol.md` and `RFC-EDGRANT.md` use "requestor" — deliberately broadened to include bots, agents, services, and humans.

The use cases doc was written before the RFC generalised the terminology. The use cases are still accurate (bots are requestors) but the framing is narrower than the protocol allows.

**Severity:** Low — not a logical contradiction, but a terminology drift that could mislead readers into thinking the protocol is bot-only.

**Resolved:** Updated use cases prerequisites to use "requestor" for protocol-level language, added note that "bot" is used as the concrete requestor in use cases. Added terminology section to CLAUDE.md.

### 2. Approval flow: dual-approval in use cases vs two-entity protocol

`spec - permission grant use cases.md` Phase 2 diagram (line 55-61) shows bot owner as a distinct participant in the approval flow, receiving the request and forwarding it to the resource owner. This implies a three-entity protocol.

`eidos/spec - edgrant protocol.md` states: "The protocol has two entities: requestor and resource owner. Any additional approval chain is the resource owner's policy concern."

The use cases doc's own note (line 108) acknowledges this: "bot owner approval... is NOT a protocol requirement." But the diagram contradicts this note by showing bot owner as a protocol-level step.

**Severity:** Medium — the diagram structurally shows a three-entity flow while the text and the protocol spec say two. Readers will see the diagram first.

**Resolved:** Replaced three-entity diagram with two-entity diagram matching RFC-EDGRANT §3. Bot owner approval shown as policy annotation inside Phase 2, not as a protocol step.

### 3. Stale RFC-opportunity note

`spec - permission grant use cases.md` line 104: "The permission request → approval → grant → access flow described in these use cases could become its own RFC (e.g., RFC-EDGRANT)."

This has now happened — RFC-EDGRANT.md exists. The note is stale.

**Severity:** Low — informational, not contradictory.

**Resolved:** Replaced with "RFC status" note linking to RFC-EDGRANT.md.

## Orphaned Links

### 4. `[[RFC-EDPROOF.md]]` in use cases (line 506) — file not found

The EdProof RFC is in a separate repository. This link cannot resolve within this project.

**Resolved:** Replaced with `[[RFC-EDGRANT.md]]` — the correct RFC for this repo.

### 5. `[[eidos/spec - edproof protocol.md]]` in use cases (line 507) — file not found

No EdProof protocol spec exists in this repo's eidos directory. This is an EdGrant-focused repo.

**Resolved:** Replaced with `[[eidos/spec - edgrant protocol.md]]`.

### 6. `[[memory/brainstorm - 2603271651 - edproof permission grants for bots and services.md]]` in use cases (line 508) — wrong path

The brainstorm file exists at the project root, not in `memory/`. Correct path: `[[brainstorm - 2603271651 - edproof permission grants for bots and services.md]]`.

**Resolved:** Fixed path to `[[brainstorm - 2603271651 - edproof permission grants for bots and services.md]]`.

## Missing Cross-References

### 7. Use cases doc does not link to `eidos/spec - edgrant protocol.md`

The eidos spec links to the use cases doc in its Mapping section. The use cases doc links to `[[eidos/spec - edproof protocol.md]]` (which doesn't exist) but not to the actual EdGrant eidos spec.

**Resolved:** Use cases now links to `[[eidos/spec - edgrant protocol.md]]` (fixed as part of #5).

### 8. Use cases doc does not link to `RFC-EDGRANT.md`

The use cases doc references `[[RFC-EDPROOF.md]]` (external) but not the RFC that was derived from these very use cases.

**Resolved:** Use cases now links to `[[RFC-EDGRANT.md]]` (fixed as part of #4).

## CLAUDE.md Candidates

### 9. "Resource owner owns the decision"

Appears in:
- `eidos/spec - edgrant protocol.md` — Resource Owner Sovereignty section
- `spec - permission grant use cases.md` — Cross-Cutting Principles #3, #8
- `RFC-EDGRANT.md` — Design Principle #2, §6.1

This is a foundational project principle that governs all design decisions. Candidate for CLAUDE.md as a design constraint.

### 10. "Two entities: requestor and resource owner"

Appears in:
- `eidos/spec - edgrant protocol.md` — Resource Owner Sovereignty section
- `RFC-EDGRANT.md` — Abstract, §2 Terminology
- `spec - permission grant use cases.md` — Prerequisites (as "bot and resource owner")
- `formal/edgrant.spthy` — header comment

Core protocol invariant. Candidate for CLAUDE.md to prevent accidental introduction of third protocol entities.

**Resolved:** Both candidates added to CLAUDE.md as "Protocol Invariants" with a "Terminology" section.

## Summary

- 2 specs analysed (+ RFC, Tamarin model, brainstorm, research as supporting docs)
- 2 contradictions found (1 low, 1 medium)
- 3 orphaned links found
- 2 missing cross-references found
- 2 CLAUDE.md candidates identified
