## Markdown-Native Source Document Strategy

Before reading this section watch YouTube video [Chunking 101: The Invisible Bottleneck Killing Enterprise AI Projects](https://www.youtube.com/watch?v=pMSXPgAUq_k&t=67s).

All AI-native Q\&A solutions are only as good as the source content they rely on. Thus, a Markdown-first strategy for source documents is mandatory. We require every knowledge source (policies, manuals, SOPs, FAQs, etc.) to be provided as clean, structured Markdown files with YAML frontmatter metadata. Markdown is human-readable, simple, and preserves the document structure (headings, lists, tables) in a consistent way. The frontmatter (the section enclosed in `---` at the top) should contain key metadata such as title, version, author, date, and tags, which helps in organizing and filtering content later. For example, a document’s Markdown might begin with:

```yaml
---
title: "Customer Support Handbook"
version: "1.0"
author: "Acme Corp Documentation Team"
date: "2025-07-01"
tags: ["support", "handbook", "customer service"]
---
```

Followed by the content in Markdown. By insisting on Markdown (with proper headings for sections, bullet lists, etc.), we ensure the AI has a pristine, semantically structured knowledge base to work with. Well-structured data is what users ultimately value, as it drives trustworthy AI outputs. In fact, modern engineering philosophy recognizes that the *product isn’t just code – it’s well-structured, trustworthy data that drives decisions*. Our AI systems can only be as reliable as the documents we feed them.

Encourage clients to provide documents directly in Markdown whenever possible. Many technical teams already use Markdown or can export to it. If documents come in other formats (Word, PDF, PowerPoint, Excel), use Microsoft's MarkItDown tool to convert them to Markdown. MarkItDown is an open-source Python utility that converts Office files and PDFs into Markdown while preserving the important structure like headings, lists, tables and more. It focuses on producing Markdown that’s friendly for LLM consumption, retaining document hierarchy and content in a clean text form. (Under the hood, MarkItDown even offers an MCP server for integration with AI workflows.) After conversion, run AI-assisted cleanup and formatting checks on the Markdown: ensure headings are properly nested (e.g., all level-2 headings start with `##`), lists use consistent `-` or `1.` notation, tables are formatted correctly, etc. The goal is that each document looks like it was *originally authored* in Markdown. If the auto-conversion isn’t perfect, treat this as a manual data preparation task – the team must edit and fix the Markdown by hand. This step is crucial; it’s where you engineer the context the AI will later use.

In cases where a document is very complex or the structure doesn't translate well to Markdown (for example, a large Excel with many sheets, or a PDF with multi-column layout), be ready to create custom processing scripts. These could be small Python or Deno programs to slice the content into sections or tables, then output Markdown. For instance, if dealing with an Excel file of financial data, you might write a script to convert each sheet into a Markdown table or CSV, and then link them. Always chunk documents in a deterministic, auditable way – e.g., each section or paragraph gets a stable identifier or HTML anchor in the Markdown. This is important for later citation: we want to be able to trace any AI answer back to the exact file and section it came from. The team should document how each format is converted and chunked. If any source cannot be converted in a straightforward manner, escalate to Shahid for guidance rather than guessing. Significant human review of the converted Markdown is expected; while AI can assist (e.g., with formatting suggestions or by highlighting sections that seem garbled), ultimately a human must verify that the Markdown version faithfully represents the source content. Preparing the documents is manual labor assisted by AI, not fully automated – this investment is necessary because it directly affects answer quality later.

By the end of this phase, you should have a repository of all relevant documents in clean Markdown form, each with consistent frontmatter and structure. This will be our system’s knowledge base. Every document is now ready to be ingested or referenced by LLMs with minimal further cleaning. In short, garbage in, garbage out – so we put the time in upfront to ensure inputs are as structured and clear as possible. Not only does this make the AI’s job easier, but it also sets the stage for verifiable interactions and answers (since we can map interactions and answers back to well-defined sections of text).

### `surveilr` Makes Every Part of Markdown SQL Queryable

Engineering Requirement: Per-Document Conversion Script and Review

surveilr allows engineers to easily query and manage Markdown documents by transforming them into  SQLite tables . This process makes every part of the document—whether it's  headings ,  paragraphs ,  tables , or  metadata —queryable using simple SQL queries. Here’s how it works:

1. **Ingest and Transform Markdown** :

* Documents, whether in Markdown format or converted from other formats like  PDF ,  DOCX , etc., are ingested into  SQLite . The content is parsed into HTML-formatted Markdown and stored in separate rows based on CSS selectors (like `h1`, `p`, `table`, etc.).

2. **Metadata Tables via YAML Frontmatter** :

* YAML frontmatter in the Markdown document (e.g., title, version, author) is extracted into a dedicated metadata table (uniform_resource.frontmatter). This allows powerful filtering and searching by metadata fields like  author ,  version , and  tags.

3. **Auditable Transformation Steps** :

* Every transformation step (such as which engineer processed the document, the script used, and the  original filename ) is logged into a transformation history table (`uniform_resource.frontmatter`), ensuring traceability and accountability.

By structuring documents this way, **`surveilr`** ensures that content and metadata are  organized ,  easily accessible , and auditable through SQL, making it simpler to query, track, and manage large sets of documents.

While many well-structured source documents can be automatically converted into Markdown using utilities such as Microsoft’s [markitdown](https://github.com/microsoft/markitdown), this strategy often fails for complex inputs. In practice, PDFs with visual layout quirks, PowerPoint decks with diagrams and notes, Word documents with nested formatting, and Excel spreadsheets with multi-tab logic do not translate cleanly. For these cases, AI-native engineering teams must prepare a custom script per complex document to ensure traceability, semantic fidelity, and citation-ready outputs.

This work is not a one-time transformation—it is an essential part of the context engineering discipline. Each document conversion is a structured data engineering task that must support stable provenance, section-level metadata, and deterministic chunking. The Markdown that results from this process is the foundation for verifiable interactions and answers, citation enforcement, and user trust.

For every complex document, the engineering team must prepare a dedicated transformation process with the following expectations:

1. Create a dedicated conversion script and metadata scaffold per document, using TypeScript (preferably in Deno) or Python as needed.
2. Embed full provenance in every output section of the resulting Markdown, including:

   * Source filename and version
   * Source page or sheet reference (if applicable)
   * Section identifier and hierarchical heading path
   * Chunk identifier and creation timestamp
3. Capture semantic structure manually where necessary, preserving:

   * Heading levels and logical grouping
   * Lists, tables, and embedded narrative order
   * Figure captions, embedded links, or cross-references
4. Ensure deterministic chunking, so that every output chunk receives a stable, addressable ID (e.g., `refund-policy_sec-5_paragraph-3`).

No complex document should enter the AI ingestion pipeline unless this provenance is in place, validated by human review.

#### Example: Directory Structure for Conversion Artifacts

```
doc-processing/
  invoice-policy-2023/
    source.docx
    convert-to-md.ts
    invoice-policy.md
    audit-log.md
```

Each source document has a dedicated directory that includes:

* The original source file
* A TypeScript or Python script used to extract and format the content
* The resulting reviewed Markdown file
* An audit log explaining any structural decisions, formatting substitutions, or known caveats

#### Language and Platform Recommendation

We are a TypeScript-first and TypeScript-native engineering team, and Deno is our preferred platform for document transformation scripting because of its:

* Native support for modern TypeScript
* Integrated runtime (no dependency sprawl or complex setup)
* Simpler file access and standard library for CLI tools
* First-class support for readable and auditable scripting in collaborative teams

If the engineering team has stronger expertise in Python, or if high-quality libraries (e.g., for PDF parsing, Excel decoding, or Word XML inspection) are only available in Python, it is acceptable to use Python for those cases. However, scripts should follow the same principles:

* Deterministic chunking
* Metadata annotations
* Logged and repeatable output with auditability

All scripts, regardless of language, must support reproducibility: any engineer should be able to rerun the transformation with the same inputs and produce the same Markdown output, enabling consistency for downstream citation and retrieval.

#### Output Format and Provenance Conventions

Each chunk of the output Markdown should include either:

* Embedded comments in the form:

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
* Or YAML frontmatter for entire documents:

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

This metadata is required to:

* Generate human-readable and AI-parseable citations
* Allow traceability from LLM output back to document chunk
* Support audit, compliance, and version governance requirements

#### Engineering Roles and Responsibilities

* Expectations Engineers define what content must be preserved for answerability and citation.
* Source Document Preparation Engineers script the transformation, verify semantic alignment, and enforce structural consistency.
* Trust Engineers verify that each document’s Markdown output meets citation and alignment standards.
* Integration Engineers confirm that all document chunks are properly indexed, loaded into the retrieval pipeline, and addressable by citation ID.

#### Why a Script per Document and Not Document Type?

In traditional software workflows, engineering teams often standardize processing based on document type — such as `.pdf`, `.docx`, or `.xlsx` — and attempt to build generic tools or one-size-fits-all pipelines for each file format. In an AI-native development model, this abstraction is not only inadequate — it is actively harmful to trust, explainability, and alignment.

##### Document Type Is a File Format; AI Needs Knowledge Format

In an AI-native system, the file type is merely the container. What matters is not that a document is a “PDF” or a “Word file” but that it is a knowledge object produced by a specific organization, for a specific purpose, in a specific structure, intended to guide human behavior or organizational operations.

For example:

* One PDF might contain a hospital's discharge planning process.
* Another PDF might encode an equipment warranty return policy.
* A Word document might define a legal indemnification clause.
* An Excel file might map billing codes to reimbursement tiers.

Technically, these may all be `.pdf`, `.docx`, or `.xlsx` — but in reality, each is a semantically unique artifact that demands individual attention and interpretation.

##### Why We Build a Script per Document, Not per Format

In an AI-native pipeline, we write a custom ingestion and transformation script for each document — even if multiple documents come in the same file format. The reason is simple: we are not processing files — we are processing meaning.

Each complex document contains:

* Its own idiosyncratic logic, layout, and headings
* Unique assumptions, business context, and user expectations
* Non-obvious structures (e.g., nested lists, semantic tables, cross-references)

Generic PDF-to-Markdown or Word-to-Markdown converters do not understand these semantic layers. Treating a PDF as just another file type ignores its role as a knowledge artifact. This is especially true in the B2B domain, where internal documentation, contracts, operational policies, and financial schedules are authored with high specificity and often with embedded intent.

##### Knowledge Containers Require Knowledge-Specific Engineering

We treat every complex document as a knowledge container, not a generic file. This means:

* A customer onboarding manual in PDF form is not "a PDF"; it is a business process narrative.
* An Excel file mapping products to SKUs is not "an Excel spreadsheet"; it is a pricing ontology.
* A Word file explaining escalation protocols is not "a Word doc"; it is a risk mitigation strategy.

Each of these deserves a purpose-built script that extracts meaning with precision, builds stable identifiers for citations, and encodes the logic that a large language model will later be asked to reason over.

##### Implications for Engineering Practice

* We do not generalize ingestion by file type.
* We generalize engineering discipline, not tooling: every document receives custom parsing and provenance tagging, but all follow the same expectations:

  * Deterministic chunking
  * Stable section identifiers
  * Embedded auditability
  * Reviewable transformation logic

This approach is slower at the front but vastly more reliable, explainable, and scalable at inference time — especially as downstream AI systems rely on well-prepared context to prevent hallucination, enforce citation, and build user trust.

In an AI-native system, the concept of document type is obsolete. It is a convenience for file systems, not for LLMs. What matters is not the `.pdf` or `.docx` extension but the unique, structured knowledge embedded inside. That knowledge must be made machine-reasonable, citation-compatible, and human-auditable — and that is why we write a script per document, not per file format.

Each document is its own schema. Each script is its decoder. The file format is incidental. The knowledge is what matters.
