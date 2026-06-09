# Premortem Reference

Load this file for Deep reviews, high-stakes work, implemented systems, security/privacy exposure, AI agents, automation, money movement, sensitive data, public launches, unclear scope, or when the first pass feels too generic.

Use it as a checklist, not a template. Report only risks that matter.

## Verification Checklist

Before escalating a finding to Tiger:

- **Context read**: inspect the relevant plan section, code region, caller, config, test, or runtime path. For code, read nearby context instead of judging one line.
- **Mitigation check**: look for validation, fallback, try/catch, retries, backoff, idempotency, circuit breaker, feature flag, rollback, test, alert, permission check, or documented non-goal.
- **Scope check**: confirm the risk is in scope for the plan, change, environment, user path, or launch.
- **Production check**: separate real production paths from tests, examples, local scripts, dev-only code, mocks, and unreachable paths.
- **Evidence check**: record the evidence or location and what mitigation was checked but not found.

Result labels:

- `Tiger`: verified threat; include mitigation checked and not found.
- `Paper Tiger`: scary at first, but existing mitigation or scope makes it acceptable.
- `Elephant`: unspoken strategic, team, incentive, ownership, or timeline risk.
- `False alarm`: initial concern disproven by verification.
- `Unverified`: plausible but not proven; needs investigation before escalation.

## Quick Checklist

Use for PRs, localized changes, and fast plan reviews:

1. What is the single biggest thing that could go wrong?
2. Which dependency can fail or change?
3. Can this be rolled back safely?
4. Which edge case is not covered by tests?
5. Which requirement, owner, or success criterion is unclear?
6. What looked risky but was verified as safe?

## Deep Checklist

Use before implementation, launch, migration, or broad architecture changes.

- **Technical**: 10x/100x load, latency, data consistency, migrations, auth, injection, error handling, concurrency, idempotency.
- **Integration**: breaking changes, external services, contracts, webhooks, rate limits, sandbox coverage, migration path, rollback, feature flags.
- **Process**: requirements, stakeholder input, non-goals, ownership, support load, maintenance burden, tech debt.
- **Testing**: unit gaps, integration tests, load tests, manual test plan, adversarial cases, recovery drills.
- **Operations**: monitoring, alert ownership, runbooks, backups, degraded mode, incident communications.
- **AI**: prompt injection, retrieval quality, structured output validation, model drift, hallucination, tool permissions, eval coverage.

## Scoring

Use 1-5 scores when ranking helps:

- **Severity**: 1 cosmetic, 2 localized inconvenience, 3 material user/business harm, 4 major outage/legal/security/revenue harm, 5 irreversible or existential harm.
- **Likelihood**: 1 rare, 2 plausible under stress, 3 plausible in normal use, 4 likely, 5 already happening.
- **Detectability**: 1 obvious, 2 normal monitoring, 3 investigation needed, 4 delayed/indirect, 5 silent until damage.
- **Reversibility**: 1 trivial rollback, 2 small cleanup, 3 coordinated recovery, 4 costly/reputation-damaging, 5 irreversible.

Priority mapping:

| Impact | Probability | Priority |
|---|---|---|
| Critical | High | P0 |
| Critical | Medium | P0 or P1 |
| High | High | P1 |
| High | Medium | P1 |
| Medium | High | P1 or P2 |
| Low | High | P2 |
| Low | Low | P3 |

Promote silent, irreversible, compounding, security/privacy-sensitive, or ownerless risks.

## Failure Heuristics

### Silent failures

- System appears to work but produces wrong, stale, duplicated, unauthorized, or unsafe output.
- Workflow continues after corrupted, missing, partial, or invalid data.
- Errors are hidden in logs, swallowed, or collapsed into generic success.
- Wrong content is published, sent, billed, stored, or acted on without review.

### Fragile dependencies

- External API, webhook, scraping target, credential, model, database, browser, queue, file store, payment/email provider, manual config, or single server.
- Vendor limits, pricing, policy, schema, uptime, or behavior assumed stable.

### Scale traps

- Works at 10 items and fails at 10,000.
- Sequential latency compounds.
- Cost grows nonlinearly.
- Retries create duplicate side effects.
- Queues, logs, cache, or storage become operational risks.

