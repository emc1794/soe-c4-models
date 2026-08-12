### Level 2: Container Diagram (Microservices Extraction)

```mermaid
C4Container
    title Container diagram for TicketWave Events - Session 6 (Microservices)

    Person(customer, "Customer", "Uses the system to find and buy tickets.")

    System_Boundary(ticketwave_boundary, "TicketWave Events Platform") {
        Container(spa, "SPA / Mobile App", "TypeScript, React", "Client application. Calls the Monolith and the Ordering Service directly — no single entry point yet.")

        Container(ordering_svc, "Ordering Service", "TypeScript, Node.js", "Extracted microservice handling reservations and ticket purchases.")
        ContainerDb(ordering_db, "Ordering DB", "MySQL", "Dedicated database for the Ordering domain.")

        Container(monolith, "Core Monolith", "TypeScript, Node.js", "Remaining monolith handling Identity, Events, Payments, and Notifications.")
        ContainerDb(monolith_db, "Monolith DB", "MySQL", "Stores core data excluding Ordering.")

        Container(broker, "Message Broker", "RabbitMQ", "Synchronizes state and handles distributed events across the newly split deployment units.")
    }

    System_Ext(payment, "Payment Gateway", "External payment processor.")
    System_Ext(venue, "Venue Systems", "Seating maps.")

    Rel(customer, spa, "Uses", "HTTPS")
    Rel(spa, monolith, "Core requests (identity, events, payments, notifications)", "JSON/HTTPS")
    Rel(spa, ordering_svc, "Ordering requests (reservations, purchases)", "JSON/HTTPS")

    Rel(ordering_svc, ordering_db, "SQL/JDBC")
    Rel(monolith, monolith_db, "SQL/JDBC")

    Rel(ordering_svc, broker, "Publishes events", "AMQP")
    Rel(monolith, broker, "Consumes events", "AMQP")

    Rel(monolith, payment, "Payments", "HTTPS/API")
    Rel(monolith, venue, "Venue Sync", "HTTPS/API")
```

**Note:** No API Gateway yet. Per the Session 6 academic focus (splitting the monolith, bounded
contexts, data isolation, independent deployment), the client calls each deployment unit directly —
the single client entry point is introduced in Session 7.
