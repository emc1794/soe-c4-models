# ADR 0001: Adopt RabbitMQ as the external message broker

- **Session:** 4 — Enterprise Integration & Messaging
- **Status:** Accepted

## Context

Through Session 3 the backend is a modular monolith with clear domain-oriented modules
(Identity, Events, Ordering, Payments, Notifications) communicating via direct, in-process calls.
Session 4's objective is to model domain events and move cross-module integration to an
event-driven style, first via an in-memory event bus, then via an external broker.

## Problem

In-process events don't survive a process restart and don't generalize to a multi-process
deployment. Session 6 will extract the Ordering module into its own deployment unit, which
requires an integration mechanism that already works across process boundaries.

## Decision

Introduce RabbitMQ (AMQP) as the message broker, replacing the in-memory event bus. Domain
modules publish and consume events (`OrderCreated`, `PaymentSuccessful`, `PaymentFailed`,
`EventCancelled`, `EventUpdated`, ...) through it instead of calling each other directly.

## Alternatives considered

- **Kafka** — rejected. The kata's fixed technology constraints specify RabbitMQ, and the
  workload (moderate-volume domain events, no log replay or high-throughput streaming needs)
  doesn't justify a log-based streaming platform.
- **Keep the in-memory event bus** — rejected. Doesn't survive restarts and doesn't generalize
  to the multi-process deployment Session 6 requires.

## Consequences

- Adds an operational dependency (RabbitMQ) and requires designing idempotent consumers.
- Enables eventual consistency and loose coupling between domain modules ahead of the Session 6
  extraction.
- Still a modular monolith at this point — no distributed deployment yet, only the integration
  style changes.

## Trade-offs

Eventual consistency and an extra moving part to operate, in exchange for decoupling and
durability that in-process calls and an in-memory bus can't provide.
