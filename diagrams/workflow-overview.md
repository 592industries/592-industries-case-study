# Workflow Overview

End-to-end journey from customer request through delivery.

```mermaid
flowchart TD
    A[Customer Request] --> B{Request Type}

    B -->|Catalog Product| C[Product Discovery]
    B -->|Custom Engineering| D[Quote Submission]

    C --> E[Cart & Checkout]
    E --> F[Address Validation]
    F --> G[Shipping Rate Selection]
    G --> H[Stripe Payment]
    H --> I[Order Created]

    D --> J[Admin Review]
    J --> K[Pricing & Quote Sent]
    K --> L[Customer Decision]

    I --> M[Processing]
    L -->|Accepted| M
    L -->|Rejected| N[Closed]

    M --> O[Production / Preparation]
    O --> P[Packing & Labeling]
    P --> Q[FedEx Shipment]
    Q --> R[In Transit]
    R --> S[Delivery]
    S --> T[Support & Account Management]

    I --> T
    N --> T
```

## Workflow phases

| Phase | Description | Primary actors |
|---|---|---|
| **Customer Request** | Browse catalog or submit engineering quote | Customer, Prospect |
| **Processing** | Payment confirmation or quote review | System, Admin |
| **Production** | Manufacturing, preparation, or project scoping | Fulfillment staff |
| **Delivery** | Shipping, tracking, and delivery confirmation | FedEx, Customer |
| **Support** | Tickets, returns, billing, and account management | Customer, Support |

## Dual revenue paths

The platform supports two complementary business models:

1. **Storefront commerce** — Self-service purchase of standardized products with automated checkout and webhook-driven order creation.
2. **Custom quotes** — Structured intake for engineering projects requiring manual pricing and admin review before acceptance.

Both paths converge at fulfillment and ongoing customer relationship management.
