
## Vector Database and Enterprise RAG Strategy

If (and only if) the prompt-first approach above is insufficient, we move to incorporate a Vector Database or other Retrieval-Augmented Generation (RAG) components – but this is treated as an *optional, last-resort step*, not a default. Many teams mistakenly start with a vector DB and embeddings, but our strategy is to avoid that unless scaling or relevance issues demand it. We want to prevent over-engineering. In cases where the knowledge base is very large or questions are very broad (so that providing all relevant text in the prompt isn’t feasible), a vector similarity search can be introduced to fetch the most relevant chunks. Excellent open-source choices include Qdrant, Weaviate, LanceDB, or Milvus, among others. These can index our Markdown chunks (transformed into embedding vectors) and retrieve by semantic similarity. The key rule: use such a vector index only when it demonstrably improves the solution – for example, if UAT tests show the assistant failing because it didn’t have the info, and adding a retrieval step fixes it.

Even when using a vector database, we do not create a custom enterprise RAG platform from scratch unless explicitly authorized. There are many existing tools and frameworks for RAG; we prefer to leverage those (more on that in AnythingLLM section of this document). If a RAG approach is needed, it must be wrapped in what we call a Model Context Protocol (MCP) layer. This means that all operations of retrieving documents and feeding them to the model are done through a standardized interface that keeps track of context provenance. MCP, an emerging open protocol from Anthropic, provides a structured way to connect LLMs with external data or tools. By wrapping RAG in MCP, we ensure that even this retrieval process is *transparent and standardized*. In practical terms, an MCP-compliant retrieval agent might take a query, perform a vector search on our docs, and return a set of snippets with sources, which the LLM then uses to answer. The citation enforcement logic can be part of this – e.g., the MCP could verify that for every answer, sources are included.

It’s worth exploring alternatives before diving into unstructured text RAG. For *structured data* (like databases, CRM data, etc.), consider using tools like Vanna for text-to-SQL. For example, if one of our internal knowledge sources is actually a database (say, customer invoice records or product inventory), it may be better to let the AI generate SQL queries to fetch answers from the database than to convert that DB to text chunks. Tools like Vanna.ai act as an AI SQL agent that writes queries for you. This approach can sometimes answer user questions more directly and accurately by querying the live data instead of relying on stale text. It’s a reminder that RAG is not the only pattern: if the query is about data (“How many open support tickets from client X?”), a direct query via text-to-SQL is cleaner than vectorizing hundreds of ticket texts. We should treat such structured retrieval methods as *first-class options*. In short, use the right tool for the job: unstructured document questions might use vector search, structured data questions might use an NL-to-SQL approach like Vanna, and some questions might even be answerable by simple keyword search or filters on the Markdown (don’t overlook classic search engines or database full-text search in an enterprise context).

&#x20;*Example – Modern LLM Application Architecture: Components include user query inputs, retrieval (vector database or search) for context, prompt construction, the LLM itself, and post-processing like filtering or citing sources. In our strategy, we only introduce the vector DB component if necessary, and always within a controlled MCP framework to maintain traceability.*

### `surveilr` as a Pre-RAG Grounding Layer — Not a Retrieval System

The `surveilr` platform is designed explicitly to _delay or prevent the need for vector databases and RAGs_ by providing a fully structured, SQL-queryable corpus of enterprise knowledge grounded in human-readable, semantically-chunked Markdown.

In traditional RAG-first architectures, teams rush to build embedding pipelines and vector search layers because they lack structured, queryable access to the documents themselves. The assumption is that LLMs require similarity search to find relevant knowledge — but this is often a crutch for _poor upstream document hygiene_.

`surveilr` challenges that assumption directly.

Here’s how `surveilr`, by design, minimizes or eliminates the need for retrieval systems:

#### 1. Human-Structured, Machine-Queried Markdown

Rather than relying on LLMs to discover relationships through vector math, `surveilr` assumes semantic chunking and tagging in Markdown early in the process (just like we discussed above). These chunks, once ingested, are indexed in SQLite tables, making them addressable by SQL, not similarity.

This allows engineers to ask:

> “Give me all onboarding clauses for ‘remote workers’ last updated after April 1st”

…instead of:

> “Find semantically similar chunks to ‘how do we onboard remote salespeople’?”

