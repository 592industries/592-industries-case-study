# System Context Diagram

This diagram shows how external actors and services interact with the 592 Industries platform.

```mermaid
flowchart TB
    subgraph ExternalActors["External Actors"]
        Customer[Customer / Buyer]
        Prospect[Prospect / Visitor]
        Admin[Operations Staff]
    end

    subgraph Platform["592 Industries Platform"]
        WebApp[Web Application]
        API[REST API & Business Logic]
        DB[(MongoDB)]
    end

    subgraph Manufacturing["Manufacturing Operations"]
        Fulfillment[Fulfillment & Production]
        Shipping[Shipping & Labeling]
    end

    subgraph ExternalServices["External Services"]
        Clerk[Clerk — Identity]
        Stripe[Stripe — Payments]
        FedEx[FedEx — Shipping]
        Postmark[Postmark — Email]
        Mapbox[Mapbox — Maps]
        Honeybadger[Honeybadger — Monitoring]
    end

    Prospect --> WebApp
    Customer --> WebApp
    Admin --> WebApp

    WebApp --> API
    API --> DB

    API --> Fulfillment
    Fulfillment --> Shipping

    API --> Clerk
    API --> Stripe
    API --> FedEx
    API --> Postmark
    API --> Mapbox
    API --> Honeybadger

    Clerk -->|Webhooks| API
    Stripe -->|Webhooks| API

    Customer -->|Hosted Checkout| Stripe
    Customer -->|Sign In| Clerk
```

## Context summary

| Boundary | Responsibility |
|---|---|
| **592 Industries Platform** | Business logic, workflows, customer data, operational tooling |
| **Manufacturing Operations** | Order processing, production, packing, and shipment creation |
| **External Services** | Specialized capabilities delegated to industry-standard providers |

The platform maintains ownership of business rules and data while integrating external services for identity, payments, shipping, and communications.
