### Level 3: Component Diagram (Reduced Monolith)

```mermaid
C4Component
    title Component diagram for Reduced Monolith (Session 6)

    Container(spa, "SPA / Mobile App", "React", "Calls the Monolith directly — no Gateway yet.")
    Container(broker, "Message Broker", "RabbitMQ", "External coordination with the Ordering Service.")
    ContainerDb(db, "Monolith DB", "MySQL", "Core database.")
    System_Ext(payment_ext, "Payment Gateway", "External payment processor.")

    Container_Boundary(monolith, "Core Monolith") {
        Component(identity, "Identity Module", "Domain Logic", "Auth and digital passes.")
        Component(events, "Events Module", "Domain Logic", "Event catalog and venues.")
        Component(fraud, "Fraud Check Module", "Domain Logic", "Validates the order for fraud risk before payment is attempted.")
        Component(payments, "Payment Module", "Integration", "Processes payments; reacts to Ordering events.")
        Component(notifs, "Notification Module", "Integration", "Alerts and emails.")

        Component(order_sync, "Ordering Event Consumer", "RabbitMQ Listener", "Listens for completed orders to issue tickets/notifications.")
    }

    Rel(spa, identity, "Auth requests")
    Rel(spa, events, "Catalog requests")

    Rel(broker, fraud, "Delivers 'OrderCreated'", "AMQP")
    Rel(fraud, payments, "Fraud check passed — process payment", "Internal Call")
    Rel(fraud, broker, "Publishes 'PaymentFailed' (fraud rejection)", "AMQP")
    Rel(payments, payment_ext, "Authorizes and captures payment", "HTTPS/API")
    Rel(payments, broker, "Publishes 'PaymentProcessed' / 'PaymentFailed'", "AMQP")

    Rel(broker, order_sync, "Receives 'OrderCompleted'")
    Rel(order_sync, identity, "Triggers ticket issuance")
    Rel(order_sync, notifs, "Triggers confirmation alert")

    Rel(identity, db, "SQL/JDBC")
    Rel(events, db, "SQL/JDBC")
    Rel(payments, db, "SQL/JDBC")
```

**Note:** No API Gateway yet — the SPA calls this monolith's endpoints directly.

**Delta from Session 5 — the Fraud Check and Payment Modules now close the saga loop over the
broker:** the Session 5 in-process Booking Saga Orchestrator doesn't survive this session's
extraction (an orchestrator can't make synchronous in-process calls across two independently
deployable services). The Fraud Check Module still gates payment exactly as it did in Session 5 —
it reacts to `OrderCreated` first and only lets the Payment Module proceed on a pass, publishing
`PaymentFailed` itself on a fraud rejection. The Payment Module then publishes the payment outcome
back over `broker` — see `level-3-components-ordering-service.md` for the Ordering Service side of
this choreographed flow.
