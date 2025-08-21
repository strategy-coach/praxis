# Custom "Script-per-Document" Technology Strategy

Ensure every *complex* knowledge artifact is transformed with a purpose‑built, auditable script that preserves meaning, not just text—using docling for layout‑aware parsing, lossless structured exports, and robust table/figure handling—then ingested into surveilr with deterministic chunk IDs and full provenance for trustworthy retrieval and citations. Docling provides a Python API and CLI, supports PDF/DOCX/PPTX/XLSX and more, produces Markdown/HTML/lossless JSON, and integrates easily with RAG tooling; it’s MIT‑licensed and designed to run efficiently. ([Docling][1], [IBM Research][2])

<mark>This entire file is to be reviewed by AI Workforce team and surveilr team. The footnotes are temporary, mainly for use during research and writing and can be removed unless quotation marks are required or copy/pasting from sources.</mark>

Keep Markdown‑first for simple cases, but adopt “script per document” for complex artifacts. Use docling to capture the real structure—reading order, tables, figures, notes—and always emit both human‑auditable Markdown and lossless structured JSON with deterministic chunk IDs and full provenance. Store everything in surveilr/SQLite for queryability and trust. This up‑front rigor buys you reliable retrieval, enforceable citations, and answers you can defend. ([Docling][1])

## When to use docling vs. MarkItDown

Use MarkItDown for straightforward conversions where “clean text + basic structure” is enough, and when MCP integration is desirable for agent workflows. Use docling whenever you need layout‑aware parsing, stable reading order, high‑quality table extraction, or lossless, richly structured JSON/DocTags for downstream comprehension. ([GitHub][3], [Microsoft for Developers][4], [Docling][1])

Typical docling triggers:

* PDFs with multi‑column layout, dense tables, footnotes/headers, code, formulas, or embedded images that matter. ([Docling][1])
* Contracts/SOPs where section hierarchy and cross‑references must be preserved for citations.
* PowerPoint decks where slide notes and visual grouping carry meaning.
* Excel workbooks where sheet structure and header hierarchies must map to first‑class data. ([Docling][1])
* Scanned or mixed‑quality documents requiring OCR within a unified pipeline. ([Medium][5])

## Architecture overview: Docling‑driven per‑document transformation

### Intake and triage

* Identify document intent (“knowledge object”), critical answerable elements, and citation needs (pages/sections/tables).
* Decide docling vs. MarkItDown; default to docling if layout/semantics are nontrivial. ([Docling][1])

### Per‑document script (Python primary, orchestrated by `surveilr` CapEx)

* Write a one‑off Python script that invokes docling’s parser (CLI or API) to produce dual artifacts: canonical Markdown for human reading and lossless JSON/DocTags for machine reasoning. ([Docling][1])
* Wrap with a `surveilr` *capturable executable* orchestrator to standardize inputs/outputs, logging, and reproducibility in the repo (keeps us TS‑first without losing docling’s strengths).

### Normalization and enrichment

* Reconstruct reading order; preserve heading hierarchy; resolve figures/captions; extract tables with structure (not just rendered text). ([Docling][1], [IBM Research][6])
* Add domain‑specific annotations (e.g., “Definition: ‘Covered Entity’”, “SLA: P1 response time”).
* Emit deterministic chunk IDs and embed provenance in each chunk.

### Ingest into surveilr

* Store Markdown, plus the lossless JSON/DocTags, into uniform\_resource.
* Populate frontmatter and chunk‑level lineage (file name/version, page/sheet, bbox when available).
* Expose everything via SQLite so every element is queryable.

### QA and promotion

* Run golden checks (counts of h1/h2, tables, figures), diff fingerprints, and human review before promoting to the knowledge base.

# Per‑document script blueprint

- [ ] TODO: devise a directory convention, something like below

```
doc-processing/
  <doc-key>/
    source.<ext>
    transform.surveilr-SQL.py     # docling-based, emits SQL? // TODO
                                  # <doc-key>.md uniform_resource row
                                  # <doc-key>.docling.json uniform_resource_transform row
                                  # <doc-key>.docdoc.py uniform_resource_transform row    
```

