## Use AI for Expectations Engineering and UAT Before Coding

Start every project by defining what success looks like from the user's perspective, even before writing any code. Use generative AI tools to help draft an Expectations Document that clearly describes the system’s intended behavior and outputs in plain language. This document should capture how the Q\&A AI agent will serve end-users (e.g. what types of questions it will answer, how it should respond) in concrete scenarios. By having AI assist in generating this, you can quickly enumerate user needs and corner cases, which the team can then refine. The goal is to have a shared understanding of *customer expectations* before any development begins.

After outlining expectations, develop a User Acceptance Testing (UAT) Strategy and Plan as the next step. UAT defines exactly how the system will be evaluated to ensure it meets those expectations. No coding should begin until UAT criteria are written and agreed upon. This approach follows the philosophy of Acceptance Test-Driven Development (ATDD) – making acceptance test cases the core of the development cycle. In practice, QA engineers and project leads (not just developers) should collaborate to write detailed UAT test cases covering all important user scenarios. These test cases represent the user’s point of view and validate that the system works as intended. By writing them first, the team stays focused on business requirements and user needs throughout development.

> Example UAT Scenario: *“Given a compliance policy document (in Markdown format) for ISO 27001, when a user asks the system ‘What does clause 9.2 require?’ then the system should respond with a concise, correct explanation of clause 9.2 and a citation pointing to the source section in the document.”* This example defines a clear expected behavior (answer with a verifiable source) that can later be tested. You would create many such UAT test cases for various documents and queries (e.g. ask a sales manual about “refund policy”, or a support handbook about “escalation process”) to capture all typical uses of the Q\&A system.

Importantly, QA and project leads must own the expectations and UAT documents. The UAT plan and cases are the first deliverables to be reviewed and approved by stakeholders (with Shahid’s guidance as needed) before coding. This ensures the team builds the *right* thing. By front-loading expectations engineering and UAT design (even leveraging AI to draft scenarios), you create a strong agreement on what “done” looks like. The team essentially writes the exam that the software must pass. This not only keeps development aligned with user needs but also makes later verification easier – if the system fails a UAT test, it’s not ready. Adopting this expectations-first approach will save time and rework, because you won’t implement features that don’t map to an accepted test. In summary, no code without UAT: get non-coders (QA, product owners) to define the acceptance criteria upfront, and only then let developers start coding against those well-defined targets.

The expectations step in modern AI-native projects is even more important than it was in prior non-LLM systems because systems powered by LLMs are probabilistic, not deterministic. Therefore, QA and UAT must focus not only on binary outcomes but also on reasoning quality, semantic alignment, and goal achievement. The agentic behavior must be validated holistically — from inputs and tools used to final answers and cited evidence.

