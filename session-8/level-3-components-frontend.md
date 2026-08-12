### Level 3: Component Diagram (Frontend - Gateway Integration)

_Unchanged from Session 7 — carried forward per the C4 modeling rule. The frontend still talks to a
single API Gateway and has no awareness that search/availability reads and booking writes are now
served by different backend containers; that split is entirely hidden behind the Gateway._

```mermaid
C4Component
    title Component diagram for Frontend - Session 7 (Gateway Integration)

    Container(gateway, "API Gateway", "Kong", "Single entry point: routing, auth, rate limiting.")

    Container_Boundary(frontend, "Frontend Application") {

        Boundary(shared_services, "Shared Services") {
            Component(auth_provider, "Auth Provider", "OIDC / JWT", "Handles login and token storage.")
            Component(api_client, "Resilient API Client", "Axios + Retry Logic", "Handles requests with built-in retry and timeout policies.")
        }

        Boundary(ui_features, "UI Features") {
            Component(search_view, "Search & Catalog", "React", "Browses events.")
            Component(booking_view, "Booking & Checkout", "React", "Buys tickets via the Saga-driven purchase flow.")
            Component(profile_view, "User Dashboard", "React", "Views tickets and profile.")
        }
    }

    Rel(search_view, api_client, "Fetch events")
    Rel(booking_view, api_client, "Submit booking")
    Rel(profile_view, api_client, "Fetch tickets")

    Rel(api_client, auth_provider, "Checks tokens")
    Rel(api_client, gateway, "HTTPS/JSON requests (with JWT)", "JSON/HTTPS")
```
