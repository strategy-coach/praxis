# How Qualityfolio and surveilr Work Together to Implement the Doctrine

This document describes how Qualityfolio (test management system) and surveilr (provenance, telemetry, and compliance system) integrate to operationalize the practices outlined in the “Why Technical Debt Grows Faster With AI-Generated Code” doctrine. Together, they transform abstract standards into daily guardrails that ensure AI collaboration increases velocity without compounding debt.

Qualityfolio ensures tests are the currency of trust. surveilr ensures provenance is the ledger of truth. Together, they operationalize the doctrine so that AI accelerates development velocity without silently compounding technical debt.

## surveilr’s Role: Provenance and Telemetry for AI-Generated Code

surveilr is the information backbone. It captures, stores, and makes queryable the metadata around all code artifacts, prompts, and test evidence.

### Key Features Supporting the Doctrine

* Provenance headers: surveilr enforces headers in every file with `AI-Origin`, `PromptFingerprint`, `Date`, `Owner`, and `TTL`.
* PROMPT.lock as first-class citizen: Prompts and versions are checked in, tracked, and linked to repos through surveilr’s `uniform_resource` schema.
* Debt telemetry: surveilr measures complexity, dependency deltas, and test-to-code ratios. These metrics flow into dashboards and alerts.
* Dependency quarantine: Any PR that introduces new libraries is tagged in surveilr, forcing review with justification.
* Deletion velocity: surveilr tracks removed LOC, expired flags, and TTL breaches. Teams get credit for deletions, not just additions.
* Audit trail: Every AI interaction (prompt + output) is logged and queryable, enabling compliance and reproducibility.

<img src="surveilr-ai-code-management-diagram.svg" alt="surveilr- Architecture">

## Qualityfolio’s Role: Test Management as Executable Contracts

Qualityfolio is the test truth system. It manages and verifies that every piece of code—AI-generated or human-authored—carries test evidence.

### Key Features Supporting the Doctrine

* Markdown-based test storage: Test cases, suites, and scenarios live as Markdown files versioned in Git, tightly coupled to the code.
* Foreign Integration Identifiers (FIIs): Link code artifacts in surveilr to test cases in Qualityfolio, ensuring traceability between AI prompts and unit/property/fuzz tests.
* Test debt metrics: For each commit, Qualityfolio computes tests-to-code ratios and surfaces gaps.
* Property/fuzz test templates: Engineers can invoke AI with Qualityfolio templates that force the generation of property-based and fuzz tests, not just happy-path unit tests.
* Flake tracking: Qualityfolio records flaky test runs and feeds back into surveilr telemetry, closing the loop on reliability.
* Evidence repository: Executed tests and results are stored, allowing auditors or future maintainers to prove behavior equivalence after refactors.

<img src="qualityfolio-architecture-diagram.svg" alt="qualityfolio- Architecture">

## Integration Model: surveilr + Qualityfolio

Think of surveilr as the ledger and Qualityfolio as the scoreboard.

1. Code provenance + prompts (surveilr)

   * Developers (or AI agents) commit code with provenance headers and prompt metadata.
   * surveilr logs every artifact into its `uniform_resource` schema.
2. Tests as contracts (Qualityfolio)

   * Companion tests are authored/generated, stored in Markdown, and ingested by Qualityfolio.
   * Qualityfolio ensures there’s at least one failing test before bugfix PRs, and contract tests for new modules.
3. Cross-linking

   * surveilr’s provenance entries reference Qualityfolio test IDs.
   * Qualityfolio dashboards embed surveilr metrics (debt deltas, TTLs, dependency sprawl).
4. CI/CD enforcement

   * PR merges blocked if surveilr detects missing provenance or PROMPT.lock.
   * PR merges blocked if Qualityfolio detects inadequate test coverage or missing property/fuzz tests.
5. Debt council reviews

   * Weekly, surveilr surfaces TTL breaches, dependency justifications, and deletion goals.
   * Qualityfolio provides a report on flake rates, failing contracts, and missing evidence.
