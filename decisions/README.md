# Architecture Decision Records

Short notes on decisions that would otherwise look arbitrary. Format: Status / Context /
Decision / Consequences. One file per decision, numbered, never deleted — superseded ones are
marked as such.

| # | Decision | Status |
|---|---|---|
| [0001](0001-vendor-build-quality-config.md) | Vendor the Gradle quality config per repo instead of `apply from:` a remote URL | Accepted |
| [0002](0002-hs256-jwt-for-v1.md) | HS256 shared-secret JWT for v1; RS256 + JWKS later | Accepted |
| [0003](0003-synchronous-orchestration.md) | order-service orchestrates synchronously with independent transactions | Accepted |
| [0004](0004-config-server-not-load-bearing.md) | config-server is demonstrable but not load-bearing | Accepted |
| [0005](0005-flyway-owns-the-schema.md) | Flyway owns the `prod` schema; Hibernate only validates | Accepted |
