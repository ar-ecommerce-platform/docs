# 2. HS256 shared-secret JWT for v1; RS256 + JWKS later

**Status:** Accepted

## Context

`auth-service` issues the token; `api-gateway` validates it. Two options:

- **HS256** — one shared secret. Simple, but the secret must be distributed to every validator,
  and a leak lets an attacker mint tokens.
- **RS256 + JWKS** — `auth-service` signs with a private key and publishes the public key at a
  JWKS endpoint; validators fetch it. This is exactly how Microsoft Entra ID works, so the
  gateway would validate local and Entra tokens through the same mechanism.

## Decision

Ship v1 with **HS256** and a shared `JWT_SECRET` (injected from the environment, dev fallback in
config). Pin the algorithm explicitly to HS256 so it doesn't vary with secret length. The
gateway's multi-issuer resolver already treats the local issuer and Entra as separate trust
anchors, so moving the local path to RS256 later is contained.

## Consequences

- Fast to build and reason about; good enough for a single-tenant demo.
- The secret is shared between `auth-service` and `api-gateway` via the same env var — acceptable
  because both are ours and it's rotated as one unit.
- **Follow-up:** `auth-service` signs with RS256 and exposes `/oauth2/jwks`; the gateway drops
  the shared secret and validates every issuer by JWKS. Tracked in the roadmap.
