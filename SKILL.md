---
name: premortem-forecast
description: "Strategic premortem and failure forecast for plans, designs, roadmaps, pull requests, codebases, launches, AI-agent workflows, automations, products, processes, campaigns, architectures, and implemented systems. Use before execution, before launch, during plan review, or after implementation to stress-test plausible failure modes, blind spots, fragile assumptions, dependencies, rollback gaps, security/privacy issues, AI/LLM risks, missing tests, observability gaps, and prevention controls. Separates verified Tigers, Paper Tigers, Elephants, and false alarms; produces a risk register, break-test plan, mitigation checklist, and go/no-go verdict."
---

# Premortem Forecast

Use this skill as a skeptical, technical, preventive review. Assume the plan or implementation has failed in a serious but plausible way, then work backward to find how it failed and what would have prevented it.

The job is not to validate the plan. The job is to break it before reality does, without inventing risks that do not survive verification.

## Modes

- **Quick**: use for PRs, localized changes, small plans, or when the user asks for a fast pass. Focus on the biggest risks, rollback, dependencies, tests, and false positives.
- **Deep**: use before implementation, launch, migration, public release, high-stakes decisions, security/privacy work, AI agents, automation, money movement, sensitive data, or broad architecture changes. Load `references/risk-taxonomy.md`.
- If scope is unclear, choose Quick for narrow artifacts and Deep for high-stakes or cross-system work.

## Risk Labels

- **Tiger**: verified threat that will likely hurt the project if not addressed. Requires evidence and a checked-missing mitigation.
- **Paper Tiger**: looks risky at first, but context, scope, existing mitigation, or tests make it acceptable. Explain why.
- **Elephant**: important unspoken concern such as ownership, incentives, team capability, timeline pressure, politics, or strategic mismatch.
- **False alarm**: initial concern disproven by verification. Mention only when it materially improves trust in the review.

## Evidence Rules

- Use a two-pass process: first collect potential risks, then verify before escalating.
- Do not flag Tigers from pattern matching alone.
- Before marking a Tiger, verify relevant context, existing mitigations, scope, fallback behavior, tests, and whether the issue is production-relevant.
- For code, inspect nearby code, callers, configs, tests, error handling, feature flags, dev-only paths, and runtime assumptions before judging.
- For plans, inspect goals, non-goals, constraints, owners, dependencies, success criteria, rollback, testing, monitoring, and launch gates.
- If evidence is missing or inconclusive, label it as an unverified risk or blind spot, not a Tiger.
- Every Tiger must include: risk, evidence/location, severity or priority, causal chain, mitigation checked and not found, detection, and prevention.

## Protocol

Run these phases. Collapse only when the user explicitly asks for brevity.

1. **Frame the target**: objective, expected flow, dependencies, success criteria, failure criteria, owners, constraints, and blast radius.
2. **Write the failure thesis**: assume it failed 30/90/180 days later; describe the most plausible bad outcome, root cause, ignored signal, and silent failure path.
3. **Verify failure modes**: classify Tigers, Paper Tigers, Elephants, false alarms, and unverified risks. Keep only plausible causal chains.
4. **Expose blind spots**: technical, operational, human, financial, legal/compliance, reputation, scale, maintenance, data, and external dependency gaps.
5. **Test assumptions**: list fragile assumptions, why each is fragile, how to verify it, and how to protect against it being false.
6. **Design break tests**: simulate load, API failure, invalid/missing/stale data, latency, duplicate execution, rollback, cost spike, abuse, observability failure, and AI-specific failures when relevant.
7. **Plan prevention**: prefer eliminating risk, reducing likelihood, detecting earlier, limiting blast radius, rehearsing recovery, and defining stop criteria.
8. **Give a verdict**: approved, approved with caveats, risky and needs fixes, fragile and not recommended, or not viable now. Include robustness score, failure probability range, top fixes, and recommended decision.

## Priority

- **P0**: fix before continuing.
- **P1**: fix before launch or scale.
- **P2**: fix for robustness after immediate blockers.
- **P3**: future improvement or accepted risk with monitoring.

Promote risks that are high-impact, silent, hard to reverse, compounding, security/privacy-sensitive, or ownerless.

## Output Shape

Use this structure unless the user asks otherwise:

```markdown
# Strategic Premortem

## 1. Target
Objective, flow, dependencies, success/failure criteria, blast radius.

## 2. Failure Thesis
Most plausible serious failure and why it would happen.

## 3. Verified Risk Register
| Type | Risk | Evidence | Causal path | Severity/Priority | Detection | Prevention |
|---|---|---|---|---|---|---|

## 4. Blind Spots And Assumptions
| Item | Why it matters | How to verify | Protection |
|---|---|---|---|

## 5. Break Tests
| Test | Simulate | Expected result | Fails if |
|---|---|---|---|

## 6. Prevention Plan
| Action | Area | Priority | Effort | Impact | Consequence if ignored |
|---|---|---|---|---|---|

## 7. Verdict
Classification, robustness score, failure probability if unchanged, success probability after fixes, top 3 mandatory corrections, recommended decision.
```

Include Paper Tigers and false alarms in the risk register or a short note when verification changed the conclusion.

## Implementation Reviews

When reviewing an implementation, inspect actual behavior before judging. Check interfaces, schemas, migrations, configs, permissions, retries, rate limits, idempotency, observability, tests, rollback, duplicate side effects, and mismatch between documented intent and code. Treat passing tests as evidence, not proof.

## AI-Agent Reviews

For AI agents, LLM apps, RAG systems, or tool-using automations, check prompt injection, untrusted context, data exfiltration, unsafe tool calls, permission boundaries, stale retrieval, hallucinated citations, invalid structured output, context loss, eval gaps, model drift, cost spikes, fallback behavior, human approval, and tool-call observability.

## Plan Updates

If the user asks to update a plan, add a `Risk Mitigations (Premortem)` section with addressed Tigers, accepted risks, unresolved Elephants, break tests, and launch gates. Re-run a Quick pass after edits to verify the mitigations.

## Quality Bar

A good premortem tells the user what can go wrong, why, how to detect it, how to prevent or contain it, what to fix first, what is acceptable, and what blocks progress. Never end with criticism alone; every significant failure needs at least one preventive action.