6. Telemetry feedback loop

   * surveilr’s telemetry feeds into Qualityfolio’s test analytics, creating a single picture of health.
   * Teams can see not just “what passed” but “what debt grew” in the same dashboards.

## Innovative Practices Enabled by the Partnership

* Prompt-to-Test Linking
  Every AI prompt that generated code must generate a test; surveilr ties the prompt to the test ID in Qualityfolio. If the test is later deleted, the provenance record surfaces a risk flag.
* TTL-based Test Expiry
  surveilr TTLs on AI code automatically generate “test renewal” tickets in Qualityfolio to ensure old tests aren’t masking drifted behavior.
* Debt-Aware Velocity Metrics
  Instead of celebrating LOC added, dashboards in Qualityfolio display *deletion velocity* and *test-to-code ratio trends*—all fed from surveilr’s measurements.
* Automated Behavior Equivalence Checks
  When code is refactored, surveilr triggers “spec diff” checks; Qualityfolio validates that golden tests and property tests still pass. Together, they ensure semantic drift is detected.
* Compliance and Audit Readiness
  In regulated domains (healthcare, finance), surveilr’s provenance + Qualityfolio’s test evidence combine into an auditable trail showing exactly which AI models, prompts, and human interventions produced the code and which tests guaranteed its safety.

## Example Workflow

1. Engineer asks AI to scaffold a new API endpoint.
2. surveilr logs the prompt, model version, provenance header, and TTL.
3. AI also produces contract tests, which are ingested into Qualityfolio.
4. CI runs Qualityfolio tests; surveilr calculates Debt Delta and confirms dependency justifications.
5. If all checks pass, code merges.
6. 90 days later, surveilr TTL triggers a review issue: “Revalidate this endpoint or delete.”
7. Qualityfolio auto-schedules re-run of property/fuzz tests; if they fail, the TTL review escalates.

Result: AI code is never “free”—it’s always paired with provenance and executable contracts.

## Qualityfolio as the Tip of the Spear for Managing AI Technical Debt

AI has made writing code easier than ever—but the debt hidden in that code grows faster than most organizations realize. Qualityfolio provides the *measurement system, governance model, and visibility layer* that turns AI-generated code from a liability into a durable asset.

As a tool, Qualityfolio can offer organizations a “free AI Technical Debt Baseline Assessment”—instantly valuable, easy to deliver, and opening the door to broader adoption of the full product.

Traditional test management tools focus on human-written code. They measure coverage, defects, and test execution. But with AI code generation, new forms of debt appear:

* Prompt Drift: Small changes in wording generate inconsistent code styles.
* Provenance Gaps: Nobody knows which model/version produced the current function.
* Debt Multiplication: AI scaffolds five times faster than humans, but testing rarely keeps pace.
* Silent Divergence: Refactored code changes semantics subtly; unit tests can’t catch all differences.

Qualityfolio is designed to track and contain these new liabilities, while still functioning as a full-spectrum test management solution.

### AI Tech Debt Assessment with Qualityfolio

Project leaders can use Opsfolio to scan repos and instantly tell you how much AI-generated code you have, how much of it is untested, and where your risk hotspots are. This assessment is done via Qualityfolio’s telemetry ingestion (integrated with surveilr) and produces a web-ui showing.

- Untested AI-generated modules.
- Functions without provenance headers.
- AI scaffolds older than their TTL.
- Flaky or failing tests tied to AI code.
- A Debt Velocity Score (how fast debt is accumulating vs. deleted).

### How Qualityfolio Implements AI Technical Debt Management

#### 1. Provenance-aware Testing

* Every test case is linked to the originating code and the AI prompt that generated it.
* If a file is missing provenance metadata, Qualityfolio flags it as orphaned code.

#### 2. Debt Delta Tracking

* On every PR, Qualityfolio computes a Debt Delta: changes in test-to-code ratio, dependency sprawl, and complexity.
* Managers can set thresholds (e.g., “no PR merges if Debt Delta > 10%”).

