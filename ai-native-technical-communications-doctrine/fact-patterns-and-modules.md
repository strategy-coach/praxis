---
title: "Fact Patterns and Modules"
description: "Breaking down products into retrievable fact units for accurate AI retrieval."
intended-audience: ["Engineers", "Product Managers", "Technical Writers", "LLM Prompt Engineers"]
version: "1.0"
ai-context:
  prompt-grounding: |-
    Identify atomic units:
    - Definitions: what a term means.
    - Procedures: how to perform a task.
    - Configuration: option name, purpose, default, range.
    - Limits: maximums, timeouts, quotas, supported versions.
    - Errors: message/code, cause, resolution.
    - Best practices: recommended patterns and anti-patterns.

    Implementation examples:
    - PostgreSQL: one module per GUC (e.g., work_mem), per SQL command, per error pattern.
    - Kubernetes: separate Service vs. Ingress vs. NetworkPolicy with clear “when to use.”
    - Redis: one module per command and per policy (e.g., eviction), plus usage patterns.
    - Linux: destructive commands documented with explicit warnings and rollback notes.

    Guidelines:
    - One question per module; keep it self-contained.
    - Cross-link related modules; minimize contradictory duplicates.
    - Tag with version and audience to avoid cross-edition confusion.
---
# Fact Patterns and Modules

One of the most practical steps in making documentation AI-native is to **break your knowledge into retrievable units**, which we can call modules or fact modules. This process is both an art and a science. It’s about identifying the *fact patterns* in your product – the recurring kinds of information or questions – and ensuring each is documented in a tidy, accessible way.

Think of your product and list out the types of “facts” someone might ask about:

* Definitions of key concepts (“What is X?”).
* How-to procedures (“How do I do X?”).
* Configuration options and their meanings (“What does setting X do?”).
* Limits and constraints (“What’s the max of X? Can I do X concurrently?”).
* Error messages and troubleshooting (“What does error code 123 mean? How to fix Y error?”).
* Best practices or recommendations (“Recommended approach for X?”).

In a human-first doc, these facts might be sprinkled across pages or discussed in paragraph form. In AI-first doc, each atomic fact or small group of related facts should ideally live in its own module or clearly delineated section**.** This ensures that when an AI searches for that fact, it finds a dedicated chunk rather than a passing mention.

For example, let’s say we have an open-source database (like PostgreSQL). Fact patterns include:

* **Configuration parameters**: Each parameter (e.g., `shared_buffers`, `work_mem`) is a fact unit. Documentation should list what it does, its default, min/max, effect, etc., ideally under a heading that is exactly the parameter name. That way, if a user asks “What does `work_mem` do in PostgreSQL?”, the AI can pull the chunk under the `work_mem` heading, which has the explanation and default value.
* **SQL commands or functions**: Each command (`SELECT`, `CREATE TABLE`, etc.) or built-in function (`NOW()`, `SUM()`) is a fact unit. Traditional docs might have one giant page for all SQL functions. AI-first would argue for each function (or at least each category of function) to be separate or at least distinctly indexed. In fact, some projects have done this – e.g., the Redis documentation gives each command its own page, which is very AI-friendly.
* **Error codes**: If your system has error codes or messages (like “ORA-XXXXX” in Oracle, or specific log errors), treat each as a fact unit. Document it with a description and solution if applicable. Users often ask, “I got error X, what does it mean?” If the doc has a one-to-one mapping (error code -> doc section), the AI can directly answer that. If errors are only described in a prose paragraph (“various errors might occur when Y fails…”), the AI might not pinpoint the right part.
* **FAQ or How-to scenarios**: As mentioned earlier, frequent tasks or issues should have their own modules. E.g., “Resetting a user password”, “Upgrading from version 1 to 2”, “Configuring TLS”, etc. Each of these scenario-based guides is effectively a factual sequence (a series of steps or an explanation of a process).

Once you identify these fact patterns, the next step is to **module-ize** them in the docs:

* Use descriptive, specific headings (H1/H2 titles) that match the concept or question. If the fact is a definition, the heading should be the term itself (“What is a Replica Set?” could just be “Replica Set” as the heading, with the text explaining it).
* Keep modules concise. A module could be a few sentences to a few paragraphs, but generally not pages long. If it’s too long, consider if it contains multiple facts that should be split. The idea is that each module should ideally answer one question really well. If the answer is too complex, maybe it’s actually multiple questions.
* Ensure each module is self-contained contextually. If a module depends on knowledge from another, either merge them or clearly reference the other. For instance, if you have a module “Transactions in Redis” and it assumes the reader knows what a Redis command is, that’s fine because presumably the AI can fetch definitions too. But within that module, if you use a concept that has its own module, you might want to mention it (e.g., “Redis transactions use the `MULTI` and `EXEC` commands (see respective command reference).”). This way the AI might bring in those references if needed.

**Avoiding overlap and conflict:** With many small modules, one concern is ensuring they don’t contradict each other or go out of sync. This is where having a single source-of-truth fact per module helps. Instead of describing the same feature in multiple places, centralize it and have others reference it. For example, if a certain limitation (say “maximum number of connections = N”) applies system-wide, document it in one module (maybe under a “Limits” section) and have other relevant sections mention that or link to it. Redundancy in wording is not as problematic as inconsistency; the AI might retrieve one or the other, so it’s okay to repeat a fact in multiple modules *as long as it’s consistent*. However, if you update a value, update it everywhere it appears. This is just good documentation practice in general, but especially important when an AI might pull from either place and we don’t want conflicting info.

