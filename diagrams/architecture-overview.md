# Architecture Overview

High-level view of platform components and their relationships.

```mermaid
flowchart TD
    subgraph Users
        Browser[Browser]
    end

    subgraph Frontend["Frontend Application"]
        NextJS[Next.js App Router]
        Pages[Marketing / Shop / Dashboard / Admin]
        Client[HTTP Client + Auth Bridge]
    end

    subgraph Backend["Backend Services"]
        Express[Express REST API]
        Domains[Domain Modules]
        Middleware[Auth / CSRF / Rate Limiting]
        Proxy[Storefront Reverse Proxy]
    end

    subgraph Database
        MongoDB[(MongoDB)]
    end

    subgraph ExternalAPIs["External APIs"]
        ClerkAPI[Clerk]
        StripeAPI[Stripe]
        FedExAPI[FedEx]
        PostmarkAPI[Postmark]
        MapboxAPI[Mapbox]
    end

    subgraph Infrastructure
        VM[Linux VM]
        PM2[PM2 Process Manager]
        Doppler[Doppler Secrets]
        GHA[GitHub Actions CI/CD]
        Images[Local Image Storage]
    end

    Browser --> NextJS
    Browser --> Express
    NextJS --> Pages
    Pages --> Client
    Client -->|REST + JWT| Express
    Express --> Middleware
    Middleware --> Domains
    Domains --> MongoDB
    Domains --> ClerkAPI
    Domains --> StripeAPI
    Domains --> FedExAPI
    Domains --> PostmarkAPI
    Domains --> MapboxAPI
    Proxy --> NextJS
    Express --> Proxy
    Domains --> Images

    GHA -->|Deploy| VM
    Doppler -->|Secrets| VM
    PM2 --> NextJS
    PM2 --> Express
```

## Component responsibilities

| Component | Role |
|---|---|
| **Next.js Frontend** | User interface for storefront, customer dashboard, and admin portal |
| **Express Backend** | Central business logic, API contracts, webhook processing |
| **MongoDB** | Primary data store for users, orders, quotes, tickets, and audit logs |
| **External APIs** | Identity, payments, shipping, email, and geocoding |
| **Infrastructure** | VM hosting, process management, secrets injection, and automated deployment |

## Data flow pattern

1. Users interact with the Next.js frontend or API directly.
2. Authenticated requests carry Clerk JWTs; mutations require CSRF tokens.
3. The backend orchestrates external service calls and persists results to MongoDB.
4. Webhooks from Stripe and Clerk trigger asynchronous state updates (order creation, user sync).