Inputs and outputs

* Inputs: original file plus a YAML “expectations” file defining the knowledge contract (what sections/tables must be extractable).
* Outputs:
  * Human‑readable Markdown
  * Lossless docling JSON/DocTags for machine use
  * A manifest with structural counts and QC metrics
  * An audit log summarizing choices/caveats

Chunking and IDs

* Deterministic path‑based IDs:
  `docId=refund-policy_2023 | sec=5.1 | para=3 | page=12 | table=T2`
  Encode as `refund-policy_2023__s5.1__p3__pg12` and store also as discrete fields to enable SQL joins.
* For tables: `...__tbl_T2__row_17__col_charge_code`
* Keep a SHA‑256 fingerprint of each chunk’s raw text to detect drift.

Provenance embedding

* Embed inline HTML comments in Markdown chunks with filename, version, page, section path, chunk ID, bbox if available, and extracted\_on timestamp.
* Mirror the same metadata in the JSON record for surveilr.

## Docling extraction patterns

Headings and reading order
Docling provides advanced PDF understanding including page layout and reading order. Use it to reconstruct the logical flow rather than relying on raw text order. ([Docling][1])

Tables
Prefer docling’s structured table outputs; the project ships models such as TableFormer and layout models to detect tables and cells. Persist both a rendered Markdown table and the normalized cell grid in JSON so you can query by header/row keys later. ([GitHub][7], [IBM Research][6])

Figures, captions, formulas, code
Use docling’s ability to classify and export these elements; capture caption text and link figure references to their anchors for precise citations. ([Docling][1])

OCR and scanned content
Enable OCR in docling when PDFs are image‑based to produce text + layout, preserving anchor points for citations. ([Medium][5])

Exports
Always write both Markdown and one of docling’s lossless exports (JSON or DocTags). The JSON is your “truth source” for machine reasoning; Markdown is for human audits and quick LLM grounding. ([Docling][1])

## Quality gates and acceptance checks

Automated checks

* Structure parity: expected vs. actual counts of h1/h2, lists, tables, figures.
* Reading order sanity: headings never decrease by >1 level between chunks.
* Table schema: required headers present; row counts within tolerance.
* Provenance completeness: 100% of chunks include page/sheet and section path.
* Drift detection: re‑run yields identical chunk\_id and content\_hash sets.

Human review

* 10–15 minute skim checklist: headings, first/last page, two random tables, and two cross‑references.
* Sign‑off by Trust Engineer before ingestion.

## Orchestration and runtime

Language choice

* Docling is Python‑native with both CLI and API; keep Python for the parser and use `surveilr` capturable executables or Deno TypeScript for orchestration, manifests, repo hygiene, and CI. ([Docling][1], [IBM Research][2])

Execution modes

* Local laptop runs (docling is designed to run efficiently).
* Containerize for CI; pin model versions for reproducibility. ([IBM Research][2])

Interfaces

* Preferred: call the docling CLI or a thin Python entrypoint from Deno’s Command API; capture stdout JSON and files.
* Alternative: run a small HTTP microservice wrapping docling for team access.
* Both modes should emit a manifest.json and exit non‑zero on QC failure.

## Document‑class playbooks

Policies/SOPs (PDF/DOCX)

* Parse with docling; preserve section hierarchy; add semantic labels for “Purpose”, “Scope”, “Definitions”, “Responsibilities”, “Procedures”, “Exceptions”.
* Extract numbered lists as step structures with stable step IDs.

Contracts/terms

* Treat definitions, obligations, rights, SLAs, penalties as first‑class fields.
* Capture clause numbers and cross‑reference targets; emit citation anchors for each clause’s canonical name.

PowerPoint decks

* Extract slide titles, bullet hierarchies, and speaker notes distinctly; treat note paragraphs as authoritative narrative text.
* Link each bullet to slide index and position for precise references.