#### 3. TTL-driven Test Expiry

* AI code often drifts after a few months. Qualityfolio enforces time-to-live (TTL) for generated modules.
* When TTL expires, the platform triggers a test renewal cycle: old tests must be re-run, re-generated, or expanded.

#### 4. Fuzz + Property Test Integration

* Qualityfolio provides AI-driven test templates that force generation of fuzz tests and property-based tests, not just unit tests.
* These templates embed directly in the repo and are enforced at merge time.

#### 5. Golden Contract Snapshots

* Every time an API or service layer changes, Qualityfolio captures a contract snapshot.
* Future AI-generated refactors must be validated against these snapshots to prevent semantic drift.

#### 6. Flake Rate Monitoring

* Qualityfolio measures flake rates (tests that pass/fail intermittently) and ties them back to the originating AI prompt.
* Helps identify AI-generated code that is unstable by nature.

#### 7. Deletion Velocity as a KPI

* Technical debt is not just about growth—it’s about how quickly you can delete unneeded code.
* Qualityfolio tracks LOC deleted vs. LOC added, surfacing teams that are managing debt proactively.

### Beyond AI: General Tech Debt Tracking

Even for non-AI-heavy codebases, Qualityfolio provides debt observability:

* Debt Heatmaps: highlight subsystems with high cyclomatic complexity and low test coverage.
* Cross-team comparison: identify which squads are adding code without tests.
* Dependency Drift Reports: track outdated libraries or high-risk imports.
* Evidence repository: keep an auditable trail of every test result linked to specific features.

### Innovative Qualityfolio Tech-debt Observaiblity Features

1. Debt NFTs (Non-Fungible Tests)

   * Every test case is minted as a unique, traceable artifact in surveilr.
   * Helps prove compliance: “This test has verified this feature since YYYY-MM-DD.”

2. Debt-as-a-Service Scoring API

   * Expose a Qualityfolio API that returns a Tech Debt Score for any repo.
   * Customers can integrate this into CI dashboards or procurement pipelines.

3. Debt Bounties

   * Teams can open “debt bounty tickets” inside Qualityfolio. Engineers earn points for reducing LOC, fixing flaky tests, or deleting dead code.

4. AI Reviewer Persona

   * A built-in AI reviewer trained to look only for debt indicators: complexity growth, missing tests, expired TTLs.
   * Works as a “second pair of eyes” alongside human code review.

5. Feature Flag Expiry Tracker

   * Qualityfolio links test cases to feature flags. If a flag is expired but tests still reference it, the system forces cleanup.

6. Behavior Equivalence Testing at Scale

   * Automatically generates differential tests: run old vs. new code paths side by side with the same inputs, flagging any semantic divergence.

7. Debt Reduction Leaderboards

   * Public or internal dashboards that celebrate engineers/teams who delete the most debt, incentivizing healthy behavior.

Qualityfolio is not just another test management tool.
It is the AI era’s debt containment system—the only platform designed to:

* Track: what code came from AI, and under what assumptions.
* Test: every behavior change, edge case, and property.
* Delete: enforce TTLs and incentivize removal of dead code.
* Prove: provide compliance-grade evidence that AI code is safe, stable, and maintainable.

## Next Steps for Adoption

1. Baseline Assessment

   * Scan repos and produce AI Tech Debt Heatmap.

2. Pilot Implementation

   * Deploy Qualityfolio + surveilr in one squad’s workflow.
   * Integrate Debt Delta checks into CI/CD.

3. Enterprise Rollout

   * Standardize on Qualityfolio for all test evidence.
   * Mandate provenance + TTL enforcement org-wide.

## Get Started

In the AI era, code is free but maintainability is priceless.
Qualityfolio, in partnership with surveilr, ensures that the cost of AI velocity doesn’t balloon into unpayable debt.

It is not just a tool for QA—it’s the strategic control point for software debt governance.