# Pull — EdGrant Formal Verification Model

**Date:** 2026-03-28
**Source:** formal/edgrant.spthy

## Collected Material

### Theory Structure

Single Tamarin theory `EdGrantPermission` with builtins `signing` and `symmetric-encryption`. One custom function `fingerprint/1` modelling SHA-256 (injective in free term algebra — collision resistance assumption).

### Key Infrastructure

Two protocol entities, each with an Ed25519 key pair:
- **Bot** (requestor): `Bot_KeyGen` generates `~sk_bot`, publishes `pk(~sk_bot)`
- **Resource Owner**: `ResourceOwner_KeyGen` generates `~sk_ro`, publishes `pk(~sk_ro)`

Bot Owner is explicitly NOT a protocol entity — stated in comments and structurally absent from rules.

### Protocol Rules (3 rules)

**Bot_Request** (Phase 1):
- Bot generates fresh `~resource`, `~scopes`, `~ttl`, `~tls_key`
- Signs `<'grant_request', ~resource, ~scopes, ~ttl>` with `~sk_bot`
- Publishes: fingerprint, resource, scopes, ttl, signature, pk_bot
- Retains `~tls_key` in state (never on public channel) for TLS model
- Action fact: `BotRequested(pk_bot, ~resource, ~scopes, ~ttl)`

**ResourceOwner_Grant** (Phase 2):
- Receives bot's message from public channel
- Consumes `TLSGrant` state fact (models TLS session)
- Verifies bot's signature via restriction `VerifyBotRequestSig`
- Generates fresh `~grant_token`
- Signs `<'resource_owner_grant', bot_fp, resource, scopes, ttl>` with `~sk_ro`
- Encrypts grant with `tls_key`: `senc(<...>, tls_key)`
- Action facts: `ResourceOwnerGranted`, `GrantSecret`, `GrantBound`

**Bot_ReceiveGrant** (Phase 3):
- Decrypts using `~tls_key` from state (only bot can decrypt)
- Action facts: `BotReceivedGrant`, `BotHasGrant`

### Modelling Decisions

1. **Phase 0 excluded** — discovery has no security-critical state (no secrets, no signatures). Correct: discovery is informational only
2. **TLS modelled as symmetric encryption** — `~tls_key` is fresh, never published, shared via state fact `TLSGrant`. This is an idealised TLS model (no TLS handshake modelled)
3. **Policy abstracted as precondition** — if `ResourceOwner_Grant` fires, policy was satisfied. What policy contains is out of scope. This cleanly separates protocol from policy
4. **Fingerprint as injective function** — models SHA-256 collision resistance in the free term algebra. Standard Tamarin approach
5. **Namespace constants as string tags** — `'grant_request'` and `'resource_owner_grant'` prevent cross-phase signature replay at the algebraic level
6. **Resource/scopes/ttl as fresh values** — models arbitrary choices (any resource, any scope set, any TTL). Tamarin explores all possible values

### Restriction

`VerifyBotRequestSig`: enforces `verify(req_sig, req_msg, pk_bot) = true`. Models the resource owner actually checking the bot's signature.

### Seven Lemmas

| # | Name | Property | Proof basis |
|---|------|----------|-------------|
| 1 | `request_authenticity` | Grant traces back to genuine bot request | Bot's signature covers (resource, scopes, ttl); only sk_bot can produce it |
| 2 | `no_scope_escalation` | Granted scopes match requested scopes | Both signatures cover same (resource, scopes, ttl) tuple |
| 3 | `bot_binding` | Grant bound to exactly one bot key | Injectivity of fingerprint/1 (collision resistance) |
| 4 | `no_request_forgery` | Attacker cannot forge request | Same as #1, stated from forgery perspective |
| 5 | `grant_secrecy` | Attacker cannot learn grant token | TLS model: senc with fresh key never on public channel |
| 6 | `grant_authorization` | Only resource owner can issue grants | TLS state fact consumed by ResourceOwner_Grant; grant contains RO signature |
| 7 | `executability` | Protocol can complete (non-vacuity) | exists-trace — confirms lemmas 1-6 aren't vacuously true |

All verified in 0.66s.

## Patterns

- **Dolev-Yao adversary** — attacker controls the network (standard symbolic model)
- **State facts for session isolation** — `BotState`, `TLSGrant` prevent cross-session confusion
- **Action facts as trace events** — security properties stated as trace formulas over action facts
- **Restriction for verification** — models the verify() call as a global constraint

## Dependencies

- Tamarin prover ≥ 1.10
- Maude ≥ 3.5
- EdProof identity assumed (not re-verified — see edproof.spthy in separate repo)

## Intent Sketch

- The formal model proves that the grant exchange is secure under a network-controlling adversary
- What's modelled: the three security-critical phases (request, grant, access) and the TLS delivery channel
- What's excluded by design: discovery (informational only), policy evaluation (resource owner's concern), specific scope semantics (opaque to protocol)
- The model abstracts TLS as symmetric encryption with a pre-shared fresh key — this is an idealisation that assumes TLS itself is secure
- Lemmas 1 and 4 prove the same property from different angles (authenticity vs forgery) — this is intentional for clarity
- The executability lemma is critical: without it, the other six could be vacuously true (true because no trace exists)
- The collision resistance assumption (injective fingerprint) is standard but should be noted — if SHA-256 collisions become practical, bot binding breaks
