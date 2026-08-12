### Level 2: Container Diagram (CQRS Read Model)

```mermaid
C4Container
    title Container diagram for TicketWave Events - Session 8 (CQRS)

    Person(customer, "Customer", "Uses the system to find and buy tickets.")

    System_Boundary(ticketwave_boundary, "TicketWave Events Platform") {
        Container(spa, "SPA / Mobile App", "TypeScript, React", "Client application.")
        Container(gateway, "API Gateway", "Kong", "Off-the-shelf API Gateway (configured, not developed). Single entry point — routes writes to the write side, reads to the Read Model.")

        Container(ordering_svc, "Ordering Service", "TypeScript, Node.js", "Write model for reservations and ticket purchases.")
        ContainerDb(ordering_db, "Ordering Write DB", "MySQL", "Source of truth for orders and seat locks.")

        Container(monolith, "Core Monolith", "TypeScript, Node.js", "Write model for Identity, Events, Payments, and Notifications.")
        ContainerDb(monolith_db, "Monolith Write DB", "MySQL", "Source of truth for events, users, and payments.")

        Container(broker, "Message Broker", "RabbitMQ", "Carries domain events, including those that update the read model.")

        Container(read_model, "Read Model Service", "TypeScript, Node.js", "Query-only service for event search and ticket/seat availability. Builds its store from domain events instead of querying the write databases.")
        ContainerDb(read_db, "Read Database", "MySQL", "Denormalized, query-optimized schema: event search index, seat/ticket availability counts. Eventually consistent with the write side.")
    }

    System_Ext(payment, "Payment Gateway", "External payment processor.")
    System_Ext(venue, "Venue Systems", "Seating maps.")

    Rel(customer, spa, "Uses", "HTTPS")
    Rel(spa, gateway, "Sends all requests to", "JSON/HTTPS")

    Rel(gateway, monolith, "Routes Core write commands (auth, payments, admin)", "JSON/HTTPS")
    Rel(gateway, ordering_svc, "Routes Ordering write commands (reserve, purchase)", "JSON/HTTPS")
    Rel(gateway, read_model, "Routes read queries (search, availability)", "JSON/HTTPS")

    Rel(ordering_svc, ordering_db, "SQL/JDBC")
    Rel(monolith, monolith_db, "SQL/JDBC")
    Rel(read_model, read_db, "SQL/JDBC")

    Rel(ordering_svc, broker, "Publishes events (OrderCreated, SeatReserved, OrderCancelled)", "AMQP")
    Rel(monolith, broker, "Publishes/Consumes events (EventUpdated, EventCancelled, OrderCompleted)", "AMQP")
    Rel(broker, read_model, "Delivers domain events to update the read store", "AMQP")

    Rel(monolith, payment, "Payments", "HTTPS/API")
    Rel(monolith, venue, "Venue Sync", "HTTPS/API")
```

**Architectural delta from Session 7:** a new Read Model Service and Read Database appear, fed
asynchronously through RabbitMQ instead of being queried directly from the write-side databases. The
Gateway now splits traffic by intent — commands still go to the Monolith or the Ordering Service;
event-search and availability queries go to the Read Model. `monolith_db` and `ordering_db` remain the
sources of truth (write model); `read_db` is a derived, eventually-consistent projection (read model).

**Why CQRS here and not elsewhere:** event search, event availability, and seat/ticket availability are
exactly the read-heavy, spike-prone workloads called out for this session — the same on-sale traffic
spikes described in `ticketwave-events.md`. Splitting them out means a flood of read traffic during a
popular on-sale no longer contends with the Ordering Service's write path (seat reservation), which was
already the system's most contention-sensitive part and the reason it was extracted first, in Session 6.
Nothing else in the system gets a read/write split — there's no workload pressure that justifies it.