See YouTube Video [The Contract-First Prompting Blueprint](https://www.youtube.com/watch?v=i4Jfl1IW-_U) for how to convey intent to AI.

<img src="Expect_UAT.drawio.svg" alt="Expectation_UAT Architecture">

### Expectations Engineering as the First Deliverable

We treat expectations documents as structured hypotheses about what the AI system should accomplish from the end-user’s perspective. This is not just a list of requirements — it's a formal articulation of user intent, acceptable behaviors, and measurable outcomes with trustworthiness definitions (meaning what makes the interactions and answers trustworthy in the business context?).

Expectations should be co-developed with domain experts, product owners, and project leads in Expectation Alignment Workshops. These documents will:

* Define key user jobs-to-be-done (JTBD) that the AI assistant must support.
* Identify ambiguities, edge cases, and counterexamples to challenge assumptions.
* Capture the strategic value of each feature, not just its technical manifestation.

> AI can assist in drafting expectations, but human validation is essential. Use LLMs to generate initial drafts of acceptance criteria, then refine with SMEs.

### UAT Before Code: Shift Left Strategy

We implement a “Shift Left” strategy where UATs are written, challenged, and approved before any backend, prompt, or UI code is written. UATs function as both design inputs and alignment mechanisms across QA, engineering, and stakeholder teams.

UATs in this context are not static checklists. They are dynamic, versioned documents that are:

* Written in machine-readable markdown with YAML frontmatter.
* Parsed into structured test cases using AI and NLP tools.
* Continuously updated as understanding of the system evolves.

#### Key Practices

* QA leads actively challenge UAT documents using AI:

  * Generate counterexamples and ambiguous interpretations.
  * Evaluate alignment between business intent and acceptance criteria.
* LLMs may act as co-reviewers of UATs, pointing out vagueness, undefined edge cases, or weak success criteria.

> Example UAT:
> *Given a Markdown document describing ISO 27001, when a user asks about “Clause 9.2,” the system must return a factually accurate, semantically correct summary and cite the exact clause’s location in the document.*

### From Deterministic to Probabilistic QA Mindset

In traditional systems, success is binary — an input yields an expected output. But LLM-based systems require a probabilistic QA strategy focused on evaluating:

* Semantic similarity instead of exact string matches.
* Confidence thresholds and explainability of reasoning chains.
* Intent coverage, not just code coverage.

| Aspect              | Deterministic Systems | Agentic AI Systems                        |
| ------------------- | --------------------- | ----------------------------------------- |
| Validation          | A == B                | Semantic similarity, rubric-based scoring |
| QA Success Criteria | Pass/Fail             | Goal achievement + reasoning correctness  |
| Coverage            | Code paths            | User intent & tool usage coverage         |
| Failure Analysis    | Stack traces          | Reasoning chain, token-level breakdown    |

### Automation from UAT to Test Execution

Once UATs are validated:

1. They are parsed into structured objects using NLP + schema enforcement (e.g. Pydantic).
2. LLMs generate:

   * Positive, negative, and boundary test cases automatically.
   * Synthetic user queries and assertions.
3. Tests are tagged with UAT IDs using tools like `pytest.mark.uat_id("UAT-001")`.
4. Full traceability is maintained from business requirement → UAT → test case → test run.

These test artifacts serve as the foundation for manual validation, automated pipelines, and agent reasoning evaluation.

### Measuring Quality in Probabilistic Agents

Once the system is operational, we go beyond traditional metrics and introduce qualitative and probabilistic KPIs:

* Intent Coverage Matrix – Every expected user intent is tested.
* Monitored Hallucination Rate – Frequency of uncited, incorrect, or fabricated AI outputs.
* Semantic Validation – Outputs are evaluated via sentence similarity scoring.
* LLM-as-a-Judge – GPT-4 or equivalent models score agent performance using a rubric:

  * Goal achievement
  * Reasoning quality
  * Factual accuracy
  * Tool usage
* Purpose Drift Audits – Regular checks to ensure agent behavior still aligns with original mission.

#### Happy Path UAT

* The **ideal scenario** where the user interacts with the chatbot exactly as expected.
* Inputs are correct, clearly structured, and within the chatbot's designed capabilities.
* To confirm that the chatbot works as designed under normal, expected conditions.
* It verifies that **core functions** (like answering FAQs, booking a ticket, or checking account info) are working.

#### Unhappy Path UAT

* Scenarios where the user makes **minor mistakes**, provides unexpected phrasing, or deviates slightly from the ideal usage.
* These are not malicious but do **test the chatbot's flexibility and robustness**.
* To ensure the chatbot can handle variations in input, including slang, typos, or slightly off-topic questions.
* Verifies the **natural language understanding (NLU)** and fallback mechanisms.

#### Miserable Path UAT

* The **worst-case scenarios**, including gibberish, hostile language, or unexpected edge-case inputs.
* It can also include repeated inputs, non-standard symbols, trolling, or intentional misuse.
* To assess **resilience, safety, and ethical boundaries** of the chatbot.
* Helps confirm that the chatbot doesn’t crash, respond inappropriately, or leak sensitive information.

#### 🚀 Why Testing All Paths Is Necessary in UAT

| Path                     | Purpose                                    | Consequence of Skipping                                    |
| ------------------------ | ------------------------------------------ | ---------------------------------------------------------- |
| **Happy Path**     | Confirms functional requirements are met   | Chatbot may pass basic tests but fail in real usage        |
| **Unhappy Path**   | Checks robustness and usability            | Users may get frustrated with unhelpful or rigid responses |
| **Miserable Path** | Ensures ethical, safe, and secure behavior | Could lead to PR issues, user complaints, or legal risks   |

> In UAT for AI chatbots, testing **only the happy path** gives you a **false sense of success**. Real-world users are unpredictable. You must test across **all three paths** to ensure the chatbot is:

* **Functional**
* **User-friendly**
* **Resilient**
* **Safe**

Each path represents a different risk area, and **well-structured test inputs** across all of them ensure the chatbot is ready for deployment.

### Why This Matters

When expectations are documented clearly and early:

* The team knows what to build.
* QA knows how to test.
* AI knows what to optimize for.
* Clients trust the outcomes.

And most importantly, every output is verifiable, traceable, and grounded in an intentional, shared understanding of what "correct" looks like — long before the first line of code is written.

### Use `surveilr` to access Expectations content via SQL

SQL should be the bridge between the expectations layer and the reality of the knowledge layer. How `surveilr` supports expectations engineering:

* Every Expectations Document, UAT Plan, and Test Case authored in Markdown is parsed and ingested by `surveilr` into a relational table of versioned clauses and criteria.
* Test coverage can be queried via SQL to ensure alignment between user expectations and what’s actually been documented or delivered.
* Frontmatter tags like `test_target`, `doc_id`, and `version` allow us to trace UAT coverage across documents and time, helping QA verify if a change in source policy invalidates older tests.

### Use Qualityfolio `surveilr` pattern to express UAT

**Qualityfolio** is a modern test management system built on top of `surveilr` to help QA and business teams manage testing in a way that is:

* Transparent: Everyone can see exactly what’s being tested, why, and how.
* Auditable: Every test case and result is version-controlled and tied back to the original requirement or expectation.
* Structured: All tests live in clean, consistent folders and files — not scattered across spreadsheets or PDFs.
* Integrated: Test plans and results connect with tools your team already uses like GitHub, JIRA, and CI/CD pipelines.

With Qualityfolio:

* Each expectation by an expectations engineer becomes a tracked test
* Each answer is validated and stored as evidence
* Every result is linkable to source documentation

Whether you're preparing for a client demo, a regulatory audit, or just want to ensure quality — **Qualityfolio gives you the structure and visibility to make it happen.**

#### How Qualityfolio Helps with UAT and Trustable AI Interactions

##### Capture Expectations as UAT Artifacts

For Trustable AI Interactions, the most important question is:

> “Can the AI assistant give the right answer to the right question, based on the right documents?”

That starts with **clear expectations** from business users — what we want the system to do. In Qualityfolio, you capture this in a structured way:

* UAT **test cases** (one per business expectation)
* UAT **test plans** (grouped by feature, policy, or document)
* Clear **pass/fail** results that link back to specific documents and AI responses

Example:

> Test Case: “Given a Markdown document about ISO 27001, when a user asks about clause 9.2, the system should return the correct citation and answer with source document link.”

##### Author Tests in Simple Markdown

Unlike older systems, where you need to use complex tools or web forms, **Qualityfolio uses Markdown** — a simple text format:

* Easy to write and read
* Works with Visual Studio Code (a popular editor)
* Can be versioned with Git like software code
* Can be reviewed or co-authored with AI assistants (like ChatGPT)

Even if you're not a programmer, you can work with the content just like editing a Word document — but with the added benefit of version tracking.

##### Organize and Track in a Web UI

Once the tests are written, Qualityfolio presents them in a **clean, web-based UI** inside surveilr:

* View all UAT test cases by feature, document, or status
* Filter to see only failing, passing, or incomplete tests
* Drill down to read the test details, test result, and linked source document

This makes it easy to present UAT progress in stakeholder meetings, or generate reports for compliance teams.

##### Automatic Linkage to External Systems

Every test case in Qualityfolio has a special **Foreign Integration Identifier (FII)** that lets it connect to:

* JIRA tickets for traceability to user stories or defects
* GitHub issues or pull requests
* CI/CD tools to fetch and store test results

This ensures your UAT tests aren’t disconnected from the rest of your work — they become part of the **full lifecycle of feature delivery**.

##### Structured Analysis with SQL

Behind the scenes, everything in Qualityfolio is stored in **SQLite databases** via surveilr:

* All tests, plans, and results are queryable with SQL
* Business and QA teams can generate **custom reports**:

  * “Show all UAT tests related to onboarding that failed last week”
  * “Which AI-generated answers failed UAT and don’t have a fix yet?”
  * “What percentage of expectations have been validated?”

No special technical skills required — analysts and leads can use preset queries or dashboards.

