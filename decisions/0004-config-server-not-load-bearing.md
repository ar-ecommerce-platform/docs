# 4. config-server is demonstrable but not load-bearing

**Status:** Accepted

## Context

Spring Cloud Config Server centralises configuration. The original setup had `config-server`
clone `config-repo` from GitHub on startup with `fail-fast: true`, and services depended on it
for basic settings like their port. That makes `config-server` a single point of failure and a
startup-order landmine, and it needs network to GitHub.

## Decision

- `config-server` uses the **native** backend, reading a local directory (a checkout locally, a
  mounted volume in Docker) — no Git clone.
- Every service ships a **self-contained `application.yml`** with sensible defaults and imports
  from config-server as **`optional:configserver:...`**.
- Environment variables (from compose / the deployment) override anything served from
  config-server (they have higher precedence in Spring's config hierarchy).

## Consequences

- The platform starts and runs even if `config-server` is down or slow.
- `config-server` still registers in Eureka and serves `GET /{app}/{profile}`, so it's a real,
  demonstrable piece — just not a dependency.
- `config-repo` is a demonstration of centralised config management, not a control point. Real
  per-environment secrets never live there; they come from the environment (Key Vault / Secrets
  Manager in cloud).
