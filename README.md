# EdGrant

Permission grant protocol for EdProof identities.

Scoped, time-limited, auditable access grants. Built on
[EdProof](https://github.com/atvirokodosprendimai/edproof) identity.

## How It Works

A bot discovers what a resource accepts, requests specific permissions
for a specific duration, and the resource owner decides. The bot carries
the grant as a capability token — verified locally, no issuer contacted.

EdProof is the passport. EdGrant is the visa.

## Documents

- [`RFC-EDGRANT.md`](RFC-EDGRANT.md) — full protocol specification (v0.1)
- [`formal/edgrant.spthy`](formal/edgrant.spthy) — Tamarin prover model

## Formal Verification

All seven security lemmas machine-checked in 0.66s:

```
request_authenticity  verified   grant traces back to genuine bot request
no_scope_escalation   verified   granted scopes match requested scopes
bot_binding           verified   grant bound to exactly one bot key
no_request_forgery    verified   attacker cannot forge bot's request
grant_secrecy         verified   attacker cannot learn grant token
grant_authorization   verified   only resource owner can issue grants
executability         verified   protocol can complete (non-vacuity)
```

```bash
tamarin-prover --prove formal/edgrant.spthy
```

Requires [Tamarin prover](https://tamarin-prover.com/) ≥ 1.10 and Maude ≥ 3.5.

## Design Principles

- **Discovery before request** — the bot learns what the resource accepts by trying
- **Resource owner decides** — approval chains are policy, not protocol
- **Least privilege** — specific scopes, specific TTL, no permanent grants
- **Token is a document** — verified locally, resource owner not contacted at access time
- **Scopes are domain-specific** — no central scope registry

## Prior Art

Built on research into UCAN, Biscuit, GNAP (RFC 9635), UMA 2.0,
OAuth RAR (RFC 9396), Macaroons, and ZCAP-LD. See
[prior art analysis](research%20-%202603271704%20-%20prior%20art%20for%20permission%20grant%20protocol.md).

## Requires

- [EdProof](https://github.com/atvirokodosprendimai/edproof) — identity protocol (Ed25519, five-layer model)