Excel workbooks

* For each sheet: extract header rows, normalize merged cells, and map to a tidy table.
* Emit typed columns (currency, percent, date) and units; preserve sheet name and region used.

Scanned forms / mixed‑quality PDFs

* Enable OCR and heuristics to rebuild structure; capture any form fields as key‑value pairs with page coordinates for future auto‑labeling. ([Medium][5])

## Governance and roles (mapped to your manifesto)

* Expectations Engineers define the knowledge contract (what must be answerable, with examples of expected citations).
* Source Document Preparation Engineers implement the doc‑specific script and QC checks.
* Trust Engineers validate structure/provenance and sign off.
* Integration Engineers wire up surveilr ingestion and build SQL views.

## Example operating procedure (SOP PDF)

1. Triage: confirm sections that must be reliably cited (e.g., 3.2 “Escalation”).
2. Author expectations.yml listing mandatory sections, tables, and citations.
3. Implement transform.py using docling to export Markdown + docling JSON; add enrichment to label “definitions”, “timelines”, “exceptions”. ([Docling][1])
4. Run orchestrate.ts to produce out/*.md, out/*.json, manifest, audit‑log.md.
5. Automated checks; then human review; sign‑off.
6. Ingest into surveilr with chunk IDs and frontmatter; publish SQL views.
7. Record the script and manifest in git for provenance.

## Retrieval and citation in downstream AI

* Guardrails: RAG must only surface content with valid chunk\_id and provenance.
* Citations show doc\_id, section path, and page; links jump to Markdown anchors; optional “view source” opens the exact chunk with bbox from docling JSON.
* LLM policies: forbid answers that cite un‑anchored text; include table cell coordinates for table‑derived claims.

## Pilot candidates and sequencing

* A complex policy manual PDF with mixed tables and callouts (docling pilot 1). ([Docling][1])
* An Excel pricing workbook with 4–6 sheets and merged headers (pilot 2). ([Docling][1])
* A PPTX deck with heavy speaker notes (pilot 3). ([Docling][1])

## Risks and mitigations

* Parsing variance across versions → pin docling and model versions; store manifests. ([GitHub][7])
* OCR errors on poor scans → enforce human review and spot checks. ([Medium][5])
* Table misalignment → use the table structure model and add post‑processors; include row/column header validation in QC. ([GitHub][7])
* Team skill split (TS vs. Python) → keep Python minimal, expose one CLI, orchestrate in Deno.

[1]: https://docling-project.github.io/docling/?utm_source=chatgpt.com "Docling - GitHub Pages"
[2]: https://research.ibm.com/blog/docling-generative-AI?utm_source=chatgpt.com "IBM is open-sourcing a new toolkit for document conversion"
[3]: https://github.com/microsoft/markitdown?utm_source=chatgpt.com "microsoft/markitdown: Python tool for converting files ..."
[4]: https://devblogs.microsoft.com/blog/10-microsoft-mcp-servers-to-accelerate-your-development-workflow?utm_source=chatgpt.com "10 Microsoft MCP Servers to Accelerate Your Development ..."
[5]: https://medium.com/%40onur.sakar1997/document-processing-and-query-automation-with-ibm-docling-converting-to-markdown-and-json-with-4f318257669a?utm_source=chatgpt.com "Document Processing and Query Automation with IBM ..."
[6]: https://research.ibm.com/publications/docling-an-efficient-open-source-toolkit-for-ai-driven-document-conversion?utm_source=chatgpt.com "Docling: An Efficient Open-Source Toolkit for AI-driven ..."
[7]: https://github.com/docling-project/docling-ibm-models?utm_source=chatgpt.com "docling-project/docling-ibm-models"
[8]: https://www.ibm.com/new/announcements/ibm-adds-open-source-projects-docling-beeaI-and-data-prep-kit-added-to-the-linux-foundation?utm_source=chatgpt.com "IBM contributes key open-source projects to Linux ..."
