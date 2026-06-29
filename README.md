# Premortem Forecast

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![Claude Code Skill](https://img.shields.io/badge/Claude%20Code-Skill-7C3AED.svg)](https://docs.claude.com/en/docs/claude-code)
[![Codex Skill](https://img.shields.io/badge/Codex-Skill-111827.svg)](agents/openai.yaml)

A skeptical, evidence-based **premortem and failure-forecast skill** for AI coding agents. Point it at a plan, design, pull request, codebase, launch, or live system and it works backward from "this already failed" to find *how* it would fail and what would have prevented it — without inventing risks that don't survive verification.

It's the opposite of a rubber stamp. The job is to break the plan before reality does.

---

## What it is

`premortem-forecast` is a portable skill (a single `SKILL.md` plus references) that instructs an agent to run a structured premortem: assume the target has failed in a serious but plausible way, trace the causal chain, and separate **real, verified risks from false alarms**.

Most "risk reviews" fail in one of two directions: they either rubber-stamp everything, or they cry wolf at every pattern that *looks* scary. This skill is built around a two-pass evidence rule — collect candidate risks first, then verify each one against actual context (code, tests, configs, mitigations, scope) before escalating it.

The output is not a wall of criticism. Every significant failure mode comes with detection and prevention, a priority, and a clear go / no-go verdict.

## Why it's different

- **Evidence before alarm.** Nothing is flagged as a real threat from pattern matching alone. Every escalated risk must cite where it lives and which mitigation was checked-for and not found.
- **It tells you what's *not* a problem too.** "Paper Tigers" and "False alarms" are reported when verification changed the conclusion, so you can trust what's left.
- **It surfaces the unspoken.** "Elephants" capture ownership gaps, incentive mismatches, timeline pressure, and other things people avoid saying out loud.
- **It ends with action, never just criticism.** Every material failure gets at least one preventive control and a priority (P0–P3).
- **Two depths.** A fast Quick pass for PRs and small changes; a Deep pass (with an expanded taxonomy) for launches, migrations, security/privacy, AI agents, and money movement.

## Core concepts

### Risk labels

| Label | Meaning |
|---|---|
| **Tiger** | Verified threat that will likely hurt the project if unaddressed. Requires evidence *and* a checked-missing mitigation. |
| **Paper Tiger** | Looks risky at first, but context, scope, existing mitigation, or tests make it acceptable — with the reason stated. |
| **Elephant** | An important *unspoken* concern: ownership, incentives, team capability, timeline pressure, politics, strategic mismatch. |
| **False alarm** | An initial concern disproven by verification. Mentioned only when it materially improves trust in the review. |
| **Unverified** | Plausible but not yet proven; flagged as a blind spot, not escalated to a Tiger. |

### Modes

- **Quick** — PRs, localized changes, small plans, or "give me a fast pass." Focuses on the biggest risks, rollback, dependencies, tests, and false positives.
- **Deep** — before implementation, launch, migration, public release, high-stakes decisions, security/privacy work, AI agents, automation, money movement, sensitive data, or broad architecture changes. Loads the full [risk taxonomy](references/risk-taxonomy.md).

### Priorities

| Priority | Meaning |
|---|---|
| **P0** | Fix before continuing. |
| **P1** | Fix before launch or scale. |
| **P2** | Fix for robustness after immediate blockers. |
| **P3** | Future improvement or accepted risk with monitoring. |

Risks that are silent, hard to reverse, compounding, security/privacy-sensitive, or ownerless get promoted.

## Installation

This is a portable skill for [Claude Code](https://docs.claude.com/en/docs/claude-code) and Codex-style agent runtimes. Skills are auto-discovered from each runtime's skills directory.

**Claude Code, user-level (available in every project):**

```bash
git clone https://github.com/anobyzy/premortem-forecast.git ~/.claude/skills/premortem-forecast
```

**Claude Code, project-level (committed alongside a repo):**

```bash
git clone https://github.com/anobyzy/premortem-forecast.git .claude/skills/premortem-forecast
```

**Codex, user-level on Windows PowerShell:**

```powershell
git clone https://github.com/anobyzy/premortem-forecast.git "$HOME\.codex\skills\premortem-forecast"
```

**Codex, user-level on macOS/Linux shells:**

```bash
git clone https://github.com/anobyzy/premortem-forecast.git ~/.codex/skills/premortem-forecast
```

The final folder name must be `premortem-forecast` so it matches the skill's `name`. Claude Code and Codex load it automatically on the next session. Codex also reads the interface metadata in [`agents/openai.yaml`](agents/openai.yaml).

## Usage

Once installed, the skill triggers automatically when you ask for a premortem, risk review, or failure analysis. You can also invoke it explicitly.

**Natural language:**

```
Run a premortem on this migration plan before I start.
Stress-test this PR for real risks versus false alarms.
What's the most plausible way this launch fails in 90 days?
```

**Explicit:**

```
Use the premortem-forecast skill (Deep mode) on this architecture.
```

The agent picks Quick vs Deep from the stakes, or you can name the mode. Point it at anything: a plan, a diff, a directory, a design doc, a launch checklist, or an already-shipped system (it inspects actual behavior before judging).

## What you get

The skill produces a structured report (it adapts the shape if you ask for brevity):

```markdown
# Strategic Premortem

## 1. Target
Objective, flow, dependencies, success/failure criteria, blast radius.

## 2. Failure Thesis
Most plausible serious failure and why it would happen.

## 3. Verified Risk Register
| Type | Risk | Evidence | Causal path | Severity/Priority | Detection | Prevention |

## 4. Blind Spots And Assumptions
| Item | Why it matters | How to verify | Protection |

## 5. Break Tests
| Test | Simulate | Expected result | Fails if |

## 6. Prevention Plan
| Action | Area | Priority | Effort | Impact | Consequence if ignored |

## 7. Verdict
Classification, robustness score, failure probability if unchanged,
success probability after fixes, top 3 mandatory corrections, recommended decision.
```

## When to use it

- **Before execution** — turn a plan into a plan-with-mitigations before writing code.
- **During plan/PR review** — a Quick pass that catches the silent, hard-to-reverse failures.
- **Before launch / migration / public release** — a Deep pass with break tests and launch gates.
- **After implementation** — review actual behavior: interfaces, schemas, migrations, retries, idempotency, observability, rollback, duplicate side effects.
- **For AI agents & automations** — dedicated checks for prompt injection, untrusted context, data exfiltration, unsafe tool calls, stale retrieval, hallucinated citations, invalid structured output, eval gaps, and cost spikes.

## How it works

Internally the skill runs an 8-phase protocol: **frame the target → write the failure thesis (30/90/180 days out) → verify failure modes → expose blind spots → test fragile assumptions → design break tests → plan prevention → give a verdict.** Findings are scored on severity, likelihood, detectability, and reversibility, then mapped to P0–P3.

The [`references/risk-taxonomy.md`](references/risk-taxonomy.md) file is loaded for Deep reviews and carries the verification checklist, scoring rubric, failure heuristics (silent failures, fragile dependencies, scale traps, automation traps, AI traps, human traps), a full risk taxonomy, and a break-test catalog.

## Repository structure

```
premortem-forecast/
├── SKILL.md                      # The skill: modes, labels, protocol, output shape
├── references/
│   └── risk-taxonomy.md          # Deep-mode checklist, scoring, heuristics, break tests
├── agents/
│   └── openai.yaml               # Interface for OpenAI/Codex-style agent runtimes
├── README.md
└── LICENSE
```

## Cross-platform

The skill content is plain Markdown and runtime-agnostic. For OpenAI/Codex-style agents, [`agents/openai.yaml`](agents/openai.yaml) declares the display name, short description, and a default invocation prompt (`$premortem-forecast`). The same `SKILL.md` body is the source of truth across runtimes.

## Contributing

Issues and pull requests are welcome. Good contributions:

- Add break-test cases or failure heuristics that are *generally* applicable.
- Tighten verification rules so fewer false alarms slip through as Tigers.
- Keep the skill runtime-agnostic — no hard dependency on a single agent platform.

Please keep changes evidence-driven: if you add a new "always check this" rule, explain the real failure it prevents.

## License

[MIT](LICENSE)
