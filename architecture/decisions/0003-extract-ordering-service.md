# ADR 0003: Extract the Ordering (Ticketing) bounded context into its own service and database

- **Session:** 6 — Microservices Deep Dive
- **Status:** Accepted

## Context

Through Session 5 the backend is a modular monolith with domain-oriented modules and an
event-driven purchase saga (ADR 0001, ADR 0002). Session 6's objective is to extract exactly one
bounded context into an independent deployment unit.

## Problem

Ordering/Ticketing carries seat allocation, general admission, assigned seating, ticket
inventory, and duplicate-sale prevention — the system's highest-contention workload, and the one
most likely to need independent scaling during an on-sale spike. Coupling it to the same
deployment unit and database as Identity, Events, Payments, and Notifications means it can't scale
or deploy independently, and a spike in Ordering traffic risks degrading unrelated modules.

## Decision

Extract Ordering into an independently deployable **Ordering Service** with its own dedicated
MySQL database (**Ordering DB**). The remaining **Core Monolith** keeps Identity, Events,
Payments, and Notifications with its own **Monolith DB**. No tables are shared between the two
databases; the services communicate only through RabbitMQ events (see ADR 0002 for how the saga
had to change as a result) and, from Session 7 on, through the API Gateway (ADR 0004).

## Alternatives considered

- **Extract Events instead of Ordering** — rejected. The kata explicitly calls out Ordering's
  contention profile (seat locking, duplicate-sale prevention) as the sharper scaling problem.
- **Extract Ordering but keep the shared database** — rejected. A shared database across two
  independently deployed services creates a distributed monolith: schema changes require
  coordinating two deployments, and it undermines the isolation the extraction is meant to buy.

## Consequences

- Ordering can now scale and deploy independently of the rest of the system.
- The purchase saga can no longer be a single in-process orchestrator (ADR 0002).
- Cross-service consistency becomes eventual; this is later addressed for read-heavy queries by
  the CQRS read model (ADR 0005).

## Trade-offs

Two services and two databases add operational complexity (deployment, monitoring, network calls)
compared to one monolith and one database — accepted in exchange for isolating the system's
highest-contention workload so it can scale and fail independently of the rest of the platform.
