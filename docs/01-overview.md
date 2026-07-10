# Overview

**592 Industries Platform — Executive Summary**

---

## Company and Platform Overview

592 Industries is an engineering and technology solutions company that provides custom manufacturing, software development, infrastructure services, automation solutions, and related technology offerings. The business model centers on customer-specific engineering solutions rather than mass production—every engagement can have different requirements.

To support this model, the company built a unified business operations platform that combines:

- A public storefront for standardized product sales
- Custom engineering quote workflows for non-catalog projects
- Customer self-service for orders, billing, support, and privacy
- Admin operations for fulfillment, finance, catalog management, and team coordination

The platform was developed as Version 1 over a focused two-week engineering sprint (June 2026), with architecture and requirements defined before implementation.

---

## Problem Statement

### Business context

Traditional website builders and ecommerce platforms provide rapid deployment but introduce fundamental limitations:

- Vendor-controlled workflows that cannot accommodate custom engineering processes
- Limited operational visibility into fulfillment, finance, and support
- Restricted integration flexibility with shipping, payments, and identity providers
- Reduced control over customer data and business logic

592 Industries needed a platform capable of supporting non-standard workflows—replacement components, custom-designed parts, automation systems, software solutions, and infrastructure deployments—while maintaining full ownership of operational processes.

### What the system must solve

| Challenge | System response |
|---|---|
| Diverse customer needs | Dual paths: storefront checkout and custom quote intake |
| Operational complexity | Admin portal with role-based access and state-machine-driven fulfillment |
| Payment compliance | Stripe-hosted checkout; no card data on platform servers |
| Shipping accuracy | FedEx address validation and rate quotes before payment |
| Customer trust | Self-service order tracking, support tickets, and privacy controls |

---

## Solution Overview

The platform is implemented as two independently deployable services sharing a single public hostname:

| Service | Technology | Responsibility |
|---|---|---|
| Frontend | Next.js 16, TypeScript, React 19 | User interfaces for marketing, commerce, dashboard, and admin |
| Backend | Express 5, Node.js 20 | Business logic, API, webhooks, and external integrations |

**Primary data store:** MongoDB  
**Identity:** Clerk  
**Payments:** Stripe Checkout with automatic tax  
**Shipping:** FedEx REST APIs  
**Email:** Postmark transactional delivery

The engineering philosophy: *build the platform first, then build interfaces around the platform.* The backend API is designed to support future clients including mobile applications and internal tools.

---

## Stakeholders

### Internal stakeholders

| Stakeholder | Primary needs |
|---|---|
| **Owner / Executive** | Revenue visibility, operational KPIs, team management, audit trail |
| **Operations Manager** | Fulfillment queue, catalog updates, shipping configuration |
| **Fulfillment Staff** | Assigned orders, status updates, label creation, tracking |
| **Support Agent** | Ticket management with customer and order context |
| **Finance Viewer** | Revenue summaries, transaction history, refund processing |
| **Platform Engineer** | Deploy automation, health monitoring, secret management |

### External stakeholders

| Stakeholder | Interaction |
|---|---|
| **Customer** | Purchases products, submits quotes, manages account |
| **Prospect** | Browses catalog and marketing content; guest checkout supported |
| **Stripe, Clerk, FedEx, Postmark** | Integrated services for payments, identity, shipping, and email |

---

## System Goals

### Product goals

- Enable self-service commerce for standardized engineering products
- Provide structured intake for custom engineering quote requests
- Deliver customer visibility into order status, tracking, and support
- Support privacy commitments through data export and account deletion

### Engineering goals

- Maintain ownership of business logic and customer data
- Design for future expansion (mobile apps, additional services)
- Integrate secure third-party services without sacrificing control
- Automate operational processes where possible (webhook order creation, optional auto-labeling)
- Enforce security at every layer (auth, CSRF, rate limiting, webhook verification)

### Operational goals

- Repeatable, auditable deployments via CI/CD
- Automated health verification before and after releases
- Role-appropriate admin access with capability-based permissions
- Documented incident response and maintenance procedures

---

## Document navigation

| Next | Topic |
|---|---|
| [Product Requirements →](02-product-requirements.md) | What the system was designed to deliver |
| [System Architecture →](03-system-architecture.md) | How components fit together |
| [User Workflows →](04-user-workflows.md) | End-to-end customer and admin journeys |
