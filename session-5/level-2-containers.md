### Level 2: Container Diagram (Saga Orchestration)

```mermaid
C4Container
    title Container diagram for TicketWave Events - Session 5 (Saga Orchestration)

    Person(customer, "Customer", "Uses the system to find and buy tickets.")
    Person(staff, "Support Staff", "Manages events and supports customers.")

    System_Boundary(ticketwave_boundary, "TicketWave Events Platform") {
        Container(spa, "Single Page Application / Mobile App", "TypeScript, React", "Provides user interface.")
        Container(api, "Backend Monolith", "TypeScript, Node.js", "Modular monolith implementing the purchase workflow as a Saga (reservation, payment, fraud check, issuance, compensation).")
        Container(broker, "Message Broker", "RabbitMQ", "Central hub for Saga command/event choreography between modules.")
        ContainerDb(database, "Main Database", "MySQL", "Stores user profiles, event details, seating maps, and transaction records.")
    }

    System_Ext(payment, "Payment Gateway", "Processes online payments.")
    System_Ext(venue, "Venue Systems", "Seating maps and access control.")
    System_Ext(notifications, "Notification Service", "Email/SMS delivery.")

    Rel(customer, spa, "Uses", "HTTPS")
    Rel(staff, spa, "Uses", "HTTPS")

    Rel(spa, api, "Makes API calls to", "JSON/HTTPS")
    Rel(api, database, "Reads from and writes to", "SQL/JDBC")

    Rel(api, broker, "Publishes and Consumes Saga events", "AMQP")

    Rel(api, payment, "Processes payments", "HTTPS/API")
    Rel(api, venue, "Syncs maps and tickets", "HTTPS/API")
    Rel(api, notifications, "Sends messages", "HTTPS/API")
```

**Note:** No API Gateway container yet. The single client entry point is introduced in Session 7 —
see the Session 7 models and the corresponding note in `CLAUDE.md`'s deviations section for why this
was corrected after initially appearing here.
