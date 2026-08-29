# 3. order-service orchestrates synchronously, with independent transactions

**Status:** Accepted

## Context

Placing an order touches four other services: price the lines (product), reserve stock
(inventory), take payment (payment), notify (notification). Design axes:

1. **Orchestration vs choreography** — one service drives the flow, or services react to events
   on a broker.
2. **Transaction boundary** — wrap the flow in one `@Transactional`, or commit each step
   separately.

## Decision

- **Synchronous orchestration.** `order-service` calls the others over HTTP (through Eureka +
  client-side load balancing). Easy to follow, debug and demo; no broker to run.
- **Independent short transactions.** `OrderTransactions` exposes `createPending` /
  `markRejectedStock` / `markPaymentFailed` / `confirm`, each its own transaction. `place()` is
  not transactional.

## Consequences

- A single `@Transactional` around `place()` was tried and rejected: it held a DB connection
  open across the external HTTP calls, and it rolled back the `REJECTED_STOCK` / `PAYMENT_FAILED`
  audit row when the method threw. A Testcontainers test against real PostgreSQL caught this.
- The order status is a real audit trail — a rejected order is persisted, not lost.
- **Known gap:** no compensation. If line 1 reserves stock and line 2 fails, line 1's
  reservation is not released. Fixing this (a saga with compensating actions, or moving stock
  reservation to a two-phase "reserve then commit") is a roadmap item.
- **Known gap:** no retry / circuit breaker. Resilience4j around the client calls is a roadmap
  item.
- Event-driven notifications (publish `OrderConfirmed`, let notification-service consume) is the
  first planned move toward choreography.