The former is deterministic and traceable. The latter is probabilistic and opaque.

#### 2. SQL as First-Class Retrieval Layer

In `surveilr`, SQL replaces vector similarity search for many context-engineering needs. Chunks, frontmatter, tags, update timestamps, authorship, sensitivity labels, and logical relationships are all modeled as SQL-friendly fields.

This means _relevance logic is encoded declaratively_, not inferred from embeddings.

* Want the 10 most recently updated policies for a specific customer tier? Use SQL.
* Want all refund rules for products tagged `premium`? SQL again.

This eliminates the need for RAG unless the queries become unbounded or cognitively ambiguous.

#### 3. Context Windows Constructed by Query, Not Search

Most modern LLMs have large enough context windows (e.g., 32K–200K tokens) to handle entire filtered context bundles. `surveilr` enables those bundles to be created and scoped via SQL — not embeddings.

The context, therefore, is not retrieved by “semantic proximity,” but filtered by explicit B2B criteria, such as document lineage, relevance tags, or regulatory role decided during the expecations engineering and UAT preparation phases.

This means every answer generated can include citations that trace directly to document IDs and chunk numbers — an outcome nearly impossible to guarantee with vanilla RAG.

### When (and Only When) RAG Becomes Necessary

The strategy defined in our architecture — and supported by `surveilr` — is that _vector retrieval is a fallback_, invoked only if:

* UAT tests reveal context insufficiency
* Prompt-only bundles exceed the token window
* Users need to ask exploratory or edge-case questions that aren’t directly covered by structured tags or SQL-accessible metadata

In such cases, `surveilr` doesn’t obstruct RAG — it *prepares it*.

This turns RAG into a tightly controlled augmentation layer, not a fulltext search substitute.

> `surveilr` ensures that LLM-based systems **don’t need RAGs unless rigorously justified**.

It enforces:

* High-fidelity chunking at the heading, sentence or paragraph levels
* Declarative filtering using SQL
* Audit-safe provenance by using SQL queries input and query results
* SQL-driven transparency for non-engineering and human analyst comprehension

Whereas most vector systems try to guess what’s relevant, `surveilr` ensures you know what’s relevant before you embed.

In your architecture, RAG is a backup strategy for fuzzy retrieval. `surveilr` is the reason you rarely need that backup.

### How to decide when to use vector/RAGs

To summarize the vector/RAG strategy: keep it as simple as possible for as long as possible. Only add a vector database after demonstrating via written justification (to Shahid or architects) that we need it. If approved, implement it using well-established open-source solutions and wrap its usage in an MCP or similar context manager that logs what was retrieved and ensures the LLM sees only vetted context. Consider RAG a “power tool” – useful, but you shouldn’t grab it for every job by default. Many problems can be solved with properly structured data and prompts. When we do need RAG, we’ll implement it in a way that is modular and safe: the retrieval component can be swapped or audited independently of the LLM, because it follows a protocol. This will help in future maintenance and for debugging (we can check if a failure was due to retrieval vs generation easily). Finally, always test the integrated system against the UAT cases again: if we add a vector DB, our AI must still pass the same acceptance tests (only now the pipeline is a bit more complex under the hood). The answers should still come with citations and match the expected results.

## Chat UI Construction

While the backend does the heavy AI lifting, the end-users interact with the system through a chat interface. We will not build this UI from scratch; instead, leverage existing open-source UI kits that already implement chat conversation patterns. Two recommended options are the `assistant-ui` library (a TypeScript/React component set for AI chat UIs) and Kibo UI’s AI conversation components. These provide pre-built chat bubbles, user input boxes, scrolling conversation windows, etc., that follow modern design practices. For instance, `assistant-ui` is a React component library specifically geared towards AI assistants (multi-turn chat, message styling, etc.), and it’s built to be easy to drop into a Next.js or React project. Kibo UI similarly offers an `<AIConversation>` component which wraps a list of messages and handles auto-scrolling and branch management of chat histories. Using these saves us a ton of time and also ensures consistency with common UI/UX patterns users expect (like newest messages at the bottom, an input box that grows for long queries, etc.).

