## Context Engineering and AI Prompting First

With a solid Markdown knowledge base in hand, the next focus is Context Engineering – designing how the AI will use the available content to answer questions. Before writing any custom backend code or setting up databases, we should prototype how prompts and contexts will work. The idea is to leverage large language models’ capabilities with our documents *in the prompt* as much as possible, before assuming we need complex pipelines like Retrieval-Augmented Generation (RAG). In other words, we first try to solve the problem with smart prompt design and large context windows, not with infrastructure.

Start by identifying likely user queries and required information using the expectations and UAT scenarios from the *Expectations Engineering* section. We can simulate Q\&A interactions using a high-context LLMs (as shown below) by directly providing relevant Markdown sections in the prompt.

### High-Context LLM Models

| Provider  | Model Name             | Context Length | Best Use Case                                     | Notes                                                             |
| --------- | ---------------------- | -------------- | ------------------------------------------------- | ----------------------------------------------------------------- |
| OpenAI    | `gpt-4-turbo`        | 128k tokens    | Production-grade apps, cost-efficient LLM usage   | Fast, cheaper than GPT-4 base, supports function calling and JSON |
| OpenAI    | `gpt-4-0125-preview` | 128k tokens    | Structured prompts, context-rich interactions     | Often used as the latest GPT-4-turbo variant                      |
| OpenAI    | `gpt-3.5-turbo-16k`  | 16k tokens     | Lightweight tasks, non-critical content retrieval | Risk of hallucination, not suitable for high-trust reasoning      |
| Anthropic | `claude-3-opus`      | 200k tokens    | Highest reasoning accuracy, large document Q&A    | Best Claude model for long-form QA, policies, and reasoning tasks |
| Anthropic | `claude-3-sonnet`    | 200k tokens    | Balanced cost-performance for production          | Good accuracy, lower cost than Opus                               |
| Anthropic | `claude-3-haiku`     | 200k tokens    | Fast, cheap, narrow-scope prompt scenarios        | Great for chat and light document interactions                    |
| Anthropic | `claude-2.1`         | 200k tokens    | Legacy support for long context                   | Older, less performant than Claude 3 models                       |

👉 Tokens can get expensive, be sure to read [Prompt Caching: Don’t Pay for the Same Prompt Again and Again](caching-prompts.md)

Essentially, treat the Markdown files as if they could be concatenated into the prompt (within token limits). Of course, we cannot always stuff entire documents if they are very large – but this exercise helps determine *how much context is actually needed*. For example, if users often ask about “clause 9.2” of a policy, we might prompt the model with: *system message:* “You are an AI assistant. Answer questions using the provided policy document,” and *user message:* “Clause 9.2 question” along with the text of clause 9.2 from the Markdown. This manual prompting test will show what kind of answers the model gives, and whether it uses the context properly.

Design context bundles for each type of query. A context bundle is a collection of document excerpts or facts likely needed to answer a question. For instance, for an “ISO 27001 Q\&A” bundle, it might include the relevant clauses’ text. For a “Customer Support Handbook Q\&A” bundle, it might include the introduction plus the section on the particular topic asked. Initially, do this without any embedding search – simply manually choose the sections (as per UAT scenarios). Through these simulations, you’ll learn how the model behaves: Does it quote verbatim? Does it need additional coaxing to cite sources? Does irrelevant text confuse it? Use these insights to refine the prompt templates and guidelines. Perhaps you discover the model sometimes ignores the provided text and answers from general knowledge – then you know you’ll need to emphasize in the prompt: *“Use only the provided document to answer.”* Or you find that including too much text causes it to get “lost in the middle” – then you know your context windows need to be tighter or chunked differently.

Crucially, do not leap to implementing vector databases or embedding-based retrieval at this stage. We assume for now that large context models or clever chunk selection might handle many queries. Try to solve as much as possible with prompt engineering: this includes using headings or section titles as cues, formatting the context (maybe providing a bullet list of facts rather than raw paragraphs), and crafting the user prompt to be specific. Only if this manual prompting reveals that we *consistently* cannot fit the needed context or the model is struggling to pick the right info, do we consider more advanced retrieval techniques. By approaching the problem prompt-first, we also keep the design simple – there’s elegance in seeing how far you can go with just an LLM and well-structured data. In some cases, you might find that for certain narrow documents, you can indeed just prepend the full text (if under, say, 30k tokens) and the model answers perfectly without any fancy retrieval. That’s a win for simplicity.

