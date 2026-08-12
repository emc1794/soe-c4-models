### Level 2: Container Diagram (API Gateway)

```mermaid
C4Container
    title Container diagram for TicketWave Events - Session 7 (API Gateway)

    Person(customer, "Customer", "Uses the system to find and buy tickets.")

    System_Boundary(ticketwave_boundary, "TicketWave Events Platform") {
        Container(spa, "SPA / Mobile App", "TypeScript, React", "Client application. Now talks to a single entry point.")
        Container(gateway, "API Gateway", "Kong", "Off-the-shelf API Gateway (configured, not developed). Single client entry point — authenticates requests and routes them to the Monolith or the Ordering Service.")

        Container(ordering_svc, "Ordering Service", "TypeScript, Node.js", "Microservice handling reservations and ticket purchases.")
        ContainerDb(ordering_db, "Ordering DB", "MySQL", "Dedicated database for the Ordering domain.")

        Container(monolith, "Core Monolith", "TypeScript, Node.js", "Handles Identity, Events, Payments, and Notifications.")
        ContainerDb(monolith_db, "Monolith DB", "MySQL", "Stores core data excluding Ordering.")

        Container(broker, "Message Broker", "RabbitMQ", "Asynchronous event communication between the Monolith and the Ordering Service.")
    }

    System_Ext(payment, "Payment Gateway", "External payment processor.")
    System_Ext(venue, "Venue Systems", "Seating maps.")

    Rel(customer, spa, "Uses", "HTTPS")
    Rel(spa, gateway, "Sends all requests to", "JSON/HTTPS")

    Rel(gateway, monolith, "Routes Core requests", "JSON/HTTPS")
    Rel(gateway, ordering_svc, "Routes Ordering requests", "JSON/HTTPS")

    Rel(ordering_svc, ordering_db, "SQL/JDBC")
    Rel(monolith, monolith_db, "SQL/JDBC")

    Rel(ordering_svc, broker, "Publishes events", "AMQP")
    Rel(monolith, broker, "Consumes events", "AMQP")

    Rel(monolith, payment, "Payments", "HTTPS/API")
    Rel(monolith, venue, "Venue Sync", "HTTPS/API")
```

**Architectural delta from Session 6:** the client no longer calls the Monolith and the Ordering
Service directly. All traffic now enters through the API Gateway, which is the only container the SPA
talks to. Routing, authentication, and rate limiting move from being (implicitly) the client's concern
to being the Gateway's responsibility — see `level-3-components-gateway.md` for its internals.
RabbitMQ continues to carry asynchronous domain events between the two backend deployment units,
unaffected by the Gateway's introduction.
