# ADR 0002: Saga for the purchase workflow — orchestrated in Session 5, choreographed from Session 6

- **Session:** 5 (introduced), revised Session 6 — Service-Based & Orchestrated Styles / Microservices
- **Status:** Accepted (revised)

## Context

The purchase workflow spans multiple domain modules — seat reservation (Events), fraud
validation, payment processing, and ticket issuance (Identity) — with compensating actions on
failure (release the seat if fraud check or payment fails). These modules have been separate,
independently reasoned-about bounded contexts since Session 3, so there is no single ACID
transaction available to coordinate the workflow.

## Problem

Need to coordinate a multi-step business transaction across bounded contexts, with well-defined
compensation when any step fails, without reintroducing tight coupling between modules.

## Decision

Model the purchase workflow as a Saga.

- **Session 5:** prefer orchestration. A `Booking Saga Orchestrator` component (still in-process,
  since the backend is one deployment unit) drives each step — reserve seat, validate fraud risk,
  process payment, issue ticket — and issues the seat-release compensation on any failure.
- **Session 6:** once the Ordering module is extracted into its own deployment unit (ADR 0003),
  a single in-process orchestrator can no longer coordinate steps that now live in two
  independently deployable services without making synchronous cross-service calls. Coordination
  becomes choreographed over RabbitMQ: the Ordering Service publishes `OrderCreated`; the
  Monolith's Fraud Check Module reacts first and gates the Payment Module, which charges the
  customer and publishes `PaymentProcessed`/`PaymentFailed` (or the Fraud Check Module itself
  publishes `PaymentFailed` on a fraud rejection); the Ordering Service's `Order Status Consumer`
  reacts to that outcome by completing the order or releasing the seat.

## Alternatives considered

- **Choreography from the start (Session 5)** — rejected for Session 5. With the whole workflow
  in one process, a single orchestrator is easier to follow and to extend with new compensation
  rules than a chain of implicit event reactions.
- **Keep an orchestrator that calls across the Session 6 service boundary synchronously** —
  rejected. Reintroduces the tight coupling the extraction was meant to remove and creates
  distributed-transaction risk (a cross-service call can fail independently of either service's
  own consistency guarantees).

## Consequences

- Session 5: the workflow has one place that encodes "what happens next" and "how to undo it" —
  straightforward to reason about, but only because everything is still in-process. Fraud
  validation is a required workflow step per the kata and is modeled as its own `Fraud Check
  Module`, gating the saga orchestrator's payment step.
- Session 6 onward: no central coordinator. Order-completion/compensation logic is split between
  the Ordering Service (`Order Status Consumer`) and the Monolith (`Fraud Check Module` and
  `Payment Module`); understanding the end-to-end flow requires reading both services' component
  diagrams.

## Trade-offs

Orchestration gives a single source of truth and simpler debugging but only works within one
deployment unit. Choreography scales across independently deployable services but is harder to
trace end-to-end and requires each participant to implement its own compensation correctly.
