## Markdown-Native Source Document Strategy (with HTML as a peer option)

Before reading this section watch YouTube video [Chunking 101: The Invisible Bottleneck Killing Enterprise AI Projects](https://www.youtube.com/watch?v=pMSXPgAUq_k&t=67s).

All AI-native Q\&A solutions are only as good as the source content they rely on. Thus, our default is a Markdown-first strategy for source documents, with HTML as the canonical alternative when fidelity or semantics require it. We require every knowledge source (policies, manuals, SOPs, FAQs, etc.) to be provided as clean, structured Markdown with YAML frontmatter or standards-compliant HTML with `<meta>` (or JSON-LD) metadata. Both are human-readable, diff-able, and machine-parsable; the choice should reflect complexity:

* Prefer Markdown + YAML frontmatter for simple or moderately structured content authored by non-technical contributors.
* Prefer HTML + metadata for documents that demand rich semantics (complex tables, figures/captions, citations, multi-page/page-anchored structures, accessibility semantics), or where high-fidelity auditability is critical.

The frontmatter (in Markdown) or head metadata (in HTML) should contain key fields such as title, version, author, date, tags, and stable identifiers for citation. For example, a document might begin with:

```yaml
---
title: "Customer Support Handbook"
version: "1.0"
author: "Acme Corp Documentation Team"
date: "2025-07-01"
tags: ["support", "handbook", "customer service"]
doc_id: "cust-support-handbook"
---
```

HTML equivalent:

```html
<!doctype html>
<html lang="en">
<head>
  <meta charset="utf-8" />
  <title>Customer Support Handbook</title>
  <meta name="version" content="1.0" />
  <meta name="author" content="Acme Corp Documentation Team" />
  <meta name="date" content="2025-07-01" />
  <meta name="tags" content="support, handbook, customer service" />
  <meta name="doc_id" content="cust-support-handbook" />
  <!-- Optional JSON-LD for richer semantics -->
  <script type="application/ld+json">
  {
    "@context":"https://schema.org",
    "@type":"CreativeWork",
    "name":"Customer Support Handbook",
    "version":"1.0",
    "author":"Acme Corp Documentation Team",
    "datePublished":"2025-07-01",
    "keywords":"support, handbook, customer service",
    "identifier":"cust-support-handbook"
  }
  </script>
</head>
<body>
  <!-- content -->
</body>
</html>
```

By insisting on Markdown (with proper headings, lists, and tables) or HTML (with proper semantic elements and metadata), we ensure the AI has a pristine, semantically structured knowledge base to work with. The product isn’t just code—it’s trustworthy, well-structured data.&#x20;

Encourage clients to provide documents directly in Markdown whenever possible. Many teams already use it or can export to it. If documents come in other formats (Word, PDF, PowerPoint, Excel), use Microsoft’s MarkItDown to convert to Markdown; then run AI-assisted cleanup to enforce heading levels, list consistency, and table hygiene so the result looks “born-Markdown.” If you discover the content needs richer, typed semantics (e.g., multi-level table headers, figure–caption binding, page-number anchoring, machine-readable citations), opt for an HTML output pipeline instead (you can still provide a Markdown companion for easy editing). Treat imperfect auto-conversions as manual data preparation tasks; a human must verify fidelity.&#x20;

In cases where a document is very complex or the structure doesn’t translate well to Markdown (e.g., multi-sheet Excel, multi-column PDF, diagram-heavy PPT), create custom processing scripts. These can slice content into sections/tables and then emit Markdown or HTML per the decision rule above. Always chunk deterministically—each section/paragraph/element gets a stable identifier (`id` in HTML or an explicit chunk\_id in Markdown comments) for exact citation mapping later. Escalate non-obvious cases rather than guessing; significant human review is expected.&#x20;

By the end of this phase, you should have a repository of all relevant documents in clean Markdown or HTML, each with consistent frontmatter/metadata and structure. This becomes the system’s knowledge base—ready for ingestion with minimal further cleaning and built for verifiable answers tied to well-defined sections.&#x20;

### `surveilr` Makes Markdown and HTML SQL-Queryable

Engineering Requirement: Per-Document Conversion Script and Review

`surveilr` transforms Markdown and HTML into SQLite tables so every part of a document—headings, paragraphs, tables, and metadata—is queryable via SQL.

1. Ingest and Transform
   Documents—either native Markdown or HTML, or converted from PDF/DOCX/etc.—are parsed and stored as discrete rows keyed by CSS/DOM selectors (e.g., `h1`, `p`, `table`, `figure`, `figcaption`).&#x20;

2. Metadata Tables

   * For Markdown, YAML frontmatter is extracted into `uniform_resource.frontmatter`.
   * For HTML, `<meta>` fields (and optional JSON-LD) are extracted into the same schema, providing a uniform metadata view for filtering and search.&#x20;

3. Auditable Transformation Steps
   Every transformation step (engineer, script, original filename) is logged in the history table to preserve traceability.&#x20;

This yields organized, accessible, and auditable content for large-scale querying and governance.&#x20;

While tools like MarkItDown are helpful, complex inputs often need custom pipelines. That work is part of context engineering: every conversion supports stable provenance, section-level metadata, and deterministic chunking—the foundation for citations and trust.&#x20;

For every complex document, prepare a dedicated transformation with these expectations:

1. Create a conversion script and metadata scaffold (TypeScript/Deno preferred; Python acceptable when libraries require it).
2. Embed full provenance in every output chunk (source filename/version, page/sheet, section path, chunk id, timestamp).
3. Preserve semantic structure (heading levels; lists; tables; figures/captions; cross-references).
4. Ensure deterministic chunking with stable IDs (e.g., `refund-policy_sec-5_paragraph-3`).
5. Emit Markdown with frontmatter or HTML with metadata based on complexity and required fidelity.&#x20;

No complex document enters ingestion without this provenance and a human review sign-off. Keep each document’s artifacts (source file, script, reviewed output, audit log) together in a directory.&#x20;

#### Language and Platform Recommendation

We are TypeScript-first on Deno for transformation scripting (modern TS, integrated runtime, clean CLI ergonomics). Use Python where library depth is needed (PDF/Excel/Word inspection). All scripts must be reproducible to produce identical outputs from identical inputs.&#x20;

#### Output Format and Provenance Conventions

Each Markdown chunk should include either embedded comments or file-level frontmatter:

```markdown
<!--
source_file: "CustomerPolicy.docx"
source_page: 12
section_heading: "5.1 Refund Eligibility"
doc_id: "cust-policy"
section_id: "5.1"
chunk_id: "cust-policy_5.1_paragraph_3"
extracted_on: "2025-09-17"
-->
```

```yaml
---
title: "Customer Policy Manual"
version: "2023.4"
source_file: "CustomerPolicy_v3.docx"
doc_id: "cust-policy"
created: "2025-09-17"
tags: ["customer", "policy", "refunds"]
---
```

HTML should carry equivalent metadata and stable anchors:

```html
<section id="cust-policy_5-1_paragraph_3" data-doc_id="cust-policy" data-section_id="5.1"
         data-source_file="CustomerPolicy.docx" data-source_page="12" data-extracted_on="2025-09-17">
  <p>…</p>
</section>
```

```html
<head>
  <meta name="title" content="Customer Policy Manual" />
  <meta name="version" content="2023.4" />
  <meta name="source_file" content="CustomerPolicy_v3.docx" />
  <meta name="doc_id" content="cust-policy" />
  <meta name="created" content="2025-09-17" />
  <meta name="tags" content="customer, policy, refunds" />
</head>
```

This metadata is required to generate human-readable and AI-parseable citations, preserve traceability from LLM output back to the exact chunk, and support audit/compliance/version governance.&#x20;

#### Engineering Roles and Responsibilities

* Expectations Engineers define what must be preserved for answerability and citation.
* Source Document Preparation Engineers script the transformation (Markdown or HTML), verify semantic alignment, and enforce consistency.
* Trust Engineers verify each document’s output meets citation and alignment standards.
* Integration Engineers ensure chunks are indexed, loaded, and addressable by citation ID.&#x20;

#### Why a Script per Document and Not per File Type?

File type is the container; we engineer for knowledge. A “PDF” could be a discharge process or a warranty policy—semantically different artifacts that demand document-specific parsing, provenance, and chunking. We therefore write a custom ingestion script per complex document, independent of whether the output is Markdown or HTML. That discipline yields reliability, explainability, and scalability at inference time: stable identifiers, deterministic chunks, and auditability prevent hallucinations and make citations first-class.&#x20;

---

### Decision Guide (summary)

Use Markdown + frontmatter when:

* Content is straightforward; non-technical teams contribute routinely.
* You need fast authoring, transparent diffs, low ceremony, and easy reviews.

Use HTML + metadata when:

* You need semantic granularity (e.g., `<table>` with `<thead>/<tbody>/<tfoot>`, `<figure>/<figcaption>`, citations, `<time>`, marked references).
* You require explicit header–cell associations, page anchoring, accessibility semantics, or complex layouts.
* You want deep interoperability with DOM tooling and browser-rendered previews.

Neither is categorically “better.” Markdown reduces friction; HTML is the web’s canonical semantic/fidelity layer. Choose the smallest surface that still preserves meaning, auditability, and downstream reliability.&#x20;
