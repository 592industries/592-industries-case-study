# Verification and Validation

How the platform was tested, validated, and confirmed ready for production release.

---

## Systems engineering approach

Verification was structured in layers, each building confidence before the next:

| Level | Method | When applied |
|---|---|---|
| **Unit** | Automated tests (Node test runner, Vitest) | Every CI build |
| **Integration** | Automated tests with mocked external services | Every CI build |
| **Launch readiness** | Environment validation gates | Deploy to production |
| **Smoke** | Post-deploy API health checks | After every deploy |
| **Manual / functional** | Human-operated workflow tests | Pre-release, regression |
| **Operational** | Monitoring and health dashboards | Continuous in production |

This layered approach ensures requirements are traceable from specification through automated verification to operational monitoring.

---

## Automated testing

### Backend test suite

**Runner:** Node.js built-in test framework  
**Trigger:** Every backend CI/CD pipeline run before deployment

| Coverage area | Examples |
|---|---|
| Admin RBAC | Role normalization, capability checks, staff access restrictions |
| Checkout | Address validation, rate options, session creation, Zod validation |
| FedEx integration | OAuth client, payload construction, sandbox rates, label automation |
| Webhooks | Stripe and Clerk webhook processing with signature verification |
| Security | CORS allowlist, path traversal protection, response envelope |
| Launch readiness | Production environment validation |
| Email | Template rendering, environment validation |

**Total:** 34+ automated backend tests

### Frontend test suite

**Runner:** Vitest  
**Trigger:** Every frontend CI/CD pipeline run before deployment

| Coverage area | Examples |
|---|---|
| HTTP client | CSRF flow, envelope parsing, retry logic |
| Clerk configuration | URL configuration, portal mode |
| UI assets | Asset URL construction |

Critical integration layer regressions are caught before deploy.

---

## Launch readiness gates

Production startup validates environment coherence before accepting traffic:

| Check | Failure action |
|---|---|
| Required environment variables present | Process exit with launch readiness error |
| Clerk key modes match (secret + publishable) | Startup error |
| Live Stripe keys in production | Startup error (unless testing override) |
| Production FedEx environment in production | Startup error (unless override) |
| FedEx API base URL configured | Missing configuration error |

This prevents accidental production deployment with test credentials—a critical safeguard for revenue-generating systems.

---

## CI/CD verification pipeline

### Backend pipeline

```
Code push → npm test → Environment coherence check → Deploy to VM
    → Apply database indexes → PM2 reload → Smoke health check
```

### Frontend pipeline

```
Code push → Vitest → Lint + typecheck → Build with secrets
    → Deploy to VM → PM2 reload → Health verification
```

Both pipelines use concurrency groups to prevent overlapping deploys per branch.

---

## Manual functional test procedures

Seven manual test procedures cover end-to-end workflows before release:

### Commerce flow

| Step | Expected result |
|---|---|
| Browse shop | Products displayed with prices |
| Add to cart and checkout | Shipping form shown |
| Enter valid US address | Address validated |
| Select shipping rate | Rates displayed with costs |
| Complete test payment | Redirect to success page |
| Check customer dashboard | Order appears with paid status |
| Check admin fulfillment queue | Order visible for processing |

### Custom quote flow

| Step | Expected result |
|---|---|
| Submit quote with all fields | Status submitted; confirmation email |
| Admin sets amount and sends | Status quoted; customer email received |
| Customer views quote detail | Quoted amount visible |

### Fulfillment flow

| Step | Expected result |
|---|---|
| Admin updates fulfillment status | Status changed; audit log entry |
| Create FedEx shipment | Label generated; tracking assigned |
| Customer checks tracking | Timeline shows shipping events |

### Support, account, and RBAC

Additional procedures verify support ticket lifecycle, account privacy controls, and role-appropriate admin access for staff, finance, and owner roles.

---

## Health check verification

| Endpoint | Pass criteria |
|---|---|
| Liveness | HTTP 200, status ok |
| Readiness | HTTP 200, database connected |
| Deep health | All dependencies healthy |

### Deep health dependencies

| Dependency | Check method |
|---|---|
| MongoDB | Connection ping |
| Clerk | API key validation |
| Stripe | API key validation |
| FedEx | OAuth token acquisition |
| Postmark | Server token validation |
| Local image storage | Directory writable |
| Launch readiness | Environment validation pass |

An ops monitor process polls health endpoints continuously and reports failures to error monitoring.

---

## Acceptance criteria for V1 release

| Criterion | Verification |
|---|---|
| All automated tests pass in CI | GitHub Actions green |
| Launch readiness passes in production | Clean startup |
| Smoke test passes post-deploy | Health script exit 0 |
| Manual functional tests pass | Sign-off on all seven procedures |
| Deep health returns all-green | Dependency check endpoint |
| No critical errors in 24h post-deploy | Ops monitor review |
| Webhooks receiving events | Recent webhook receipt entries |

---

## Regression strategy

| Trigger | Scope |
|---|---|
| Every push to main or feature branches | Full automated test suite |
| Pre-production deploy | All manual functional procedures |
| Post-deploy | Smoke test + deep health |
| Monthly | Full manual regression suite |
| After integration changes | Affected procedures + integration tests |

---

## Validation philosophy

Verification was not an afterthought—it was embedded in the development process:

1. **Requirements defined verification methods** during specification
2. **Automated tests mapped to requirements** for continuous confidence
3. **Launch gates prevented misconfiguration** before production exposure
4. **Manual procedures covered user journeys** that automation cannot fully simulate
5. **Operational monitoring provided ongoing validation** in production

This approach reflects systems engineering methodology: verify at the level appropriate to the risk, and maintain traceability from requirement to test to operational indicator.

---

## Related documents

- [Product Requirements →](02-product-requirements.md)
- [Operations →](07-operations.md)
- [Risks & Tradeoffs →](08-risks-and-tradeoffs.md)
