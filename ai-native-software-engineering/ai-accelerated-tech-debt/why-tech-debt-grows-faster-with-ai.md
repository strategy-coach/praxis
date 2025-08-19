# Why Technical Debt Grows Faster With AI‑Generated Code (and how to beat it)

AI makes software faster to write, not faster to understand. Debt grows when the rate of code creation exceeds the rate of shared comprehension, tests, and runtime verification. AI is a force multiplier on both sides of that equation. Treat AI like an ultra‑productive junior team that never sleeps: it will do exactly what you asked—plus a few things you didn’t notice you asked.

## Why AI accelerates technical debt

1. Throughput > comprehension. Models generate large diffs quickly. Review bandwidth and architectural memory don’t scale at the same pace, so implicit decisions leak into production.
2. Prompt drift. Slight wording changes or tool availability shifts alter outputs. Without prompt pinning and provenance, the same “instruction” yields divergent codebases over time.
3. Pattern oscillation. Models flip between idioms (framework choice, error handling style, naming) across files. Inconsistency is a multiplier on maintenance cost.
4. Undeclared dependency sprawl. AI tends to “reach for” libraries. Each new import adds surface area, transitive risk, and patch cadence the team must absorb.
5. Hidden coupling via context windows. The model infers connections from transient context rather than explicit interfaces, producing modules that work only under the original prompt’s assumptions.
6. Non‑determinism in scaffolds. Regenerating “the same” function later produces variants that subtly break callers, especially when tests are missing or brittle.
7. Documentation latency. AI can write docs, but nobody prompts for it by default. Undocumented behavior is indistinguishable from bugs.
8. Over‑fitted happy paths. Generated code often mirrors example inputs. Edge cases and failure modes are under‑represented unless explicitly demanded.
9. AI supply chain risk. Different models, versions, or toolchains produce different semantics. If you don’t capture “who/what generated this,” you can’t reproduce fixes.
10. “Free code” bias. When code is cheap, deletion discipline falls. Accretive features outrun consolidation, and entropy wins.

## First line of defense: tests as executable contracts

Think of tests as the price of merge. Without them, AI increases code volume faster than you can validate intent.

Minimum test standard (per module or service):

* Contract tests: validate public APIs, invariants, and error semantics (not just happy path).
* Property‑based tests: encode general truths (e.g., idempotence, monotonicity, conservation) so future generations can’t regress them.
* Fuzz tests on parsers, serializers, and boundary inputs.
* Golden tests for stable IO when refactors are frequent.
* Mutation testing budget: ensure tests actually fail when behavior changes.
* Test determinism CI check: re‑run flaky suites until stable or block merge.

Policy:

* No new dependency without a dependency test (prove you can mock/isolate it).
* No behavior change PR without at least one failing test first, then a fix commit.
* Coverage is a floor, not a goal; assert on behavior, not line counts.

## Maintainability prompts: your new “Standards of Performance”

Treat prompts like lint rules for the model. Add these to the top of every generation request (and bake them into tools, not human memory).

Core maintainability prompt (template):

```
You are contributing to a long-lived codebase.
Constraints:
- Prefer standard library; if adding a dependency, justify in a 1-line comment and provide a standard-lib alternative.
- Max function length 40 LOC; extract helpers; keep cyclomatic complexity < 10.
- Public interfaces first; private details second. Emit a minimal interface docstring with 1-2 usage examples.
- State assumptions and invariants as comments and executable assertions.
- Output a companion unit test file FIRST (failing test if a bugfix), then code.
- Include a 5-line “Delete Plan”: what we can remove if requirements change.
- Add a file header with: AI-Origin, Model, PromptFingerprint, Date, Owner, TTL.
```

Diff‑guard prompt (used for refactors):

```
Task: Behavior-preserving refactor. 
Do not change public signatures or error types.
Emit a "Behavior Equivalence Notes" block listing any risk of semantic change.
```

Dependency gate prompt:

```
If you propose any new package, also emit: 
- "Why stdlib is insufficient" (1-3 bullets)
- "Blast radius of updates" (transitive deps, known CVEs)
- "Mocking strategy" (how tests isolate it)
```

Test‑first bugfix prompt:

```
Write a failing unit test that reproduces the reported bug with minimal fixtures.
Only after the failing test, submit the smallest code change to pass the test.
```

## Under‑the‑radar tactics that cut AI debt

1. Prompt lockfiles. Check in `PROMPT.lock` next to generated modules capturing the exact instruction, tool list, and model/version. Treat it like `package-lock.json`.
2. TTL for generated code. Every AI file gets a `TTL:` header (e.g., 90 days). A weekly job opens an issue when TTL expires: “Revalidate or delete.” Debt doesn’t get to hide.
3. Code rental vs. code ownership. Label directories as RENTAL (ephemeral scaffolds behind flags) vs OWNED (core domain). RENTAL code can be deleted without ceremony.
4. Quarantine branches. All net-new AI components land in `ai/quarantine/*` with strict flags and no external dependency additions until passing a 2‑week burn‑in.
5. Debt budget per PR. CI computes a debt delta (complexity, dependencies, public API surface). PRs must fit a sprint‑level budget; overshoot requires explicit waiver.
6. Two‑model verification. Generate with Model A, independently review with Model B instructed to find divergence, dead code, and hidden coupling. Resolve before merge.
7. Semantic diff checks. Run “spec diffs” that compare callable signatures, exceptions, and invariants between HEAD and PR. Block merges that change contracts without version notes.
8. Deletion‑first Fridays. Reserve time for removing RENTAL code, unused helpers, and dark‑launched flags. AI makes code cheap; your edge is making deletion cheap.
9. Interface snapshots. Store machine‑readable “API snapshots” per release (JSON of types, routes, errors). Tests assert snapshot stability across refactors.
10. AI PR reviewer with maintainability persona. A dedicated reviewer prompt only checks: naming consistency, module boundaries, and dependency rules—not correctness. It catches entropy.
11. Provenance headers. At file top:

   ```
   // AI-Origin: ChatGPT-5-Thinking
   // PromptFingerprint: sha256:...
   // Date: 2025-08-19
   // Owner: team-data-platform
   // TTL: 90d
   ```

