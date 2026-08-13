### Level 3: Component Diagram (Frontend - Saga-Aware Checkout)

```mermaid
C4Component
    title Component diagram for Frontend - Session 5 (Saga-Aware Checkout)

    Container(api, "Backend Monolith", "TypeScript, Node.js", "Handles requests and drives the purchase Saga.")

    Container_Boundary(frontend, "Frontend Application") {

        Boundary(shared_services, "Shared Services") {
            Component(auth_provider, "Auth Provider", "OIDC / JWT", "Handles login and token storage.")
            Component(api_client, "Resilient API Client", "Axios + Retry Logic", "Handles requests with built-in retry and timeout policies.")
        }

        Boundary(ui_features, "UI Features") {
            Component(booking_ui, "Booking Flow", "React", "Submits the purchase request that starts the Saga.")
            Component(saga_status, "Saga Status Tracker", "React", "Polls/listens for the workflow's current step (reserved, paid, issued, failed, compensated) and reflects it in the UI.")
            Component(profile_ui, "User Dashboard", "React", "User interface for viewing tickets.")
        }
    }

    Rel(booking_ui, api_client, "Requests booking (starts Saga)")
    Rel(saga_status, api_client, "Polls order/Saga status")
    Rel(profile_ui, api_client, "Requests tickets")

    Rel(api_client, auth_provider, "Checks tokens")
    Rel(api_client, api, "HTTPS/JSON requests (with JWT)", "JSON/HTTPS")

    Rel(booking_ui, saga_status, "Hands off after submission")
```

**Note:** No API Gateway container yet — the client talks to the Backend Monolith directly. The
single client entry point is introduced in Session 7.
