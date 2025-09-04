## Multi-Layered Retrieval for Verifiable AI

To further improve grounding, reduce hallucination, and enforce factual verifiability in AI-native Q\&A systems, our architecture incorporates a layered retrieval strategy. This combines full-text search, semantic search, ontological reasoning, and knowledge graphs — each designed to enhance the system’s ability to produce traceable, citation-backed answers.

This section outlines when and how to incorporate each retrieval modality, and how to fit them into the AI-native toolchain using the Model Context Protocol (MCP) and AnythingLLM.

### Full Text Search (FTS): High-Precision Grounding Layer

Full text search (FTS) is the most deterministic retrieval strategy and should be implemented as a first-pass retriever. It excels at:

* Matching verbatim phrases or structured queries.
* Answering compliance-related or section-specific questions (e.g., “What does Clause 9.2 say?”).
* Supporting auditable citations by returning exact text locations.

#### Implementation Guidelines

* Use open-source tools such as Typesense, Meilisearch, or PostgreSQL FTS.
* Index Markdown and HTML chunks with identifiers and section-level metadata (`doc_id`, `heading`, `slug`, etc.).
* Wrap search via an MCP tool (e.g., `fts_lookup`) that:

  * Accepts user queries.
  * Returns top-ranked matching chunks.
  * Logs all retrieved spans for auditability.

FTS results should always be available to the LLM and used preferentially for any direct factual queries. This ensures maximum transparency and source fidelity.

### UI Framework Strategy

As part of the NAT system’s ongoing evolution, we evaluated two user interface libraries, Assistant-UI and MCP-UI, to determine the best fit for delivering a flexible and intuitive user experience. Based on our initial implementation review and team feedback, we propose the following approach:

### Preferred Option: Assistant-UI

Assistant-UI currently offers a higher degree of flexibility in designing dynamic and interactive UI elements aligned with our product goals. It supports modular composition, faster iteration cycles, and seamless integration with our current architecture. As such, we propose using Assistant-UI as the primary interface for NAT.

### Complementary Role for MCP-UI

While MCP-UI is tailored to Model Context Protocol environments, its rigidity and limited customization options present constraints for certain use cases. However, MCP-UI is being evaluated for targeted integrations, particularly where alignment with MCP-native workflows is essential. We see value in selectively leveraging MCP-UI components in contexts that demand protocol adherence or prebuilt visualization support.

### Combined Approach

The team recommends a hybrid strategy:

- Primary UI: Assistant-UI will drive the core interface development.
- Selective Use of MCP-UI: MCP-UI components will be integrated where technically beneficial or required by MCP-bound flows.

This approach allows us to balance flexibility and compliance while keeping the door open for evolving UI needs. We will continue prototyping with both libraries and revisit this strategy as the system matures.

### Semantic Search (Vector Search): Relevance-Based Fallback

Semantic search via vector databases provides approximate, meaning-aware matching and should be treated as a fallback when FTS yields no or insufficient results.

#### Use Cases

* When queries use paraphrased language not present verbatim in the corpus.
* When exploring broader concepts (e.g., “How do we handle client onboarding?”).

#### Implementation Notes

* Use tools like Qdrant, Weaviate, or LanceDB.
* Embed Markdown and HTML chunks with positional metadata.
* Integrate via an MCP tool (e.g., `vector_lookup`) configured to:

  * Return top-k semantically relevant chunks.
  * Map embeddings back to their original source Markdown and HTML sections.

> Important: Whenever semantic results are used, LLMs must still cite based on source chunk IDs (not just semantic embeddings) to preserve traceability.

### OWL Ontologies (Protégé): Structured Meaning and Inference

In domains where conceptual clarity and regulated relationships are critical (e.g., legal, healthcare, compliance), implement ontologies using OWL (Web Ontology Language) and tools like Protégé.

#### Why Use Ontologies

* Define formal relationships (e.g., “Data Processor ⊆ Third Party”).
* Disambiguate user queries by entity normalization.
* Enable automated inference (e.g., “If X is a type of Y, then Z applies”).

#### Integration Strategy

* Build OWL ontologies with Protégé and export to RDF format.
* Use a semantic reasoning engine (e.g., Apache Jena, Stardog).
* Expose reasoning queries through an MCP tool (`ontology_query`):

  * Given a question, extract terms/entities.
  * Return related classes, hierarchies, or facts as context enrichment.

This structured data layer allows LLMs to reason over concepts instead of relying solely on lexical or embedding matches.

### Knowledge Graphs (KG): Navigable Evidence Structures

To capture cross-document relationships and user-intelligible explanations, incorporate lightweight knowledge graphs built from Markdown or HTML source documents and UAT scenarios.

#### Benefits

* Supports multi-hop reasoning across documents.
* Provides explanation paths (e.g., “Complaint → Support Policy → Refund Clause”).
* Facilitates evidence graphs for users who ask “Why is this true?”

#### Integration Approach

* Build knowledge graphs using Neo4j, Memgraph, or RDF triple stores.
* Nodes represent: documents, sections, concepts, entities.
* Edges represent: “references”, “is part of”, “is regulated by”, etc.
* Add an MCP-compatible tool (`graph_lookup`) to:

  * Answer relationship queries.
  * Return path explanations or ranked related nodes.

LLMs can use these graphs to generate more coherent and explainable responses — especially useful in “how” and “why” questions.

### Layered Retrieval Architecture: When and How to Use Each Layer

| Retrieval Layer | Primary Role                          | Tooling                | MCP Tool Example   | Use When...                                 |
| --------------- | ------------------------------------- | ---------------------- | ------------------ | ------------------------------------------- |
| FTS             | Precise, traceable matching           | Typesense, Meilisearch | `fts_lookup`     | User query matches source text closely      |
| Semantic Search | Approximate conceptual matching       | Qdrant, Weaviate       | `vector_lookup`  | Paraphrased or abstract queries             |
| Ontology (OWL)  | Rule-based inference, disambiguation  | Protégé, Jena        | `ontology_query` | Conceptual questions or taxonomy required   |
| Knowledge Graph | Explanations and relationship tracing | Neo4j, Memgraph        | `graph_lookup`   | “How does X relate to Y?” style questions |

### Practical Integration Sequence

1. Index all documents into both FTS and vector DB, with references to chunk ID and Markdown or HTML path.
2. Define entity classes and relationships in Protégé, covering terms found in documents.
3. Derive a lightweight knowledge graph using markdown or HTML section headers, cross-references, and UAT insights.
4. Wrap each retrieval layer in a separate MCP tool, each with logging and fallback capability.
5. In your LLM orchestrator (e.g., AnythingLLM):

   * First invoke `fts_lookup`, return top-N.
   * If inadequate, try `vector_lookup`.
   * For disambiguation or regulatory logic, invoke `ontology_query`.
   * For user-facing explanations, invoke `graph_lookup`.
6. Consolidate all retrieved context into the LLM input window, with priority given to *traceable and cited sources*.

### Summary: Retrieval Stack as AI Grounding Fabric

Trustworthy AI Q\&A systems require more than semantic embeddings. By stacking retrieval methods — beginning with precision, escalating to semantics, and grounding with structured logic — we deliver:

* Factual outputs
* Cited answers
* Explainable reasoning paths
* Auditable traceability

This retrieval strategy becomes the grounding fabric that AI models operate over, making hallucinations less likely and user trust significantly higher.