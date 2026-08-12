# ADR 0004: Introduce Kong as the API Gateway

- **Session:** 7 — API Gateway
- **Status:** Accepted

## Context

After the Session 6 extraction, the SPA calls the Core Monolith and the Ordering Service
directly — two base URLs, with routing, authentication, and rate limiting left implicitly to the
client.

## Problem

Client complexity grows with every future service extraction, and cross-cutting concerns
(authentication, routing, rate limiting) would otherwise have to be duplicated in every backend
service, or worse, in the client itself.

## Decision

Introduce Kong as a single client entry point, sitting in front of the Core Monolith and the
Ordering Service and routing requests to each based on domain. Kong is an off-the-shelf product
that gets configured, not developed — it is modeled as an opaque Container in C2, with no C3
diagram, the same treatment already given to MySQL and RabbitMQ.

## Alternatives considered

- **A hand-built Node.js reverse proxy / router** — rejected. Reinvents commodity infrastructure
  the kata's technology constraints don't call for; adds development and maintenance cost with no
  architectural benefit over an off-the-shelf gateway.
- **No gateway, keep two client-facing base URLs** — rejected. Doesn't scale as more services are
  extracted in the future and pushes routing/auth concerns onto the client.

## Consequences

- Single client entry point; the SPA talks to one host.
- New operational dependency (Kong) to configure and run.
- Routing rules become configuration, not code — no internal architecture of ours to document
  beyond the C2 container.

## Trade-offs

An added infrastructure piece and one more network hop, in exchange for a simplified client and
centralized cross-cutting concerns that would otherwise be duplicated per service.
