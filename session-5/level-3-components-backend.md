### Level 3: Component Diagram (Saga Orchestration)

```mermaid
C4Component
    title Component diagram for Backend Monolith - Saga Workflows

    Container(broker, "Message Broker", "RabbitMQ", "Central event bus.")
    ContainerDb(database, "Main Database", "MySQL", "Stores system state.")

    Container_Boundary(monolith, "Backend Monolith") {
        
        Boundary(ordering_context, "Ordering Context") {
            Component(ordering_svc, "Ordering Service", "Domain Logic", "Handles order management.")
            Component(saga_orchestrator, "Booking Saga Orchestrator", "State Machine", "Coordinates the Booking -> Payment -> Issuance workflow.")
        }

        Boundary(events_context, "Events Context") {
            Component(events_svc, "Events Module", "Domain Logic", "Handles seat reservations and releases.")
        }

        Boundary(payment_context, "Payment Context") {
            Component(fraud_svc, "Fraud Check Module", "Domain Logic", "Validates the order for fraud risk before payment is attempted.")
            Component(payment_svc, "Payment Module", "Domain Logic", "Processes payments and handles compensations.")
        }

        Boundary(identity_context, "Identity Context") {
            Component(identity_svc, "Identity Module", "Domain Logic", "Issues digital passes.")
        }
    }

    Rel(saga_orchestrator, events_svc, "1. Command: Reserve Seat", "Internal Call")
    Rel(events_svc, saga_orchestrator, "2. Event: Seat Reserved", "Internal Event")
    
    Rel(saga_orchestrator, fraud_svc, "3. Command: Validate Fraud Risk", "Internal Call")
    Rel(fraud_svc, saga_orchestrator, "4. Event: Fraud Check Passed", "Internal Event")

    Rel(saga_orchestrator, broker, "5. Command: Process Payment", "AMQP")
    Rel(broker, payment_svc, "Delivers Payment Command", "Events")
    
    Rel(payment_svc, broker, "6. Event: Payment Successful", "Events")
    Rel(broker, saga_orchestrator, "Delivers Success Event", "Events")
    
    Rel(saga_orchestrator, identity_svc, "7. Command: Issue Ticket", "Internal Call")
    Rel(identity_svc, saga_orchestrator, "8. Event: Ticket Issued", "Internal Event")

    Rel(saga_orchestrator, ordering_svc, "9. Update Order Status to 'Completed'", "Internal Call")
    
    Rel(saga_orchestrator, events_svc, "Compensation: Release Seat", "Internal Call (on failure — fraud rejection or payment failure)")
```

**Note:** this diagram is scoped to the saga's own steps, so it omits the Notification Module —
unchanged since Session 2 and still present in the monolith (see the C2 diagram's
`Rel(api, notifications, ...)`) — since sending the confirmation alert isn't itself a saga step
with a compensation path.
