### Level 3: Component Diagram (Frontend - Microservices Era)

```mermaid
C4Component
    title Component diagram for Frontend - Session 6

    Container(monolith, "Core Monolith", "TypeScript, Node.js", "Identity, Events, Payments, Notifications.")
    Container(ordering_svc, "Ordering Service", "TypeScript, Node.js", "Reservations and ticket purchases.")

    Container_Boundary(frontend, "Frontend Application") {

        Boundary(services, "Service Adapters") {
            Component(ordering_adapter, "Ordering Service Adapter", "Axios", "Calls the Ordering Service base URL directly.")
            Component(core_adapter, "Core Service Adapter", "Axios", "Calls the Core Monolith base URL directly.")
        }

        Boundary(feature_views, "Feature Views") {
            Component(search_view, "Search & Catalog", "React", "Uses Core Adapter.")
            Component(booking_view, "Booking & Checkout", "React", "Uses Ordering Adapter.")
            Component(profile_view, "User Dashboard", "React", "Uses Core Adapter.")
        }
    }

    Rel(search_view, core_adapter, "Fetch events")
    Rel(booking_view, ordering_adapter, "Submit booking")
    Rel(profile_view, core_adapter, "Fetch profile")

    Rel(core_adapter, monolith, "HTTPS/JSON")
    Rel(ordering_adapter, ordering_svc, "HTTPS/JSON")
```

**Note:** No API Gateway yet — each adapter targets its service's own base URL. Session 7 introduces
a single Gateway entry point that both adapters will route through instead.
