# 5. Flyway owns the schema in `prod`; Hibernate only validates

**Status:** Accepted

## Context

Locally, services run on in-memory H2 with `hibernate.ddl-auto: update` — Hibernate creates the
schema from the entities on every boot. That's fine for a throwaway database but unacceptable for
a real one: no history, no review, no rollback, and `update` never drops or alters safely.

## Decision

- **Dev (H2):** keep `ddl-auto: update`, Flyway disabled. Fast iteration, no migration to write
  when an entity changes.
- **`prod` (PostgreSQL):** **Flyway** runs versioned SQL migrations from
  `src/main/resources/db/migration/V*.sql`; Hibernate is set to `ddl-auto: validate` — it only
  checks the entities match the schema Flyway built, and refuses to start on a mismatch.
- One logical database per service. The datasource comes entirely from the environment
  (`SPRING_DATASOURCE_*`).

## Consequences

- The schema is a reviewed, ordered set of SQL files — the same artifact a DBA would expect.
- `validate` turns entity/schema drift into a startup failure instead of silent breakage or a
  destructive auto-alter. (The order-service integration test already runs against real
  PostgreSQL via Testcontainers, so drift is caught in CI too.)
- Writing the first migration by hand is a one-off cost; subsequent entity changes need a new
  `V{n}__*.sql`. Acceptable, and it's the industry norm.
- Dev and `prod` now have genuinely different schema-management paths. The migrations are the
  source of truth; the H2 path is a convenience that could drift — mitigated by the JPA slice
  tests running on H2 and the integration test running on PostgreSQL.
