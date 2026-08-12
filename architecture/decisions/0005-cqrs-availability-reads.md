# ADR 0005: CQRS for read-heavy, spike-prone availability queries

- **Session:** 8 — CQRS
- **Status:** Accepted

## Context

Event search, event availability, and seat/ticket availability are read-heavy and spike hard
during popular on-sales. Writes to Ordering (seat reservation) are comparatively low-volume but
contention-sensitive, and were already isolated as their own service and database in Session 6
(ADR 0003) precisely because of that sensitivity.

## Problem

Serving read-heavy, spike-prone availability queries from the same write databases (Monolith DB,
Ordering DB) risks read traffic contending with, and degrading, the write path that matters most
for correctness — seat reservation and duplicate-sale prevention.

## Decision

Introduce a **Read Model Service** with its own read-optimized store, populated asynchronously
over RabbitMQ from events already published by the Events Module (`EventUpdated`,
`EventCancelled`) and the Ordering Service (`OrderCreated`, `SeatReserved`, `OrderCancelled`).
Search and availability reads are served from the Read Model; writes still go through the
Monolith and Ordering Service write paths and databases, now labeled "Write DB" to make the
split explicit.

## Alternatives considered

- **Apply CQRS everywhere** — rejected. No workload pressure justifies a read/write split for
  Identity, Notifications, or Payments; it would add complexity without benefit.
- **Read replicas of the write databases instead of a dedicated read model** — rejected. Still
  couples read scaling to the write-side schema and service boundary, and doesn't fit
  search-shaped queries (e.g. filtering availability across events) as naturally as a purpose-built
  read model.

## Consequences

- Read traffic for search/availability no longer contends with the write path during on-sale
  spikes.
- The read model is eventually consistent with the write side; a customer can briefly see stale
  availability immediately after a reservation.
- Requires monitoring event-propagation/replication lag between the write services and the read
  model.

## Trade-offs

Eventual consistency on reads, in exchange for read and write workloads that can be scaled and
evolved independently — the read-heavy, spike-prone workload the kata calls out for this session.
