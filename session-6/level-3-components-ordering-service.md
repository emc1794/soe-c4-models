### Level 3: Component Diagram (Ordering Microservice)

```mermaid
C4Component
    title Component diagram for Ordering Microservice

    Container(spa, "SPA / Mobile App", "React", "Client application. Calls this service directly — no Gateway yet.")
    Container(broker, "Message Broker", "RabbitMQ", "Event bus.")
    ContainerDb(db, "Ordering DB", "MySQL", "Service database.")

    Container_Boundary(ordering_svc, "Ordering Service") {
        Component(order_api, "Order Controller", "REST", "Handles incoming booking requests.")
        Component(order_manager, "Order Manager", "Domain Logic", "Validates and creates orders.")
        Component(reservation_svc, "Reservation Service", "Domain Logic", "Manages temporary seat locks.")
        Component(order_repo, "Order Repository", "TypeORM / Sequelize", "Persistence layer.")
        Component(event_publisher, "Event Publisher", "RabbitMQ Client", "Publishes domain events (OrderCreated, OrderCompleted).")
        Component(order_status_consumer, "Order Status Consumer", "RabbitMQ Listener", "Reacts to the payment outcome: completes the order or releases the seat.")
    }

    Rel(spa, order_api, "Sends booking requests directly", "JSON/HTTPS")
    Rel(order_api, order_manager, "Invokes")
    Rel(order_manager, reservation_svc, "Requests seat lock")
    Rel(order_manager, order_repo, "Saves order")
    Rel(order_manager, event_publisher, "Triggers 'OrderCreated' event")

    Rel(broker, order_status_consumer, "Delivers 'PaymentProcessed' / 'PaymentFailed'", "AMQP")
    Rel(order_status_consumer, order_manager, "Marks order 'Completed'")
    Rel(order_status_consumer, event_publisher, "Triggers 'OrderCompleted' event (on payment success)")
    Rel(order_status_consumer, reservation_svc, "Compensation: Release Seat (on payment failure)")

    Rel(order_repo, db, "SQL/JDBC")
    Rel(event_publisher, broker, "Sends events", "AMQP")
```

**Note:** No API Gateway yet — the client calls this service's own base URL directly.

**Delta from Session 5 — orchestration becomes choreography:** the single in-process Booking Saga
Orchestrator doesn't survive this session's extraction into two independently deployable services — an
orchestrator can't make synchronous in-process calls across a service boundary without reintroducing
the coupling the extraction was meant to remove. Coordination is now choreographed over `broker`: this
service publishes `OrderCreated`, the Monolith's Fraud Check Module reacts first and gates the
Payment Module (which charges the customer and publishes `PaymentProcessed`/`PaymentFailed`, or the
Fraud Check Module itself publishes `PaymentFailed` on a fraud rejection — see
`level-3-components-monolith-backend.md`), and the `Order Status Consumer` here reacts to that
outcome — completing the order (whose `OrderCompleted`
event the Monolith's `Ordering Event Consumer` still consumes for ticket issuance/notification, as
before) or compensating by releasing the seat. This is a consequence of the extraction, not a
preference for choreography over orchestration.