All our UIs will follow the design system principles of shadcn/ui (a popular library of accessible React components styled with Tailwind CSS). Both `assistant-ui` and Kibo UI are compatible with or inspired by shadcn’s approach, meaning they use consistent typography, spacing, and dark/light mode handling. This will make our chat UIs not only functional but also clean and modern-looking without extensive custom CSS. We will incorporate Tailwind CSS utility classes for custom styling as needed, but heavy design work is not necessary since these kits come with a good default style.

When building the chat UI, focus on functionality and integration rather than fancy visuals. Key features the UI must support include: a scrollable chat history pane that can display both user questions and AI answers (with formatting like markdown, code snippets, and inline citations), the ability to show sources (for example, as footnote numbers that on click show the source text or link), and an input box with support for multi-line queries (since user questions might be long). Additionally, think about support for resetting the conversation, or showing multiple conversation threads (if a user wants separate contexts). The open-source kits often have these features or examples. For instance, Kibo UI’s conversation component automatically adds a “scroll to bottom” button when new messages come in and the user has scrolled up – such subtle features enhance usability, especially for long Q\&A sessions.

Crucially, the frontend should not directly call LLM APIs like OpenAI or Anthropic from the browser. Instead, it will communicate with our backend via APIs (most likely a REST or GraphQL endpoint that we design as part of the MCP backend). The UI sends the user’s question to our server, the server uses the MCP/LLM logic to get an answer with citations, and then the UI displays the answer. This separation ensures we can easily swap the backend model or add logging and authentication on the server side. It also avoids exposing any secret keys or internal data to the client code. Essentially, treat the UI as a dumb terminal – all intelligence is on the server. The UI will probably call an endpoint like `POST /api/ask` with the question and maybe a conversation ID, and then render whatever answer and sources come back.

By following a component-based UI kit and the `shadcn/ui` patterns, even a small team of 1-2 frontend developers can get a polished chat interface running quickly. We should also prepare some basic theming (matching the client’s branding, if needed) and internationalization consideration if the users might not be English speakers – but since our context is B2B internal users, a simple English interface is often acceptable initially (still, keep it in mind if deploying in India ODC context – maybe supporting Hindi or regional languages if asked). The main point is we don’t reinvent the wheel on UI. Use what’s already available (open source components) and focus our creativity on the AI and integration logic. For completeness, we will also include typing indicators, error messages (if the backend is down or returns error), and loading spinners as needed, but again most UI kits have patterns for these. The UI should remain simple and clear: a header, the conversation window, user input at bottom, and maybe a sidebar for context or settings (optionally, for example to let user pick which document or knowledge base to use – but that can be a later enhancement). At this stage, a straightforward chat experience that connects to our backend is the goal.

## Backend Design with MCP and AnythingLLM

Now we detail the backend architecture that powers the AI system. The backend is essentially the brain of the operation: it will handle user queries, orchestrate the retrieval of context (if using RAG or other sources), call the LLM, and return the answer with citations. We enforce a strict architecture pattern here: every request should go through a Model Context Protocol (MCP) orchestrator or an equivalent agent mechanism. This means rather than calling an LLM API directly with a prompt, we wrap the call in an “agent” that knows about the available tools (our documents, search index, etc.) and the rules (e.g., always cite sources). Think of MCP as the traffic controller ensuring the AI uses the right routes to get to an answer. As mentioned, MCP is an open standard (backed by Anthropic and community) that standardizes how LLMs interface with external tools and data. By using MCP, we gain compatibility with many existing tools and a clear structure for adding new ones.

In practice, using MCP in our backend could look like the following: we spin up an MCP server (or use an existing one, since AnythingLLM supports connecting to MCP servers) which might have tools like `document_search` or `sql_database` registered. When a query comes in, the MCP layer decides, based on our instructions, whether to just prompt the LLM with context or to invoke a tool. For example, if the query is “How many support tickets were closed last month?”, the MCP could route this to a tool that queries the support database (if integrated). If the query is “Explain clause 9.2 of ISO 27001,” the MCP might call a `document_search` tool that looks up “clause 9.2” in the vector index and returns the text, then feed that into the LLM prompt. All these operations are defined in a prompt logic script or configuration that we write. We basically script the AI’s thought process: first look here, then call model, etc. The key is *determinism and auditability* – the sequence of steps the AI takes should be logged and debuggable (no hidden chain-of-thought that we can’t see). MCP encourages that by making the interactions explicit.

