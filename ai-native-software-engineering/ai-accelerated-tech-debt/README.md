# Doctrine: Wrangle AI-Accelerated Tech Debt using Testing First Principles

> by Shahid N. Shah

AI makes software faster to write, not faster to understand. Debt grows when the rate of code creation exceeds the rate of shared comprehension, tests, and runtime verification. AI is a force multiplier on both sides of that equation. Treat AI like an ultra‑productive junior team that never sleeps: it will do exactly what you asked—plus a few things you didn’t notice you asked.

> Read more about [Why AI accelerates technical debt](why-tech-debt-grows-faster-with-ai.md).

This doctrine explains why debt accelerates in the AI era, why unit tests and maintainability prompts become non‑negotiable, and what contrarian practices cut debt growth while preserving AI speed.

> Learn about how [Qualityfolio can be the tech debt observability platform](qualityfolio-as-tech-debt-manager.md).

In an AI code creation world, speed increases while shared understanding lags. Long‑term maintainability depends on three anchors:

1. unit tests from first principles,
2. integration tests from first principles,
3. durable provenance that distinguishes AI vs human authorship with clear ownership.

Clean code is the enabler; tests are the executable specification; provenance is the memory. Without all three, human–AI collaboration decays into untraceable entropy.

## First principles of unit testing

Definition

* A unit is the smallest testable behavior that can be reasoned about in isolation.
* The goal is fast, deterministic confirmation of intent, not coverage for its own sake.

Axioms

* Specify, don’t simulate: write tests that assert observable behavior and invariants, not implementation details.
* Determinism: control time, randomness, and environment; tests must produce the same result every run.
* Small state surface: prefer pure functions; keep side effects behind thin seams.
* One reason to fail: each test clarifies exactly which contract was violated.
* Fast feedback: sub‑second tests make refactoring safe and frequent.

Core practices

* Arrange–Act–Assert with minimal fixtures.
* Property‑based tests for general truths (idempotence, order‑invariance, monotonicity, conservation).
* Table‑driven cases for edge coverage.
* Mutation testing budget to prove tests are sensitive to real faults.
* Golden tests for stable I/O formats; store small, human‑reviewable fixtures.

Anti‑patterns to avoid

* Testing private internals instead of public contracts.
* Mocking everything by default; prefer real collaborators unless latency or nondeterminism requires a fake.
* Snapshots used as a substitute for understanding; keep snapshots narrow and intentional.

Unit test checklist

* Public API and error semantics covered
* Happy path plus at least two edge cases
* Property test present where a law exists
* No hidden reliance on global time, RNG, or external state
* Mutation score above your threshold for the module

## First principles of integration testing

Definition

* Integration tests validate that components compose correctly across process, network, storage, and schema boundaries.

Axioms

* Reality over mocks: use real infrastructure in ephemeral form (containers, local cloud emulators) when feasible.
* Stable contracts: consumer‑driven contracts or API snapshots prevent silent drift.
* Isolated data: each test controls its own dataset; truncate or namespace per run.
* Observable side effects: assert on durable outcomes (rows written, messages published), not log strings.
* Eventual consistency awareness: assert with time bounds and retry policies where appropriate.

Core practices

* Test containers or ephemeral environments in CI.
* Contract tests between services; versioned and published with the consumer.
* Migrations verified against realistic data subsets.
* Fault injection for resilience properties (timeouts, partial failures).
* Performance smoke checks on critical paths to catch pathological slowdowns early.

Integration test checklist

* Contracts verified for both success and error cases
* External dependencies version‑pinned in test env
* Seed and teardown strategies defined
* Latency and throughput thresholds asserted for hot paths
* Idempotency, retries, and duplicate message handling tested

Testing pyramid guidance in AI collaboration

* Keep end‑to‑end tests few and meaningful.
* Grow unit and contract tests aggressively; they are cheaper to evolve when AI refactors internals.

## Clean code as a collaboration protocol

Principles

* Small units and simple control flow reduce the prompt size and mental load needed for AI and humans to reason.
* Narrow, explicit interfaces minimize coupling that AI might accidentally introduce through context inference.
* Consistent idioms and naming form a “lingua franca” that stabilizes future AI generations.

Team guardrails

* Function length caps and cyclomatic complexity budgets enforced in CI.
* Standard error handling patterns and result types across the codebase.
* Feature flags with auto‑expiry to keep scaffolds and experiments from fossilizing.

## Provenance: durable, maintainable authorship and ownership

Why it matters

* Reproducibility: you must be able to regenerate or explain code origin.
* Accountability: someone must own the semantic intent after merge.
* Review quality: different scrutiny for AI‑generated vs human‑crafted code paths.

Provenance model (file‑level header)

* AI‑Origin: model name and version (e.g., GPT‑5‑Thinking)
* Prompt‑Fingerprint: stable hash of the instruction set and tool list
* Generated‑At: timestamp
* Owner‑Of‑Record: team or person responsible for semantics
* Ownership‑TTL: revalidation window (e.g., 90 days)
* Dependencies‑Added: list and justification
* Behavior‑Contract: brief statement of invariants and error semantics

Example headers

TypeScript

```ts
// Provenance:
// AI-Origin: GPT-5-Thinking
// Prompt-Fingerprint: sha256:1cf2…a9e
// Generated-At: 2025-08-19T04:00:00Z
// Owner-Of-Record: team-platform-data
// Ownership-TTL: 90d
// Dependencies-Added: none
// Behavior-Contract: function is idempotent; throws DomainError on invalid state
```

Python

