# Operations

How the platform is deployed, monitored, maintained, and recovered from incidents.

---

## Infrastructure overview

| Component | Role |
|---|---|
| Frontend (Next.js) | PM2-managed process on Linux VM |
| Backend API (Express) | PM2-managed process on Linux VM |
| Ops monitor | Health polling and error reporting |
| MongoDB | Remote managed database |
| Image storage | Local filesystem on application server |
| Secrets | Centralized secret management (Doppler) |

The platform runs on Linux virtual machines with PM2 process management. MongoDB is hosted remotely. All runtime secrets are injected at build and runtime—never stored in repositories or on-disk environment files.

---

## Deployment approach

### Automated deployment (standard)

Deployments trigger automatically on branch push via GitHub Actions. No manual intervention required for standard releases.

**Backend deploy flow:**

1. Push to production or staging branch
2. CI runs full automated test suite
3. Integration environment coherence validated (Stripe mode, FedEx environment)
4. Code deployed to VM via rsync (excludes dependencies and secrets)
5. Database indexes applied
6. Secrets injected; PM2 reloads API and ops monitor processes
7. Smoke health check verifies endpoints

**Frontend deploy flow:**

1. Push triggers Vitest, lint, typecheck, and production build
2. Build executed with injected secrets
3. Artifacts deployed to VM
4. PM2 reloads frontend process
5. Health verification confirms frontend responds

### Environment management

| Environment | Branch | Integration modes |
|---|---|---|
| Production | Main branch | Live payments, production shipping |
| Staging | Feature branches | Test payments, sandbox shipping |

A temporary testing override flag allows sandbox integrations in production for validation—but must be removed before live commerce.

### Rollback

PM2 maintains process history. Previous release directories are retained on the VM. Rollback involves re-pointing the deployment symlink and reloading processes, followed by health verification.

---

## Monitoring philosophy

### Layered observability

| Layer | Mechanism | Purpose |
|---|---|---|
| **Process health** | PM2 status monitoring | Detect crashed or restarting processes |
| **API health** | Liveness, readiness, and deep health endpoints | Dependency status at multiple depths |
| **Error tracking** | Honeybadger integration | Exception reporting with context |
| **Ops monitor** | Dedicated PM2 process | Continuous health polling and check-ins |
| **Admin dashboard** | System health page | Human-readable dependency status |

### Health endpoints

| Endpoint | Checks |
|---|---|
| Liveness | Process is running |
| Readiness | Database connection |
| Deep health | MongoDB, Clerk, Stripe, FedEx, Postmark, image storage, launch readiness |

The ops monitor polls liveness every minute and deep health every five minutes. Failures trigger alerts.

### What we monitor vs. what we alert on

- **Monitor continuously:** Process status, health endpoints, webhook receipt freshness
- **Alert on:** Unhandled exceptions, dependency failures, PM2 restart spikes
- **Review daily:** Error dashboard for new error groups
- **Review weekly:** Email delivery logs, audit logs, fulfillment queue for stuck orders

FedEx errors are fingerprinted to prevent duplicate alert storms from the same root cause.

---

## Reliability considerations

### Single-server architecture

The platform runs on a single VM without horizontal scaling. This is an accepted tradeoff for V1 scale, mitigated by:

- PM2 auto-restart on process crash
- Ops monitor with external alerting
- Documented rollback procedure
- Health gates before and after every deploy

### Webhook reliability

Order creation depends on Stripe webhook delivery. Mitigations:

- Idempotent processing prevents duplicates on retry
- Success page polls for order by session ID
- Operational monitoring of webhook receipt freshness
- Diagnostic tooling for webhook reconciliation

### Graceful degradation

| Failure | System behavior |
|---|---|
| Postmark unavailable | Operations continue; emails not sent |
| Mapbox unavailable | Manual address entry still works |
| Honeybadger unavailable | System operates; errors not reported |
| FedEx unavailable | Checkout blocked (no rates); admin can enter manual tracking |

---

## Incident response

### Severity levels

| Level | Description | Response time |
|---|---|---|
| P1 — Critical | Site down, payments failing | Immediate |
| P2 — High | Checkout broken, admin inaccessible | Under 1 hour |
| P3 — Medium | Single feature degraded | Under 4 hours |
| P4 — Low | Cosmetic or non-blocking errors | Next business day |

### Common incident playbooks

**Site down:** Check process status → health endpoints → logs → restart or rollback

**Payment failure:** Verify payment provider status → webhook configuration → error dashboard → test in staging

**Shipping failure:** Verify carrier API status → environment configuration → review failed labels → manual tracking fallback

**Authentication failure:** Verify identity provider status → key configuration → webhook delivery

**Email failure:** Check email provider dashboard → delivery logs → test send endpoint

---

## Maintenance practices

### Daily

- Review error dashboard for new error groups
- Check system health for dependency warnings
- Review fulfillment queue for stuck orders

### Weekly

- Review email delivery logs
- Verify webhook receipt freshness
- Review audit logs for unusual activity
- Check process uptime and memory usage

### Monthly

- Full manual regression test suite
- Review secret rotation policy
- Check disk usage for image storage
- Review database storage and index performance
- Apply security dependency patches

### Per release

- All CI tests pass
- Smoke test passes post-deploy
- Deep health all-green
- Spot-check critical user paths
- Monitor errors for 24 hours post-deploy

---

## Backup and recovery

| Asset | Strategy |
|---|---|
| **Source code** | GitHub repositories (source of truth) |
| **Deploy artifacts** | Previous releases retained on VM |
| **Database** | Managed at infrastructure level (provider backups) |
| **UI images** | Local filesystem; admin can re-sync from image library |
| **Secrets** | Centralized secret manager with version history |

Database and image backups are infrastructure responsibilities—not automated in application code. Backup schedules should be verified with the hosting provider.

---

## Operational scripts

Key maintenance operations run via documented scripts:

- Database index application
- Account deletion processing (recommended daily cron)
- Smoke API health verification
- UI asset seeding and synchronization

---

## Related documents

- [Verification & Validation →](06-verification-validation.md)
- [Risks & Tradeoffs →](08-risks-and-tradeoffs.md)
- [System Architecture →](03-system-architecture.md)