### Automation traps

- Missing idempotency, deduplication, schema validation, timeout, backoff, lock, state tracking, rollback, reconciliation, alerting, pause/resume, or replay safety.
- Partial success is reported as full success.

### AI traps

- Hallucination, invalid JSON, fragile prompt, ambiguous instruction, stale retrieval, poisoned context, fabricated sources, ignored constraints, over-trust, model drift, prompt injection, context loss, or using a model where deterministic rules are safer.

### Human traps

- Operator confusion, manual steps, missing checklist, missing owner, no escalation path, no emergency process, or review after irreversible action.

## Risk Taxonomy

- **Strategy/product**: vague goal, wrong user, weak adoption incentives, hidden scope, no non-goals, deadline-driven quality debt.
- **Requirements**: assumptions stated as facts, "temporary" manual work, missing negative cases, ambiguous stakeholder language.
- **User/workflow**: correction/cancellation/recovery gaps, invisible status, duplicated entry, incentives to bypass, accessibility/language/device gaps.
- **Architecture**: unclear ownership, weak contracts, state divergence, unsafe defaults, brittle parsing, concurrency/order/clock assumptions.
- **Data/migration**: old records, nulls, duplicates, backfills, rollback, PII leakage, retention/deletion/audit gaps, valid-but-wrong data.
- **Integration**: timeouts, partial success, rate limits, revoked credentials, vendor changes, missing contract tests or sandbox fixtures.
- **Security/privacy/abuse**: UI-only auth, tenant leaks, injection, path traversal, SSRF, XSS, unsafe deserialization, command execution, data exfiltration, missing audit trails.
- **Reliability/ops**: correctness unmonitored, noisy alerts, untested restore, no degraded mode, runbook depends on one person, unclear incident authority.
- **Cost/performance**: unbounded fan-out, repeated model calls, large queries, scraping loops, unbounded storage, missing quotas or kill switches.
- **Organization**: boundary ownership gaps, future cleanup without owner/date, launch readiness disagreement, support absorbs hidden complexity.
- **Rollout/recovery**: no canary, feature flag, migration pause, rollback, kill criteria, or prepared communications.

## Break-Test Catalog

| Test | Simulate | Expected result | Fails if |
|---|---|---|---|
| Load | 10x volume, burst traffic, large batch | Degrades safely within limits | Timeouts, queue growth, cost spike, data loss |
| API outage | Dependency unavailable | Safe fallback, retry policy, alert | Silent skip, crash loop, duplicate side effects |
| Malformed API | Wrong shape, nulls, partial success | Validation rejects or repairs safely | Bad data accepted as valid |
| Rate limit | Vendor throttling | Backoff, queueing, visible status | Retry storm or permanent failure |
| Invalid input | Wrong type, hostile content | Validation blocks with recovery | Unsafe processing or vague error |
| Missing/stale data | Required fields absent or cache old | Explicit stop, freshness check, warning | Fabricated defaults or stale output |
| Duplicate execution | Same job/webhook/tool call twice | Idempotent result | Double billing, duplicate send, repeated mutation |
| Partial failure | Step fails after side effects | Reconciliation or rollback | Reports success or cannot recover |
| Rollback | Revert deploy, config, prompt, model, migration | Tested path works | Rollback impossible or corrupts state |
| Cost spike | 10x usage or adversarial large input | Budget guard, quota, kill switch | Spend grows unnoticed |
| Security | Permission bypass, injection, exfiltration | Blocked and logged | Unauthorized action or data leak |
| Observability | Failure without manual inspection | Alert names symptom and owner | Users discover it first |
| Recovery | Restore backup or replay queue | Data/service restored safely | Backup unusable or replay unsafe |
| AI hallucination | Unsupported question | Refusal, uncertainty, citations | Confident fabrication |
| AI prompt injection | Untrusted content gives instructions | Treated as untrusted data | Tool call, leak, or policy bypass |
| AI invalid output | Malformed structured output | Schema validation and fallback | Parser crash or unsafe default |
| AI context loss | Long workflow or summary | Critical constraints retained | Agent forgets permissions/goals/limits |