Reproducibility is a debt reducer.

12. Feature flags as debt valves. All new features ship behind typed flags with default OFF. Flags auto-expire; CI fails if expired flags remain.
13. “No new nouns” rule. AI often invents new domain terms. Require mapping to the existing ubiquitous language; otherwise, the term is rejected or aliased.
14. Refactor bounties. Incentivize deleting 500 LOC by paying the same internal “points” as adding 1,500 LOC. Reward reduction, not accumulation.
15. Scaffold generators with constraints. Provide house templates that embed the maintainability prompts and test harnesses so engineers don’t need to remember them.

## Operating model (Leadership Coaching Required)

Cadence:

* Daily: PR triage enforces test‑first bugfixes and debt budgets.
* Weekly: “Debt Council” reviews expiring TTLs, oversized PRs, and dependency proposals.
* Sprint: Mandatory deletion/refactor goal tied to velocity.

Roles:

* Code Owners guard domain language and public interfaces.
* Prompt Owners maintain `PROMPT.lock` and house templates.
* Test Sheriffs own property and fuzz testing health.

Definition of Done (AI era):

* Contract tests exist and pass.
* Provenance header and `PROMPT.lock` checked in.
* Debt delta within budget; no new nouns without mapping.
* Behavior Equivalence Notes included for refactors.
* Feature flag added with auto‑expire date.

## Metrics and alerts (evidence of telemetry)

Track at repo and service level:

* Debt Growth Rate (DGR) = f(Δcyclomatic complexity, Δpublic API, Δdeps, Δuntested LOC). Show per PR.
* Tests‑to‑Code Ratio for new code (target > 0.3 LOC tests per LOC code; tune per language).
* Flake Rate < 1% (alerts on regression).
* Dependency Freshness SLA (days since last security patch on critical libs).
* TTL Breach Count (should trend to zero).
* Prompt Churn (changes/week to prompts touching core modules).
* Deletion Velocity (LOC removed per sprint; higher is healthier).

Alert on:

* PRs that increase public API surface without API snapshot updates.
* New dependency without dependency test or justification.
* Files created without provenance headers.

## 30‑60‑90 rollout

First 30 days

* Ship house maintainability prompt pack and code scaffolds.
* Add provenance headers + `PROMPT.lock` to CI checks.
* Enforce test‑first on bugfix PRs. Introduce Debt Delta in PR UI.
* Start feature flag auto‑expire.

Days 31‑60

* Add property/fuzz testing to critical modules.
* Quarantine branch policy for new components.
* Stand up Debt Council and weekly TTL review.

Days 61‑90

* Turn on two‑model verification for high‑risk PRs.
* Enable semantic diff checks and API snapshots.
* Introduce refactor bounties; measure Deletion Velocity.

## PR template (paste into your platform)

Title: \[Feature|Fix|Refactor] — Scope

Summary

* What changes and why now.

Behavior Equivalence Notes (for refactor)

* [ ] Public signatures unchanged
* [ ] Error semantics unchanged
* Risks:

Debt Delta

* Complexity: +X / −Y
* New deps: list + justification
* Public API change: yes/no
* Tests added: unit N, property N, fuzz N

Provenance

* AI-Origin:
* PromptFingerprint:
* TTL:

Delete Plan

* What we can remove in 90 days if X/Y/Z happens.

## Example: maintainability prompt bundle (copy/paste)

```
System:
You are a maintainability-first code generator for a long-lived enterprise system.

Policies:
- Prefer stdlib; avoid new deps unless justified with (why, risk, mock plan).
- Keep each function < 40 LOC; complexity < 10. Extract helpers.
- Public API first with docstring and 1-2 usage examples.
- State invariants and error semantics; use assertions.
- Write tests BEFORE code for bugfixes; otherwise, include contract + property tests.
- Insert provenance header and Delete Plan at file top.
- If refactoring: preserve behavior; emit Behavior Equivalence Notes.

Deliverables order:
1) tests/…*_test.<lang>
2) src/….<lang>
3) Behavior Equivalence Notes (if refactor)
4) Delete Plan (5 lines)
5) PROMPT.lock contents
```

## Guidance

AI doesn’t “cause” technical debt; it amplifies whatever governance you already had—good or bad. If your Standard of Performance elevates tests to contracts, prompts to lint rules, and deletion to a first‑class outcome, AI becomes a compounding asset instead of compounding liability.

Adopt these practices, instrument them with telemetry, and coach your team to win boring: small diffs, strong contracts, and fast deletions. That’s how you keep AI’s speed and shrink its debt.
