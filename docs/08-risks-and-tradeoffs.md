# Risks and Tradeoffs

Known limitations, engineering tradeoffs, and how risks are monitored and mitigated.

---

## Risk management approach

Risks were identified during architecture design and tracked through release. Each risk includes likelihood, impact, mitigation strategy, and residual risk assessment. Some risks are accepted as deliberate tradeoffs for V1 scope.

---

## High-priority risks

### Webhook delivery failure

| Attribute | Detail |
|---|---|
| **Risk** | Stripe webhooks fail to deliver; orders not created despite successful payment |
| **Impact** | Critical — customer charged but no order record |
| **Mitigation** | Idempotent webhook processing; success page polls by session ID; operational monitoring of webhook freshness; manual reconciliation from payment dashboard |
| **Residual** | Medium — requires active operational monitoring |

### Single-server architecture

| Attribute | Detail |
|---|---|
| **Risk** | Frontend, backend, and storage on one VM with no horizontal scaling |
| **Impact** | High — VM failure takes entire platform offline |
| **Mitigation** | PM2 process monitoring; ops monitor with alerts; auto-restart; documented rollback |
| **Residual** | High — no redundancy at application layer |

### FedEx API unavailability

| Attribute | Detail |
|---|---|
| **Risk** | Carrier outage prevents rate quotes and label creation |
| **Impact** | High — checkout blocked or fulfillment delayed |
| **Mitigation** | Sandbox rate estimation fallback; manual tracking entry; failed label status for retry; health check surfaces carrier status |
| **Residual** | Medium — checkout cannot complete without rates |

### Environment misconfiguration

| Attribute | Detail |
|---|---|
| **Risk** | Production deployed with test payment keys or sandbox shipping |
| **Impact** | Critical — no real revenue; invalid shipping labels |
| **Mitigation** | Startup launch readiness validation; CI/CD coherence checks; test mode banner in UI; runtime status endpoint |
| **Residual** | Low — explicit override required for testing mode |

---

## Accepted tradeoffs

These are deliberate architectural decisions, not oversights:

| Tradeoff | Reason accepted | Future path |
|---|---|---|
| **Stripe-hosted checkout** | PCI scope reduction; industry-standard UX | Limited customization accepted |
| **US-only shipping** | FedEx integration scope; business focus | International via quote workflow |
| **Quote-to-order gap** | Custom projects need manual pricing | Automated invoicing planned |
| **No real-time inventory** | Made-to-order engineering model | Admin availability flags sufficient for V1 |
| **Local filesystem storage** | Simpler V1 deployment | Azure Blob client ready for migration |
| **Clerk as sole identity provider** | Security and maintenance reduction | No fallback by design |

---

## Engineering tradeoffs

### Custom platform vs. SaaS builder

| Gained | Sacrificed |
|---|---|
| Complete workflow control | Higher development cost |
| Data ownership | Full security/compliance responsibility |
| Future expansion flexibility | Infrastructure management burden |

### Separate frontend and backend

| Gained | Sacrificed |
|---|---|
| Multi-client API reuse | Two deploy pipelines |
| Clear separation of concerns | CORS and proxy complexity |

### MongoDB without ORM

| Gained | Sacrificed |
|---|---|
| Schema flexibility | Application-level enforcement |
| Rapid iteration | Manual index management |

### AI-assisted development

| Gained | Sacrificed |
|---|---|
| Accelerated module implementation | Potential for subtle inconsistencies |
| Faster V1 delivery | Requires thorough review of generated code |

**Mitigation for AI-assisted code:** Automated test suite, manual functional tests, error monitoring, and engineer review of all critical paths before merge.

---

## System limitations

### Architectural

| Limitation | Impact | Future direction |
|---|---|---|
| Monolithic VM deployment | No horizontal scaling | Container orchestration |
| No CDN | Assets served from origin | Edge CDN integration |
| No message queue | Synchronous webhook processing | Event-driven architecture |
| No caching layer | All requests hit database | Redis for catalog and sessions |

### Functional

| Limitation | Notes |
|---|---|
| No subscription billing | One-time payments only |
| No multi-currency | USD only |
| No product reviews | No rating system |
| No live chat | Async support tickets only |
| No partial shipment | Single shipment per order |
| No automated quote pricing | Admin sets amounts manually |
| No automated quote-to-checkout | Accepted quotes need manual billing |

### Security

| Limitation | Notes |
|---|---|
| No WAF | Rate limiting only at application layer |
| No customer action audit | Audit logs cover admin actions only |
| No documented penetration test | Security via design patterns and code review |

### Scalability thresholds

| Constraint | Approximate limit |
|---|---|
| Single VM | Concurrent users limited by VM resources |
| Global rate limit | 800 requests per 15 minutes per IP |
| JSON body limit | 25 MB |
| Local image storage | VM disk capacity |

---

## Risk monitoring indicators

| Indicator | Where to check | Risk addressed |
|---|---|---|
| Webhook receipt age | Database collection | Payment webhook failure |
| Carrier health status | Deep health endpoint | Shipping outage |
| Process restart count | Process manager status | Server instability |
| Error rate trends | Error monitoring dashboard | Code quality issues |
| Test mode banner visible | Frontend UI | Environment misconfiguration |
| Disk usage | Server filesystem | Asset storage capacity |
| Orders without shipment > 48h | Admin exceptions bucket | Fulfillment delays |

---

## Decision-making lens

When evaluating future changes, these questions frame the tradeoff analysis:

1. **Does this increase or decrease platform ownership?**
2. **What is the blast radius if this component fails?**
3. **Can we verify this change before production exposure?**
4. **Does this serve current scale or anticipated scale?**
5. **What operational burden does this add?**

The V1 platform consciously optimized for ownership, flexibility, and verifiability over maximum scalability and feature completeness. Future evolution should address the highest-priority residual risks—particularly single-server architecture and quote-to-order automation—while preserving the architectural foundations established in Version 1.

---

## Related documents

- [Technical Decisions →](05-technical-decisions.md)
- [Operations →](07-operations.md)
- [Lessons Learned →](09-lessons-learned.md)
