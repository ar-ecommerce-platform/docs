# 6. The gateway asserts caller identity; services scope data to it

**Status:** Accepted

## Context

The gateway verified tokens, but services never learned *who* was calling. Any signed-in user
could read any order (`GET /api/orders?userId=anyone`), list every user profile, read any
payment, and place orders in someone else's name (`userId` came from the request body). Every
internal endpoint (payment authorization, stock reservation, notification writes) was also
reachable from outside.

## Decision

- **Identity header.** After authentication the gateway sets `X-User-Id` to the verified token's
  subject and strips any `X-User-Id` the client sent. Services read the caller only from this
  header, never from a body field or query parameter.
- **Default-deny API surface.** The gateway security config is an allow-list of public routes:
  auth, catalog reads, `/api/orders/**`, `GET /api/notifications`, `GET /api/users/me`. Everything
  else (payments, stock reservation, notification writes, the user directory) is denied at the
  edge and stays service-to-service only.
- **Not found, not forbidden.** Reading another user's order returns 404, so callers cannot probe
  which ids exist.

## Consequences

- Services trust the header, which is only safe while they are reachable **solely through the
  gateway**: no public ports locally (bound to 127.0.0.1) and private subnets in the cloud. A
  service exposed directly would accept a forged header.
- Services stay free of JWT/crypto config; only the gateway holds `JWT_SECRET`.
- For local tokens the subject is the email. Microsoft Entra tokens use an opaque subject, so a
  user signing in both ways has two identities. Acceptable for v1.
- Roles are not forwarded yet: there is no admin surface. Add `X-User-Roles` the same way when one
  exists.
- Clients lost `userId` parameters and `GET /api/payments/{id}`; the order carries `paymentId`
  and status.
