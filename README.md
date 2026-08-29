# docs

Architecture and design notes for the [ar-ecommerce-platform](https://github.com/ar-ecommerce-platform) —
a Java 21 / Spring Boot microservices e-commerce backend built to practise distributed-systems
design and DevOps.

- **[architecture.md](architecture.md)** — context, container and component views; the order
  placement sequence; tech stack; what's deliberately deferred.
- **[decisions/](decisions/)** — Architecture Decision Records (ADRs): the *why* behind the
  choices that would otherwise look arbitrary.

## The system in one paragraph

Clients call a single **API gateway** (`:8080`), which validates a JWT and routes `/api/**` to
one of seven business services via **Eureka** service discovery + client-side load balancing.
Auth is issued by **auth-service** (local accounts) and can also accept **Microsoft Entra ID**
tokens. **order-service** is the orchestrator: placing an order fans out over HTTP to
product-, inventory- and payment-service and then notification-service. Each service owns its
own database. **config-server** serves shared configuration. Everything runs locally with one
`docker compose up`; the target is a hosted demo on Azure or AWS.

## Repos

| Group | Repos |
|---|---|
| Edge / platform | `api-gateway`, `discovery-server`, `config-server`, `config-repo` |
| Business services | `auth-service`, `user-service`, `product-service`, `inventory-service`, `order-service`, `payment-service`, `notification-service` |
| Shared | `common` (JWT helper) |
| Ops & tests | `infra` (compose, scripts, runbook), `e2e-tests` (REST Assured), `docs` (this repo) |