We will use AnythingLLM as our default backend engine. AnythingLLM is an open-source platform that provides ingestion, embedding, and querying of private data, along with a convenient API and UI for chat. It’s essentially a ready-made RAG + agent orchestration framework. It can handle chunking documents into “workspaces,” embedding them, and even includes a simple chat UI and multi-user management. We’ll leverage it primarily for its robust ingestion (it supports PDF, TXT, DOCX, etc., converting them similarly to our Markdown approach) and its agent plugin system. Notably, AnythingLLM is fully MCP-compatible – it allows plugging in external MCP servers and tools easily. This means our backend can integrate with other services (e.g., a web search, or a custom API) via MCP without custom wiring; AnythingLLM will manage running those tools when the LLM requests them. By using AnythingLLM, we avoid reinventing a lot of low-level features like vector DB connectivity, LLM API integrations, chat history handling, etc. It already supports many LLM providers (OpenAI, Azure, local models via Ollama, etc.) and vector stores, which we can configure as needed.

That said, the team must still write some custom logic – particularly around prompt construction, post-processing, and citation enforcement. We will implement an MCP “agent” or wrapper that ensures each answer going out meets our trust requirements. For example, after the LLM generates an answer, we might have a step to scan the answer for any statements not backed by the provided context. If something looks suspicious or outside context, the agent could either reject it or append an “\[Source?]” note for transparency. We can also enforce a format: e.g., instruct the model (in the system prompt) to answer in markdown and include citations like 【DocName†Section】 for each answer. If the model doesn’t include citations, the backend could refuse to output the answer (or better, append the sources itself by matching text spans, though that’s advanced). The idea is that the backend is not a dumb relay – it’s an intelligent middle layer that guarantees the AI’s output conforms to our expected structure (especially regarding trust). This is what we mean by being *“Trust Engineers”* – writing code that wraps the AI and checks its work.

If at any point the team feels they need to deviate from using MCP/AnythingLLM – for instance, writing a completely custom retrieval pipeline or a new agent system – they must consult Shahid. The default stance is to compose existing tools, not create new ones. The combination of AnythingLLM + MCP + our Markdown corpus should suffice for most needs. We configure AnythingLLM with our documents (ingest the Markdown files as a workspace), possibly configure its vector DB (e.g., use Qdrant) if needed, set up any MCP servers (MarkItDown even has an MCP server for file conversion, though we did conversion offline; or a web search MCP tool if needed for external info), and then focus on the prompts/agents.

To outline the request flow: when a user asks a question on the UI, it hits our `/api/ask` endpoint. The backend (using AnythingLLM’s API or SDK) will take that query in a given workspace (a workspace might correspond to a collection of documents or a project) and run it through the agent. The agent’s logic says: (pseudo-code) “Given user query, search the docs for top 3 relevant sections. Form a prompt with those sections and the user question. Call the LLM. If LLM responds with an answer, return that. If answer lacks citations or confidence, possibly iterate or return an error.” All this would happen in a few seconds (depending on LLM speed). The user then sees the answer with footnotes. Because we use AnythingLLM, we get features like clear citations in responses by default (its UI highlights sources, etc.). This aligns perfectly with our trust requirements.

Logging and monitoring will also be set up at this layer – we’ll log each query and the steps taken (search terms, which documents were retrieved, etc.). This helps in debugging when the AI gives a wrong answer (we can see maybe it retrieved the wrong text) and is useful for future audit or improvement cycles.

In summary, our backend is built *around* an MCP-oriented architecture: it’s modular, each piece has a defined role (retriever, LLM, agent logic), and we reuse the robust AnythingLLM framework to avoid coding from scratch. The team should gain familiarity with AnythingLLM’s configuration and plugin system, as well as the basics of MCP (which is essentially running tools in response to model’s needs). By routing all AI interactions through this controlled pipeline, we ensure consistency (every query goes through the same steps), debuggability, and the ability to extend functionality easily (e.g., add a new data source tool in MCP without overhauling the system). This design reflects the principle that we integrate and orchestrate – we don’t build new ML algorithms. We rely on LLMs as a service and existing libraries, and our value-add is the clever wiring and guardrails around them.