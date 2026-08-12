### Level 3: Component Diagram (Ordering Microservice)

_Internal structure unchanged from Session 7 — carried forward per the C4 modeling rule. The existing
`Event Publisher` already emits the events (`OrderCreated`, `SeatReserved`, `OrderCancelled`) that the
new Read Model Service consumes to keep availability counts current; no new component was needed here
to support CQRS._

```mermaid
C4Component
    title Component diagram for Ordering Microservice (Session 7)

    Container(gateway, "API Gateway", "Kong", "Routes Ordering write commands here.")
    Container(broker, "Message Broker", "RabbitMQ", "Delivers events to the Monolith and the Read Model.")
    ContainerDb(db, "Ordering Write DB", "MySQL", "Service database.")

    Container_Boundary(ordering_svc, "Ordering Service") {
        Component(order_api, "Order Controller", "REST", "Handles incoming booking requests.")
        Component(order_manager, "Order Manager", "Domain Logic", "Validates and creates orders.")
        Component(reservation_svc, "Reservation Service", "Domain Logic", "Manages temporary seat locks.")
        Component(order_repo, "Order Repository", "TypeORM / Sequelize", "Persistence layer.")
        Component(event_publisher, "Event Publisher", "RabbitMQ Client", "Publishes domain events (OrderCreated, SeatReserved, OrderCancelled, OrderCompleted).")
        Component(order_status_consumer, "Order Status Consumer", "RabbitMQ Listener", "Reacts to the payment outcome: completes the order or releases the seat.")
    }

    Rel(gateway, order_api, "Directs booking traffic", "JSON/HTTPS")
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
