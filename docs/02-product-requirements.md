# Product Requirements

What problem was the system designed to solve, and what capabilities does it deliver?

---

## User Needs

### Customers and prospects

| Need | How addressed |
|---|---|
| Browse engineering products with specifications | Public storefront backed by Stripe catalog with fulfillment metadata |
| Purchase without friction | Guest checkout; cart persistence across sessions |
| Accurate shipping costs before payment | FedEx rate quotes gated before Stripe session creation |
| Order visibility | Dashboard with order history, tracking timeline, and receipts |
| Custom project intake | Quote submission form with file links, budget, and deadline |
| Post-sale support | Support tickets with optional order linkage |
| Account self-service | Profile, addresses, billing portal, notification preferences |
| Privacy control | Data export and scheduled account deletion |

### Operations staff

| Need | How addressed |
|---|---|
| Centralized order management | Filterable fulfillment queue with pipeline buckets |
| Controlled status transitions | Fulfillment and shipping state machines |
| Shipping automation | FedEx label creation with optional auto-label on payment |
| Financial visibility | Revenue summaries and integrated refund processing |
| Team access control | Five admin roles with 22 granular capabilities |
| Operational accountability | Audit logging for administrative mutations |

---

## Functional Requirements

Requirements are grouped by domain. Internal requirement identifiers have been omitted for public readability.

### Identity and access

- Customer authentication via Clerk-hosted sign-in and sign-up
- Role-based access control for admin operations (owner, manager, staff, support agent, finance viewer)
- CSRF protection on all state-changing API requests
- Retirement of legacy first-party authentication routes

### Product catalog

- Public display of active, storefront-eligible products merged with fulfillment profiles
- Public download of product-associated PDF documents (datasheets, guides)
- Admin catalog management with Stripe sync and fulfillment profile editing

### Commerce and checkout

- Shopping cart persistence in browser local storage
- FedEx address validation with validated address snapshots
- Shipping rate computation with configurable pricing rules
- Stripe Checkout session creation with automatic tax
- Guest checkout support with optional account linking
- Idempotent order creation via Stripe webhooks

### Order management

- Customer order history with tracking timeline and Stripe receipt URLs
- Customer-initiated cancellation before shipment
- Return request submission and tracking

### Custom quotes

- Customer quote submission with project details and file references
- Quote status lifecycle (submitted → under review → quoted → accepted/rejected)
- Email notification when admin sends completed quote

### Support

- Customer support ticket creation with message threads
- Admin ticket management with replies and internal notes

### Account and privacy

- Profile and notification preference management synced to Clerk
- Saved address management with US address autocomplete
- Stripe billing portal and saved payment methods
- GDPR-style data export and scheduled account deletion

### Admin operations

- Fulfillment queue with assignment, status updates, and notes
- FedEx label creation, download, and tracking sync
- Configurable auto-label creation on successful payment
- Refund processing and revenue reporting
- Admin user role management
- Shipping settings and UI asset management
- Audit logging and customer record search

### Platform services

- Health and readiness endpoints checking all dependencies
- Launch readiness validation at startup
- Storefront reverse proxy on public API hostname
- Transactional email delivery with delivery logging
- Clerk webhook user synchronization
- Runtime status disclosure for integration modes
- Versioned legal policy pages

---

## Non-Functional Requirements

### Security

| Requirement | Implementation |
|---|---|
| No payment card storage | Stripe-hosted checkout only |
| API rate limiting | Global and domain-specific limiters |
| CORS origin allowlist | Reject unauthorized cross-origin requests |
| Webhook signature verification | Stripe and Clerk (Svix) signatures required |
| Security headers | Helmet CSP and CORP on API responses |
| Path traversal protection | Safe path resolution for local image storage |

### Reliability and operations

| Requirement | Implementation |
|---|---|
| Request tracing | Unique request ID on every API response |
| Error reporting | Honeybadger integration with FedEx error fingerprinting |
| Idempotent mutations | Idempotency keys on quotes, tickets, and checkout |
| Secrets management | Doppler injection; no secrets in repositories |
| Environment coherence | Production validates live Stripe and production FedEx |
| Graceful shutdown | SIGTERM handling with connection draining |
| Deploy automation | GitHub Actions with tests, rsync, and PM2 reload |
| Database indexing | Indexes applied at deploy time |

### API design

| Requirement | Implementation |
|---|---|
| Consistent response envelope | `{ success, data }` or `{ success, error }` |
| Input validation | Zod schemas on checkout requests |
| Staff order access restriction | Staff see only assigned or unassigned orders |

### Business constraints

| Requirement | Implementation |
|---|---|
| US shipping only | Checkout restricted to US addresses |
| Automatic tax | Stripe Tax enabled on checkout sessions |
| Frontend test coverage | Unit tests for HTTP client, Clerk config, and UI assets |

---

## Constraints

| Constraint | Rationale |
|---|---|
| PCI scope reduction | All card processing on Stripe-hosted pages |
| US domestic shipping | Current FedEx integration and business scope |
| Clerk as sole identity provider | Centralized auth reduces security maintenance |
| Doppler for secrets | No on-disk secrets in production |
| No real-time inventory | Made-to-order engineering products; availability via admin flags |
| Quote acceptance not automated | Custom projects require manual billing follow-up in V1 |

---

## Design Goals

1. **Flexibility over fixed workflows** — Support both catalog commerce and custom engineering intake
2. **Data ownership** — Business logic and customer data remain in-house
3. **Defense in depth** — Security at auth, CSRF, rate limiting, and webhook layers
4. **Operational visibility** — Health endpoints, audit logs, and admin dashboards
5. **Future-ready API** — Backend designed for multi-client consumption
6. **Requirements traceability** — Every capability maps to verification methods

---

## Requirements summary

| Category | Count |
|---|---|
| Functional | 44 |
| Non-functional | 20 |
| **Total** | **64** |

---

## Related documents

- [System Architecture →](03-system-architecture.md)
- [User Workflows →](04-user-workflows.md)
- [Verification & Validation →](06-verification-validation.md)
