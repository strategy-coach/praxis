---
title: "Case Studies"
description: "Applying the AI-native documentation doctrine to real-world projects (`surveilr`, Opsfolio, SQLite, PostgreSQL, Redis)."
intended-audience: ["Engineers", "Product Managers", "Technical Writers", "LLM Prompt Engineers"]
version: "1.0"
ai-context:
  prompt-grounding: |-
    `surveilr` (compliance and security):
    - Modules per control, task-based guides (e.g., “Generate HIPAA audit report”). 
    - Strong warnings where actions affect evidence or alerts.
    - Version and audience tags to tailor operator vs. auditor guidance.

    Opsfolio (risk and operations):
    - Role-tagged modules for admins vs. risk officers.
    - Entities as modules (assets, risks, controls) and error references.
    - Troubleshooting library seeded from support tickets.

    SQLite:
    - Q&A for limits, PRAGMA behaviors, backup methods.
    - Separate modules for data types, functions, transaction semantics.

    PostgreSQL:
    - One module per GUC, command, and common error pattern.
    - Versioned facts; retrieval filtered by user’s version when known.

    Redis:
    - One module per command and per memory/eviction policy.
    - Pattern guides (rate limiting, caching) with examples and caveats.
---
# Case Studies

To make the doctrine more concrete, let’s explore how it would apply in a few real-world contexts. We’ll look at two of Netspective’s products (`surveilr` and Opsfolio) and three popular open-source projects (SQLite, PostgreSQL, Redis). In each case, we’ll consider the current state (old world) and how the new AI-native approach could transform their documentation and user experience.

`surveilr`: `surveilr` is an automated compliance and security management platform. Its documentation likely covers a lot of policy configurations, compliance controls, and evidence gathering procedures. In the old world, an engineer using `surveilr` might have to read through a user guide PDF or navigate a web portal to figure out how to, say, “generate a HIPAA compliance report” or “add a new compliance policy.” Applying AI-native principles, we’d restructure `surveilr`’s docs into a modular knowledge base:

* Each compliance standard (HIPAA, GDPR, etc.) gets its own section with fact modules on specific controls, requirements, and how `surveilr` addresses them.
* Common tasks (like *“Gather evidence for control XYZ”* or *“Automate an audit report”*) are documented step-by-step in self-contained articles. Each article’s title is phrased as a user query (“How to generate an audit report for HIPAA”, “How to onboard a new system for monitoring”).
* `surveilr`’s system might have many configuration options; each option is documented with its purpose and impact. For example, if there’s a setting “Enable continuous monitoring”, its doc entry clearly states what it does and when to use it.
* Warnings and best practices are embedded so that the AI will relay them. If a certain action might disable alerts, the documentation module for that action would include a note: “*Important: Disabling this will stop all email alerts.*” An AI reading that will include it in the answer.
* By ingesting this documentation into an AI assistant (perhaps within the `surveilr` interface or as a chatbot on the support site), a compliance officer could ask in plain English: “How do I prove that our database backups are encrypted for HIPAA?” The AI would pull up the relevant `surveilr` doc on evidence collection for encryption controls and guide the user: “`surveilr` automatically logs backup encryption status. To generate a report, go to… etc.” In the background, it used the doc that we wrote for that very question.
* `surveilr` often integrates with regulations; keeping information updated is key (regulations change). An AI-first doc approach means if a regulation update happens, you update that fact in one place in the docs (“Policy XYZ new requirement as of 2025…”) and the AI will then incorporate that into all relevant answers. This is more efficient than retraining an AI or relying on memory – we’re leveraging the dynamic doc.

**Opsfolio:** Opsfolio captures risks, IT assets, and documents ops work (risk & compliance management). Its documentation spans multiple user roles (sysadmins, project managers, risk officers). In an AI-first model:

* We tag documentation modules by role. For example, an “Asset Inventory Guide” might be tagged `audience: System Administrators` and another “Risk Dashboard Usage” tagged `audience: Risk Officers`. The AI assistant, if it knows the user’s role (perhaps through login context or by asking), can tailor answers accordingly. If a sysadmin asks “How do I add a new server to Opsfolio?”, the AI will use the technical how-to. If a risk officer asks something similar but in less technical terms (“How do we record new assets?”), the AI might provide a higher-level answer drawing from more summary docs.
* Opsfolio likely has a lot of interconnected pieces (assets, risks, controls). Documentation would have a module for each major entity and its fields (“Asset: definition and fields”, “Risk: how it’s calculated”, etc.). If a user asks “What is the ‘Exposure Score’ in Opsfolio?”, the AI would answer with the definition straight from the docs (e.g., “Exposure Score is a calculated risk metric that...” – presumably referencing the doc).
* For troubleshooting (Opsfolio integration issues, etc.), every error message or common issue (like “data import fails”) is documented. The AI can serve as a first-line support, walking the user through solutions from the docs (“Have you provided the API key? According to docs, error 401 means invalid credentials.”).
* Both `surveilr` and Opsfolio have a compliance angle, so accuracy is crucial. The AI-first approach helps ensure that if, for example, a regulation threshold changes, updating the docs will immediately correct the AI’s guidance. This reduces the risk of an AI giving outdated compliance advice (a big no-no).
* By implementing AnythingLLM internally, Netspective’s engineers could have a private chatbot for Opsfolio/`surveilr` docs that speeds up onboarding new team members or answering client questions by pulling the answers from documentation instantly.

**SQLite (open-source database):** SQLite’s docs are known to be very detailed, but largely human-oriented (a sprawling FAQ, lengthy spec documents). With AI-native documentation:

* We would create modules for each SQL feature and each C API function (since SQLite is embedded). For instance, a question “What is the maximum number of columns in a SQLite table?” would be answered by an explicit doc entry (“SQLite supports up to 2000 columns per table by default.”). Currently, a user might have to search the web or SQLite limits doc for that. An AI can give it immediately if that fact is isolated in docs.
* The **“How do I...”** style is very useful for SQLite. e.g., “How do I create an index in SQLite?” – The docs might already cover it in narrative, but we’d add a clear Q\&A or recipe. The assistant would then output: “To create an index in SQLite, use the CREATE INDEX command. For example: `CREATE INDEX idx_name ON table(column);`”.
* SQLite has some idiosyncrasies (like how typing works, or that it’s file-based). Those key concepts become modules (“How data types work in SQLite”, “Transaction behavior in SQLite”). If a user asks conceptual questions (“Does SQLite support foreign keys?”), the AI answers from docs (“Yes, but foreign keys are disabled by default before version 3.6.19; you must enable PRAGMA foreign\_keys…” citing the appropriate part of the doc).
* One challenge is version differences: SQLite is backwards compatible mostly, but new features come. Our doc modules would note “Added in 3.35” for example, and the AI could mention that if asked.
* Implementing this could be as simple as ingesting the SQLite docs and FAQ into a vector DB and using an LLM. In fact, the SQLite community or others might have done similar via Stack Exchange data, but an official doc-driven one would likely be more accurate.

**PostgreSQL (open-source relational DB):** PostgreSQL’s documentation is extensive and structured by version (each version has its set). In an AI-first approach:

* We would use the existing structure (which is quite modular in content but delivered as big HTML pages) to create finer modules. For example, each configuration parameter (there are hundreds) becomes a doc entry (this is somewhat the case in the official docs which have a reference page listing them – those could be broken out or at least indexed by name).
* A developer might ask, “What does the work\_mem setting do in Postgres 13?” The AI would retrieve the module for `work_mem` (with a note “in v13, default 4MB, used for sort operations etc.”) and answer accordingly, citing that. If the question was just “what does work\_mem do?” and context not given, the AI might either clarify or default to the latest version, but ideally it would mention differences if any.
* Querying and error help: “Why am I getting `ERROR: could not serialize access due to read/write dependencies`?” – The doc chunk about transaction isolation and that specific error (if documented in the MVCC or SSI section) would allow the AI to explain, “This error indicates a serialization failure in repeatable read transactions, meaning two concurrent transactions conflicted. The solution per docs is to retry the transaction.”
* PostgreSQL also has a lot of how-tos and tutorials scattered in wiki or blogs. An official AI assistant could integrate official docs with curated knowledge from wiki perhaps, but sticking to official to start is safer. The modular approach ensures it doesn’t confuse, say, configuration of streaming replication between major versions.
* The frontmatter for each page could include version: if an AI knows user’s PG version from conversation, it could filter answers (if user says “I’m on Postgres 12”, answers about features from 14 wouldn’t be offered unless relevant).

**Redis (open-source key-value store):** Redis documentation includes command references, topics on persistence, replication, etc. In AI-first usage:

* Each Redis command (like `GET`, `SET`, ` KEYS`, etc.) is already a standalone reference page. That’s good. We’d ensure those are ingested. So if a user asks, “What does the KEYS command do in Redis?” the assistant directly pulls that reference (maybe warning: “Note: KEYS is O(n) and not recommended on large databases” which is a famous caveat, present in docs).
* Higher-level questions like “How do I expire keys in Redis?” – the assistant would combine knowledge of the `EXPIRE` command and how expiration works (could be separate doc modules: one for the command, one for the concept of expiration).
* Troubleshooting: “Why is my Redis memory full?” – If the docs have a section on maxmemory and eviction policies (they do), the assistant can explain eviction policy, how to configure it, etc., based on that content.
* We would include config parameters (like `maxmemory-policy`, each as a module with explanations). That way, “What are the eviction policies in Redis?” yields an answer listing them, drawn straight from docs that list those modes.
* An AI advantage: linking related commands or concepts on the fly. If a user asks about replication but actually their issue is persistence, the assistant might pick up on relevant sections (thanks to embedding similarities or follow-ups).
* Redis’s docs also have some warnings about not doing certain things (like using `FLUSHALL` in production). Those would be clearly surfaced. If user asks “How do I clear all data in Redis?”, the doc’s warning on `FLUSHALL` should be included by the assistant: “You can use FLUSHALL, but be careful – it deletes everything and is irreversible.”

Through these case studies, a common theme emerges: **the AI-first approach doesn’t necessarily require creating completely new documentation** – it often involves **reorganizing, tagging, and sometimes augmenting existing docs** to fill gaps (especially around common tasks and troubleshooting). The payoff is a drastically improved support experience: instead of combing through manuals or Googling, users (or engineers internally) get immediate, context-specific answers.

For the internal Netspective products (`surveilr` and Opsfolio), adopting this doctrine could be a competitive advantage – customers get an “expert assistant” alongside the product. For open-source communities like PostgreSQL or Redis, it can reduce repetitive Q\&A on forums as the official docs themselves become more accessible through an AI.

One more mini case: **Myths vs Truths documentation page ([like this one](old-vs-new-world.md)).** Imagine if an AI had ingested this doctrine paper itself. If someone asked, “Is AI going to replace documentation writers?”, the AI would answer with the truth we stated: that AI won’t replace the need for quality docs or writers, because it only can work from existing content, and that human expertise is needed to provide accurate information. In other words, our own documentation here is ready to be consumed by an AI to answer such meta-questions about documentation strategy!

Having explored these cases, you can see how the doctrine can be applied broadly: from enterprise tools to developer tools. The specifics differ, but the underlying patterns – modular docs, embedded context, factual focus – remain constant.