Document the outcomes of these prompt experiments. For example: *“When asked about X from Document Y, including sections A and B in the prompt yielded a correct answer with citation, but including section C caused confusion.”* This will inform whether we need to break documents differently or introduce retrieval. Remember, adding vector search or external memory comes with complexity – treat it as a “plan B” unless justified. Industry experts often note that one should not jump onto embeddings/RAG without first trying simpler approaches. Many Q\&A needs are low-volume or can be segmented such that a direct lookup might suffice. Our architecture principle is: start simple, only increase complexity when absolutely required. Shahid or lead architects should review any decision to move beyond prompt-based solutions and approve if, for example, it’s truly needed for scaling or because context windows fundamentally can’t cover the use case. By the end of this stage, we aim to have a clear idea of how the AI will be prompted, what context it needs, and whether those contexts can be supplied as direct text or require a retrieval mechanism. Essentially, we define the “conversation design” here – what the assistant will see before it produces an answer.

### `surveilr` as the Context Composition layer

surveilr’s SQLite-backed design gives engineering teams the ability to generate LLM context bundles the same way they query structured business data — with rigor, visibility, and control. We are not inventing a new language for context. We are simply using the most battle-tested language in computing — SQL — to do the one thing it’s always done best: get the right data, in the right shape, at the right time.

With Markdown as our content layer, and `surveilr` as our _composition_ engine for prompt preparation, context engineering becomes a reproducible process — one that scales, audits, and defends itself — exactly what trustable AI demands.

In traditional LLM application development, context engineering is often treated as a loose, ad hoc activity: developers manually gather content from file systems, copy/paste from various documents, or write brittle scripts to string together chunks of text. These context bundles are typically stored as local files or hardcoded into prompts — with no real visibility, traceability, or reuse across the team.

But in a trust-first, B2B-grade AI system, context is not ephemeral — it’s  a first-class data asset, and it must be handled with the same rigor as source code or structured data. That’s why we introduce a fundamentally different approach:  **context engineering through SQL**, powered by the SQLite-based RSSDs in `surveilr`.

#### SQL as the Language of Context Bundling (_Composition_)

When all Markdown content — and its frontmatter metadata — is ingested into `surveilr`, each document becomes part of a  **queryable knowledge layer** . Each heading, section, paragraph, bullet, and tag is stored in tables, with everything pre-parsed and queryable.

Instead of searching for files on disk or manually slicing text, engineers can _use standard `SQL` `SELECT` statements_ to construct context windows with high precision. For example:

* Select all onboarding policy sections tagged with `sales` and updated in the last 90 days
* Select all security guidelines for `remote access` authored by the compliance team
* Select all SOPs that contain specific keywords but are not marked `internal_only`

No custom code. No guesswork. Just SQL — the universal, declarative language every data-aware engineer understands.

This  shifts context creation from unstructured artifact hunting to structured knowledge retrieval using _context composition via SQL_. It allows engineers to assemble prompt inputs using simple, repeatable rules. These queries can even be versioned, audited, and tested like software components — aligning tightly with our principle that all trustable systems must be inspectable.

While LLMs operate in natural language, context engineering operates in structured logic. By expressing our content filters and retrieval logic in SQL, we gain several advantages:

* Precision: We define exactly what knowledge is relevant using clear WHERE clauses.
* Reusability: The same context selection query can be reused across different prompts, workflows, or products.
* Audibility: We can show exactly how a context bundle was constructed — including document IDs, chunk order, and source metadata.
* Non-technical accessibility: Even non-engineers who understand the basics of querying can read, understand, or even help design prompt scaffolds using simple SQL.

Compare this to the fragile, opaque method of writing file glob patterns or regexes to extract content — approaches that are difficult to debug, cannot be cited, and don’t scale across teams.

#### `surveilr` Makes Prompting Auditable by Default

Because `surveilr` uses SQLite behind the scenes, context bundles can be persisted, diffed, visualized, and recomputed. When a prompt includes five document chunks from three different files, surveilr ensures:

* Each chunk can be traced to its original Markdown file and source format
* Each included paragraph retains metadata like authorship and effective date
* Any changes to those chunks after the prompt was generated can trigger alerts or invalidation workflows

This means every AI prompt becomes an auditable unit — not just in terms of content, but in terms of context and prompt _construction_ and _composition_ logic. We can answer questions like:

* Why was this paragraph included in the context?
* What would happen if we reran the prompt-building SQL today?
* Has this context bundle been verified by compliance?

These are not hypotheticals. These are production requirements in a world where trust must be structured, not assumed.

#### From Prompt Engineering to Prompt Governance

In many AI systems, prompts are fragile art — crafted through trial-and-error and locked into code. With surveilr and SQL, prompts become governed data pipelines. They are no longer handwritten text blocks hidden inside source code. They are the output of live SQL queries over a structured Markdown corpus — meaning we can:

* Automate their creation
* Validate their inputs
* Track their performance
* Explain their rationale

This transforms prompt engineering from a craft into a discipline. We move from designing one-off prompts to managing prompt programs, powered by content-aware SQL logic in `surveilr` SQLite RSSDs.

👉 Tokens can get expensive, be sure to read [Prompt Caching: Don’t Pay for the Same Prompt Again and Again](caching-prompts.md)
