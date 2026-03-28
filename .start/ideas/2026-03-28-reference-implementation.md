# Reference Implementation — EdGrant Protocol Proof-of-Concept

**Date:** 2026-03-28
**Purpose:** Prove the protocol works end-to-end with standard UNIX tools

## Approach

Shell scripts + ssh-keygen + local HTTP server. Zero dependencies beyond OpenSSH and Python (for http.server). Proves EdGrant is implementable with nothing but standard tools.

## Architecture

Three scripts in `ref/`:

### `edgrant-setup`
- Generates Ed25519 key pairs for requestor and resource owner
- Creates a demo workspace directory

### `edgrant-resource`
- Runs a minimal HTTP server (Python http.server or socat) on :8400
- Phase 0: Returns 401 + requirements JSON on unauthenticated requests
- Phase 1: Receives signed grant requests, verifies sshsig signature (namespace: edgrant-request)
- Phase 2: Issues capability token signed with resource owner's key (namespace: edgrant-grant)
- Phase 3: Receives access requests with credential + token, verifies both, serves scoped response

### `edgrant-request`
- CLI that walks through the requestor side
- `discover` — hits resource, parses 401 requirements
- `grant` — constructs and signs grant request, receives capability token
- `access` — makes access request with identity + token

## Demo Flow

```
$ ./edgrant-setup
$ ./edgrant-resource &
$ ./edgrant-request discover :8400
$ ./edgrant-request grant :8400 read 1h
$ ./edgrant-request access :8400
$ ./edgrant-request access :8400 write   # → 403
```

## Scope

### In
- All four phases (discovery, request, grant, access)
- sshsig signing with correct namespaces (edgrant-request, edgrant-grant)
- JSON message formats matching the RFC
- TTL checking at access time
- Scope verification at access time
- Signature verification with ssh-keygen -Y verify

### Out
- TLS (localhost HTTP — proves protocol, not transport)
- Revocation registries
- Real legacy resource integration
- Persistent state
- Production error handling

## Technical Decisions

- **Python http.server** for the resource side (available everywhere, simple request routing)
- **ssh-keygen -Y sign/verify** for all signatures (sshsig wire format, namespace separation)
- **jq** for JSON construction and parsing
- **curl** for HTTP requests from requestor
- **No external dependencies** beyond standard UNIX tools + OpenSSH

## Parking Lot

(empty)
