### Level 3: Component Diagram (Reduced Monolith)

_Internal structure unchanged from Session 7 — carried forward per the C4 modeling rule. The Events
Module already publishes the domain events (`EventUpdated`, `EventCancelled`) that the new Read Model
Service (see `level-3-components-read-model.md`) subscribes to; no new component was needed here to
support CQRS._

```mermaid
C4Component
    title Component diagram for Reduced Monolith (Session 7)

    Container(gateway, "API Gateway", "Kong", "Routes Core write commands here.")
    Container(broker, "Message Broker", "RabbitMQ", "Delivers events to the Ordering Service and the Read Model.")
    ContainerDb(db, "Monolith Write DB", "MySQL", "Core database.")
    System_Ext(payment_ext, "Payment Gateway", "External payment processor.")

    Container_Boundary(monolith, "Core Monolith") {
        Component(identity, "Identity Module", "Domain Logic", "Auth and digital passes.")
        Component(events, "Events Module", "Domain Logic", "Event catalog and venues.")
        Component(fraud, "Fraud Check Module", "Domain Logic", "Validates the order for fraud risk before payment is attempted.")
        Component(payments, "Payment Module", "Integration", "Processes payments; reacts to Ordering events.")
        Component(notifs, "Notification Module", "Integration", "Alerts and emails.")

        Component(order_sync, "Ordering Event Consumer", "RabbitMQ Listener", "Listens for completed orders to issue tickets/notifications.")
    }

    Rel(gateway, identity, "Auth requests")
    Rel(gateway, events, "Catalog admin requests")

    Rel(broker, fraud, "Delivers 'OrderCreated'", "AMQP")
    Rel(fraud, payments, "Fraud check passed — process payment", "Internal Call")
    Rel(fraud, broker, "Publishes 'PaymentFailed' (fraud rejection)", "AMQP")
    Rel(payments, payment_ext, "Authorizes and captures payment", "HTTPS/API")
    Rel(payments, broker, "Publishes 'PaymentProcessed' / 'PaymentFailed'", "AMQP")

    Rel(broker, order_sync, "Receives 'OrderCompleted'")
    Rel(order_sync, identity, "Triggers ticket issuance")
    Rel(order_sync, notifs, "Triggers confirmation alert")

    Rel(events, broker, "Publishes 'EventUpdated' / 'EventCancelled'", "AMQP")

    Rel(identity, db, "SQL/JDBC")
    Rel(events, db, "SQL/JDBC")
    Rel(payments, db, "SQL/JDBC")
```
