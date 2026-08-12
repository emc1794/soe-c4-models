### Level 3: Component Diagram (Read Model Service)

```mermaid
C4Component
    title Component diagram for Read Model Service

    Container(gateway, "API Gateway", "Kong", "Routes read queries here.")
    Container(broker, "Message Broker", "RabbitMQ", "Source of domain events.")
    ContainerDb(read_db, "Read Database", "MySQL", "Denormalized query-optimized schema.")

    Container_Boundary(read_model, "Read Model Service") {
        Component(query_api, "Query Controller", "REST", "Serves event search and availability read endpoints.")
        Component(event_consumer, "Event Consumer", "RabbitMQ Listener", "Subscribes to domain events and dispatches them to the projectors.")
        Component(event_projector, "Read Model Projector", "Domain Logic", "Applies events (EventUpdated, SeatReserved, OrderCreated, OrderCancelled...) to the read views.")
        Component(event_search_view, "Event Search View", "Read Repository", "Denormalized event/venue catalog for fast search.")
        Component(availability_view, "Ticket/Seat Availability View", "Read Repository", "Running counts of available seats and tickets per event.")
    }

    Rel(gateway, query_api, "Search / availability queries", "JSON/HTTPS")
    Rel(query_api, event_search_view, "Reads")
    Rel(query_api, availability_view, "Reads")

    Rel(broker, event_consumer, "Delivers domain events", "AMQP")
    Rel(event_consumer, event_projector, "Dispatches event")
    Rel(event_projector, event_search_view, "Updates on EventUpdated/EventCancelled")
    Rel(event_projector, availability_view, "Updates on SeatReserved/OrderCreated/OrderCancelled")

    Rel(event_search_view, read_db, "SQL/JDBC")
    Rel(availability_view, read_db, "SQL/JDBC")
```

**Consistency note:** the Read Model is intentionally eventually consistent — a seat reservation is
visible in the Ordering Service's write path immediately, but the Availability View only reflects it
once the corresponding event has been consumed off RabbitMQ and projected. This is acceptable for
search/availability display; the actual reservation is still enforced by the Ordering Service's write
path, not by anything read here.
