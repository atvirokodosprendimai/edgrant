---
tldr: Forward opportunities after eidos bootstrap merge — no open plans/todos/comments
category: core
---

# Next: Forward Opportunities After Merge

## Active Plans / Todos / Comments / Goodjobs

None.

## Forward Opportunities

1. **Spec the formal model** — formal/edgrant.spthy has no dedicated eidos spec. Its modelling decisions (what's in scope, what's abstracted, why Phase 0 is excluded) and the 7 verified properties deserve their own spec
2. **Reference implementation** — plan and build a minimal SDK or CLI that demonstrates the four-phase flow (discovery → request → grant → access)
3. **Push to remote** — 8 commits on main since initial, not yet pushed to origin
4. **Spec the sidecar pattern** — legacy resource adaptation is described briefly in the protocol spec but the sidecar architecture (proxy, credential mapping, scope translation) could be its own spec
5. **Spec revocation registries** — revocation is declared as "registry concern" but the registry format, update mechanism, and verification-time behaviour are underspecified