```python
# Provenance:
# AI-Origin: GPT-5-Thinking
# Prompt-Fingerprint: sha256:1cf2…a9e
# Generated-At: 2025-08-19T04:00:00Z
# Owner-Of-Record: team-platform-data
# Ownership-TTL: 90d
# Dependencies-Added: requests>=2.32 (justify: HTTP/2, retries)
# Behavior-Contract: pure transform; no I/O; raises ValueError on bad schema
```

Git commit trailers

* Origin: AI|Human|Mixed
* AI-Model: gpt-5-thinking\@2025-08-19
* Prompt-Fingerprint: sha256:…
* Co-Authored-By: optional human pair reviewer
* Owner-Of-Record: team or username
* Change-Type: feat|fix|refactor|test
* Contract-Impact: none|minor|breaking

Example commit footer

```
Origin: AI
AI-Model: gpt-5-thinking@2025-08-19
Prompt-Fingerprint: sha256:1cf2…a9e
Owner-Of-Record: team-platform-data
Contract-Impact: none
```

Git `blame` style ownership tagging

* CODEOWNERS defines review gates by path.
* Owner‑of‑Record in headers ties ongoing stewardship to a team, not just a historical committer.
* Ownership‑TTL creates renewal tasks; if no owner revalidates, CI blocks further changes in that path.

Enforcement

* Pre‑commit hook injects headers if missing.
* CI fails if provenance is absent, malformed, or TTL expired.
* PR template requires Behavior Equivalence Notes for refactors and dependency justifications for new imports.

## Human–AI maintenance collaboration rules

* No prompt, no merge: any AI‑generated code must have a recorded prompt fingerprint.
* Test‑first bug fixes: failing test added before the fix.
* Two‑model review for risky changes: generate with Model A, critique with Model B focused on maintainability and hidden coupling.
* Quarantine for new subsystems: land behind flags, with minimal dependencies, burn‑in for two weeks.
* Delete plans: every new module includes a short plan outlining what can be removed when requirements change.

## PR template

Title

* \[feature|fix|refactor|test] concise scope

Summary

* What changed and why now

Behavior Equivalence Notes (for refactors)

* public signatures unchanged yes/no
* error semantics unchanged yes/no
* risk areas and rationale

Debt delta

* complexity +/−
* new dependencies and mock strategy
* public API surface +/−
* tests added: unit N, property N, contract N, fuzz N
* mutation score before/after

Provenance

* AI‑Origin, Prompt‑Fingerprint
* Owner‑Of‑Record, Ownership‑TTL

Delete plan

* what becomes removable in 90 days and under what conditions

## Metrics and thresholds

* Tests‑to‑Code Ratio for new code per PR; track trend, not just absolute
* Mutation score per module; block merges under threshold on critical paths
* Flake rate under 1%; quarantine flaky tests and escalate ownership
* Contract drift index: count of API snapshot changes without version notes
* Dependency freshness SLA: days since last patch for critical libraries
* Ownership TTL breaches: should trend to zero
* Deletion velocity: LOC removed per sprint; higher is healthier

## 30‑60‑90 day rollout

Days 0–30

* Adopt provenance headers and commit trailers; add CI checks.
* Ship unit test templates and property‑based scaffolds for each language.
* Establish CODEOWNERS and Owner‑Of‑Record policy.
* Start test‑first requirement for bug fixes.

Days 31–60

* Introduce contract testing between services and API snapshotting.
* Add mutation testing to critical modules.
* Turn on Ownership‑TTL reminders and block merges on expired TTL.
* Begin dependency gate reviews with justifications and mocking plans.

Days 61–90

* Enable two‑model review for high‑risk PRs.
* Add ephemeral integration envs in CI with seeded data.
* Track debt delta and deletion velocity in dashboards.
* Run a deletion‑first day each sprint to remove unused code and flags.

## Minimal templates you can paste today

Unit test prompt (for AI assistant)

```
Write unit tests first for <function/module>. Constraints:
- Cover happy path and two edge cases.
- Include at least one property-based test stating the core law.
- Control time/randomness; tests deterministic.
- Avoid mocking unless required; if mocking, show the seam.
Output tests before implementation. Language: <lang>.
```

Behavior‑preserving refactor prompt

```
Refactor for readability and smaller functions; preserve behavior.
Do not change public signatures or error types.
Emit "Behavior Equivalence Notes" listing any semantics that could change.
Provide a delete plan and update provenance header.
```

Integration test template (contract)

```markdown
Contract name: <ServiceA -> ServiceB>
Success cases:
- input, output, status, latency budget
Error cases:
- invalid input -> error type, status
- dependency timeout -> retry/backoff semantics
Snapshot ID: <hash>
Owner-Of-Record: <team>
```

Pre‑commit header snippet (shell)

```sh
#!/usr/bin/env sh
for f in $(git diff --cached --name-only --diff-filter=ACMR | grep -E '\.(ts|js|py|go)$'); do
  if ! grep -q "Provenance:" "$f"; then
    now=$(date -u +"%Y-%m-%dT%H:%M:%SZ")
    sed -i '' "1i\\
$(printf '%s' "// Provenance:\n// AI-Origin: unknown\n// Prompt-Fingerprint: unknown\n// Generated-At: $now\n// Owner-Of-Record: TBD\n// Ownership-TTL: 90d\n// Dependencies-Added: none\n// Behavior-Contract: TODO")\\
" "$f"
    git add "$f"
  fi
done
```

## Abstract

First‑principles unit tests make intent explicit. First‑principles integration tests make composition safe. Provenance and git‑style ownership make accountability durable. Together, they turn AI speed into sustainable progress and ensure that future humans and future models can collaborate on the same codebase without losing the plot.