**Semantic Chunking:** The technical process behind the scenes (in RAG – retrieval augmented generation) is that the doc text is chunked into vectors. We, as documentation architects, can influence chunk boundaries by how we structure the text. If a heading and paragraph go together, that might become a chunk. If one paragraph covers two topics, that’s bad because one chunk may contain both, confusing the retrieval. By making modules as standalone sections, we essentially align our content with how we want chunks to be. An AI can sometimes retrieve part of a chunk, but typically it brings the whole chunk. Thus, we want each chunk to be “clean”. The earlier example from Mintlify illustrated that if you discuss permissions and rate limits in one paragraph under one heading, an AI might return both when only one is asked about. Fact modules solve that: one module for “Updating permissions”, another for “API rate limits”.

**Just-in-time context assembly:** When modules are well-designed, an AI can assemble them to answer multi-faceted questions. Suppose a user asks, “How do I update permissions and what are the rate limits?”. If we have separate modules, the AI can fetch both and present a combined answer. If it was one blob in the docs, the AI might fetch that one blob and give the answer – fine in this case – but if the question was just about one part, it might still fetch the blob and include unnecessary info. With separate modules, the AI has the flexibility to grab one or many as needed. In essence, each module is like a puzzle piece that the AI can use to build a response specific to the query.

**Identification exercise:** A recommended exercise in adopting this doctrine is to review your documentation (or knowledge base) and list the top 50–100 factual questions that one could ask about your product. Then map each question to where in the docs that answer lives. If some answers are not easy to point to (or require reading through paragraphs to find), that’s a candidate for creating a new module or section. This approach ensures you cover the most important ground. For example, in a project like **Linux kernel documentation**, questions might be “What is the scheduler frequency in Linux?” or “How to enable debug mode for driver X?”. If the answers are buried in long text, create an explicit Q\&A entry. Some modern documentation sites (like Microsoft’s docs or MDN for web standards) have already moved toward this modular reference style – each API or error or concept has its own page or at least its own anchor you can link to. We are extending that idea further by also carving out how-to solutions and conceptual explanations similarly.

**Case in point – error handling:** Many support questions revolve around errors. In an AI-first doc approach, you’d maintain an **Error Reference** where each error message or code is an entry with cause and solution. This is gold for an AI assistant: the moment a user mentions an error, the assistant can look it up and provide an explanation and fix. Without such structuring, the AI might try to infer the meaning or combine bits from various places (potentially risky). Having a module per error is a straightforward but high-value practice.

Another example: **configuration keys**. Systems like Kubernetes have YAML keys, Docker has config options, etc. Listing each key with its meaning is helpful. If a user asks “What does `enableAudit` flag do?”, it helps if the docs literally have a line or section “`enableAudit`: when true, enables audit logging, which... (and so on).” If that key is only mentioned in passing in a paragraph, the AI might miss it or not have enough surrounding context to generate a good explanation.

Finally, let’s highlight **versioning** in fact modules. When products have versions, facts can change. One challenge is that an AI might mix information from different versions if the documentation is not clearly separated or labeled. The solution at architecture level is to either maintain separate documentation sets per version (common in docs sites, e.g., “View docs for v1.0 / v2.0”) or to annotate facts with version. The latter can be done within modules like: “*(Added in v2.0, replaced old behavior of X)*”. If using frontmatter, you might tag modules with applicable versions. A well-behaved AI can then use this to contextualize answers. For example, if the user’s context says they use v1.0, the AI should prefer facts tagged for v1.0. If a fact has a note “as of v2.0, this limit increased to 100”, the AI can include that nuance if relevant or at least not confuse the two.

**Example – Redis Modules:** Redis documentation is fairly modular already. Each command (like `ZRANGE`) has its own section with syntax and examples. If applying our doctrine, we ensure each command’s doc is truly standalone (and indeed Redis docs do this well). We might add modules for common “patterns” (e.g., rate limiting using Redis, or caching patterns) which might not be in core docs. Also, an AI-first approach would capture things like “What happens if memory is full?” as a Q\&A in docs, if that’s commonly asked. So the fact “Redis eviction policy” becomes its own article, etc. If implementing, we’d create pages like `maxmemory-and-eviction.md` with frontmatter tags. A user asks the assistant “What happens when Redis runs out of memory?”, it jumps to that module and explains eviction policies, exactly as documented. Without that module, the AI might attempt an answer from general training (which could be outdated or incomplete). By providing the explicit fact module, we ensure the AI’s answer is grounded in the official, up-to-date explanation.

In summary, breaking documentation into **fact modules** ensures that each piece of knowledge about your product is easy for an AI to isolate and retrieve. It’s about aligning documentation units with the natural questions or information pieces a user might need. This granular approach is a cornerstone of preventing AI hallucinations: if the real answer exists in a neat little package, the AI has no reason to hallucinate an answer. And when multiple facts are needed, it can assemble multiple modules logically. We’ve essentially turned our documentation into a collection of LEGO blocks that can be snapped together to answer a wide variety of queries accurately. Next, we’ll address the flip side of accuracy – how to prevent the AI from going off-track even when docs are provided, by using guardrails and verification in the documentation workflow.
