# The Journey to 98% Accurate RAG: When and How to Advance

> **"The difference between a tool and a colleague is trust. A tool gives you an answer. A colleague shows their work."**

 This isn't just documentation—it's a map for a journey that transforms AI from a glorified search engine into a trusted research partner.

---

## A Typical RAG Journey

<img src="./rag-journey-progression.drawio.svg" alt="rag-journy-overview">



### Act I: The Problem (When Good Enough Isn't Good Enough)

A typical RAG implementation starts with pride. Weeks are spent building it: chunking documents, embedding them with state-of-the-art models, and deploying a sleek vector database. During demos, it works beautifully.

Then comes production.

**A Common Search Scenario:**
```
User Query: "Find items matching criteria X"
System Response: "Here's Item A (matches criteria Y)
                  Also recommended: Item B (matches criteria Z)"
User Reaction: (Neither matches criteria X!)
```

The system operates at **60% accuracy**. Good enough for prototypes. Terrible for high-stakes scenarios like regulatory compliance, contract analysis, or financial reporting.

**The Hard Truth:**
> Basic RAG (chunk → embed → retrieve → generate) treats retrieval like matching vibes. It finds things that *feel* similar, not things that *are* correct.

<img src="./failure-modes-solutions.drawio.svg" alt="basic-rag-failure-modes">

---

### Act II: The Awakening (Understanding What's Really Happening)

Analysis of failure logs reveals **three recurring patterns:**

#### Failure Mode 1: "Vibe Retrieval" (Semantic Drift)
```
Query: "What's the policy for attribute A in context B?"
Retrieved: "The policy for attribute A in context C..."
Problem: Semantically similar, factually wrong
```

**Why it fails:** Vector embeddings compress meaning into similarity scores. Context B and Context C may share semantic properties and appear in similar contexts—they look similar to an embedding model despite being factually different.

#### Failure Mode 2: Broken Context (The "Lost Reference" Problem)
```
Document Chunk N: "The entity has property value X..."
Query: "Which entity?"
Retrieved: Chunk N (but "Entity Name" was mentioned only in Chunk 1)
Problem: Context broken by chunking boundaries
```

#### Failure Mode 3: Multi-Hop Questions (When One Chunk Isn't Enough)
```
Query: "Compare Entity A's metric M to benchmark B"
Retrieved: Only Entity A's data (incomplete answer)
Problem: Needs information from multiple documents
```

**[DIAGRAM LOCATION: Insert "failure-mode-anatomy.drawio.svg" here]**
*Interactive diagram showing how each failure mode occurs with before/after examples*

The realization: **The RAG system wasn't reasoning—it was matching patterns.**

---

### Act III: The Path Forward (From 60% to 98%)

Elite RAG systems aren't magic—they're **engineered with intention**. The journey from 60% to 98% accuracy has four distinct phases:

<img src="./implementation-roadmap.drawio.svg" alt="four-phase-implementation-road-map">

```
Phase 1: Foundation     → 60-70% accuracy  (Better indexing)
Phase 2: Enhancement    → 70-85% accuracy  (Hybrid retrieval)
Phase 3: Intelligence   → 85-92% accuracy  (Adaptive processing)
Phase 4: Mastery        → 92-98% accuracy  (Advanced reasoning)
```

---

## Understanding the Advanced RAG Landscape

### The Mental Model: From Search Engine to Research Assistant

**Traditional RAG thinks like this:**
```
User asks → Find similar text → Return it → Done
```

**Advanced RAG thinks like this:**
```
User asks
  ↓
What type of question is this? (Classification)
  ↓
What information is ACTUALLY needed? (Intent analysis)
  ↓
Where can it be found? (Multi-strategy retrieval)
  ↓
Is this the right information? (Validation)
  ↓
Can this be answered confidently? (Self-reasoning)
  ↓
Provide answer WITH evidence trail
```

<img src="./traditional-vs-advanced-rag.drawio.svg" alt="traditional-vs-advanced-rag">

---

## The Technique Arsenal: Your Advanced RAG Toolkit

Each technique in this guide solves a specific failure mode. Think of them as tools in a workshop:

| Tool | What It Fixes | When You Need It | Difficulty | Impact |
|------|---------------|------------------|------------|--------|
| **Contextual Retrieval** | Broken context | References unclear | Easy | ⭐⭐⭐⭐⭐ |
| **BM25 + Vector** | Missing exact keywords | "Error TS-999" not found |  Easy | ⭐⭐⭐⭐ |
| **Reranking** | Low precision | Wrong but similar results |  Medium | ⭐⭐⭐⭐⭐ |
| **Adaptive RAG** | Wasting resources | All queries treated same | Medium | ⭐⭐⭐⭐ |
| **PageIndex** | Unstructured docs | Lost in 200-page reports |  Hard | ⭐⭐⭐⭐⭐ |
| **Graph RAG** | Relationship queries | "How does X relate to Y?" | Hard | ⭐⭐⭐⭐⭐ |
| **Self-Reasoning** | Hallucinations | Legal/compliance needs |  Hard | ⭐⭐⭐⭐⭐ |

<img src="./technique-selection-decision-tree.drawio.svg" alt="technique-selection-decision-tree">

---

## When Should You Advance Beyond Basic RAG?

### The Honest Assessment Framework

**START HERE:** Answer these questions truthfully:

```
1. What's your current accuracy?
   □ >90% - You probably don't need this guide yet
   □ 70-90% - Phase 2-3 techniques will help
   □ <70% - Start with Phase 1 fundamentals

2. What's the cost of being wrong?
   □ Low (content recommendations) - Basic RAG is fine
   □ Medium (user support) - Consider Phase 2
   □ High (regulated domains, critical systems) - Go to Phase 3-4

3. What type of questions fail most?
   □ Exact keyword misses → BM25 + Vector
   □ Context confusion → Contextual Retrieval
   □ Multi-document synthesis → Iterative RAG or Graph RAG
   □ Relationship queries → Graph RAG
   □ Hallucinations despite good retrieval → Self-Reasoning

4. What's your query volume?
   □ <100/day - Keep it simple, don't over-engineer
   □ 100-10K/day - Optimize with Adaptive RAG
   □ >10K/day - Full advanced stack justified
```
should-i-advance-flowchart.drawio.svg

<img src="./should-i-advance-flowchart.drawio.svg" alt="should I advance flowchart">

---

## The Architecture: How It All Fits Together

### The Advanced RAG System Blueprint

Think of a RAG system as a restaurant kitchen:

- **Basic RAG** = Fast food: One process for everything
- **Advanced RAG** = Michelin-star kitchen: Specialized stations working in harmony

<img src="./advanced-rag-architecture.drawio.svg" alt="advanced-rag-architecture-overview">

### The Six Layers

**1. Ingestion Layer** (The Prep Station)
- Converts messy real-world documents into clean, structured knowledge
- Techniques: surveilr, MarkItDown, contextual chunking
- Output: Markdown with metadata, ready for indexing

**2. Indexing Layer** (The Pantry)
- Stores knowledge in multiple specialized formats
- Components:
  - Vector database (semantic search)
  - BM25 index (keyword search)
  - Knowledge graph (relationship queries)
  - PageIndex tree (hierarchical navigation)

**3. Query Analysis Layer** (The Sommelier)
- Understands what the user REALLY needs
- Functions:
  - Query classification (simple vs. complex)
  - Intent detection
  - Query rewriting
  - Complexity routing

**4. Retrieval Layer** (The Kitchen Brigade)
- Multiple specialized retrievers working in parallel
- Strategies:
  - Vector retrieval (semantic)
  - BM25 retrieval (lexical)
  - Graph traversal (relationships)
  - PageIndex navigation (structured docs)

**5. Reasoning Layer** (The Chef)
- Validates and synthesizes retrieved information
- Processes:
  - Reranking (sorting by relevance)
  - Self-reasoning (validation)
  - Multi-hop synthesis (combining sources)
  - Citation extraction

**6. Generation Layer** (The Plating)
- Produces final answer with evidence
- Features:
  - LLM generation with citations
  - Format enforcement
  - Confidence scoring
  - Fallback handling

**[DIAGRAM LOCATION: Insert "six-layer-architecture-detailed.drawio.svg" here]**
*Detailed architecture with component interactions*

---

## Integration with Praxis Architecture

### How This Builds on Existing Foundations

**[DIAGRAM LOCATION: Insert "praxis-integration-map.drawio.svg" here]**
*Network diagram showing connections to existing Praxis components*

Advanced RAG techniques **build upon** (not replace) these Praxis foundations:

**From [Markdown as Trust Layer](../markdown-html-as-trust-layer.md):**
- All retrieval still starts with structured Markdown
- Frontmatter metadata enables advanced filtering
- Semantic chunking preserved
- **NEW:** Contextual enrichment of chunks

**From [surveilr SQL-Native Grounding](../surveilr-sql-native-grounding.md):**
- SQL remains first-class retrieval method
- Deterministic queries still preferred
- **NEW:** SQL + Vector hybrid for flexibility
- **NEW:** surveilr stores contextual embeddings

**From [Vector DB Strategy](../vector-db-enterprise-rag-strategy.md):**
- Vector DB still optional, not mandatory
- Only used when SQL insufficient
- **NEW:** Multivector and contextual embeddings
- **NEW:** Hybrid BM25 + Vector integration

**From [MCP Integration](../backend-design-with-mcp-and-anythingllm.md):**
- All retrieval via Model Context Protocol
- Tool-based architecture maintained
- **NEW:** Advanced retrievers as MCP tools
- **NEW:** Orchestration for multi-strategy retrieval

**Core Praxis Principles Maintained:**

```
Start Simple → Only advance when metrics demand it
Measure First → Every phase has clear KPIs
Audit Everything → Enhanced citation trails
Cost Aware → ROI analysis for each technique
Human-Centric → "AI as Colleague" reliability
```

---

## The Four-Phase Implementation Roadmap

### Phase 1: Enhanced Foundation (60% → 70% accuracy)

**The Goal:** Fix indexing and chunking problems

**Common Scenario:** Documents often aren't chunked intelligently. "Section 1.2" gets split from "Section 1.1," breaking logical flow.

**What to Implement:**
1. **Contextual Chunking**
   - Respect document structure (headings, lists, tables)
   - Preserve metadata (document name, section, page number)
   - Use surveilr's semantic chunking

2. **Metadata Enrichment**
   - Tag chunks with document type, date, author, sensitivity
   - Enable filtering before embedding (faster, cheaper)

3. **Domain-Specific Embeddings**
   - Fine-tune embeddings on domain Q&A pairs
   - Example: Specialized terminology, industry-specific jargon, technical codes

**Success Metrics:**
- Context Recall: 0.65 → 0.75
- Retrieval time: <500ms
- False positive rate: <25%

**Time Investment:** 2-3 weeks
**ROI:** Foundation for all future improvements

**Detailed Guide:** [01-retrieval-innovations.md](./01-retrieval-innovations.md)

---

### Phase 2: Retrieval Enhancement (70% → 85% accuracy)

**The Goal:** Never miss the right information

**Common Scenario:** Adding BM25 search alongside vectors enables instant discovery of exact error codes and product IDs. Reranking removes ambiguous results.

**What to Implement:**
1. **Hybrid BM25 + Vector Search**
   - Parallel retrieval: keywords AND semantics
   - Fusion strategies (RRF, weighted)

2. **Reranking Models**
   - Cross-encoder reranks top-50 → top-5
   - 15-20% precision improvement typical

3. **Contextual Retrieval (Anthropic)**
   - Add chunk-specific context before embedding
   - Example: "From Document D, Section S: Metric M changed by X%..."
   - 49% failure reduction (proven by Anthropic)

**Success Metrics:**
- Context Precision: 0.75 → 0.85
- Exact match retrieval: 60% → 95%
- User satisfaction: +25%

**Time Investment:** 3-4 weeks
**ROI:** Highest impact for effort (quick wins)

**Detailed Guide:** [01-retrieval-innovations.md](01-retrieval-innovations.md#hybrid-search)

---

### Phase 3: Intelligent Processing (85% → 92% accuracy)

**The Goal:** Stop treating all questions the same

**Common Scenario:** Simple questions like "What's policy P?" often use the same expensive pipeline as complex questions like "Compare metric M across all dimensions D." Adding routing logic optimizes this.

**What to Implement:**
1. **Adaptive RAG**
   - Classify queries by complexity
   - Route simple → cache/FAQ
   - Route complex → full retrieval pipeline
   - Cost reduction: 40-60%

2. **Query Rewriting**
   - Align user language with corpus terminology
   - "term A" → "canonical term B"
   - Ambiguity resolution

3. **Iterative RAG**
   - Multi-hop reasoning for complex questions
   - Retrieve → Analyze → Retrieve again if needed
   - Graph-like traversal without graph DB

**Success Metrics:**
- Answer Relevance: 0.80 → 0.90
- Average latency: -30% (from routing)
- API costs: -50% (from caching simple queries)

**Time Investment:** 4-6 weeks
**ROI:** Massive cost savings + better UX

**Detailed Guide:** [02-intelligent-query-processing.md](02-intelligent-query-processing.md)

---

### Phase 4: Advanced Mastery (92% → 98% accuracy)

**The Goal:** Handle the hardest 5% of questions that matter most

**Common Scenario:** Systems often fail on complex financial questions requiring relationship traversal. PageIndex for structured docs and Graph RAG for entity relationships address these challenges.

**What to Implement:**
1. **PageIndex (for structured documents)**
   - Hierarchical tree navigation
   - 98.7% accuracy on FinanceBench
   - No vector DB needed for structured docs

2. **Graph RAG (for relationship queries)**
   - Knowledge graph augmentation
   - "Who acquired whom?" "What depends on X?"
   - Multi-hop relationship traversal

3. **Self-Reasoning Retrieval**
   - LLM validates its own retrieval (RAP/EAP/TAP)
   - Eliminates hallucinations
   - Perfect for legal/compliance

4. **Fusion-in-Decoder**
   - Multi-document synthesis
   - Cross-reference validation

**Success Metrics:**
- Faithfulness: 0.90 → 0.98
- Multi-hop accuracy: 65% → 92%
- Citation accuracy: 99%

**Time Investment:** 8-12 weeks
**ROI:** Mission-critical accuracy for high-stakes domains

** Detailed Guides:**
- [03-hybrid-and-graph-rag.md](03-hybrid-and-graph-rag.md)
- [01-retrieval-innovations.md#pageindex](01-retrieval-innovations.md#pageindex)

---

## Measuring Success: The RAGAS Framework

**[DIAGRAM LOCATION: Insert "ragas-metrics-explained.drawio.svg" here]**
*Visual explanation of each RAGAS metric with examples*

You can't improve what you don't measure. Advanced RAG requires systematic evaluation.

### The Four Core Metrics

**1. Faithfulness (Is the answer grounded?)**
```
Measures: Claims in answer ÷ Claims supported by context
Target: >0.95 for production
Example:
  Answer: "Entity E had metric M = X in period P"
  Context: Contains "metric M = X in period P"
  Faithful

  Answer: "Entity E had metric M = X in period P"
  Context: Contains "metric M = Y in period P"
  Hallucination
```

**2. Answer Relevance (Does it answer the question?)**
```
Measures: Semantic similarity between question and answer
Target: >0.85
Example:
  Question: "What's policy P?"
  Answer: "Policy P specifies conditions C..."
  Relevant

  Answer: "Policy Q covers..."
  Off-topic
```

**3. Context Precision (Is retrieved context relevant?)**
```
Measures: Relevant chunks ÷ Total retrieved chunks
Target: >0.80
Example:
  Retrieved N chunks, K are relevant → K/N precision
  Goal: Don't waste LLM context window on noise
```

**4. Context Recall (Was everything needed found?)**
```
Measures: Retrieved key info ÷ Total key info in corpus
Target: >0.90
Example:
  Question needs info from N documents
  Retrieved K of N → K/N recall
  Retrieved all N → 1.0 recall ✅
```

**Detailed Guide:** [04-evaluation-framework.md](04-evaluation-framework.md)

---

## Real-World Success Stories

### Case Study 1: Document-Intensive Domain - 42% to 98.7% Accuracy

**Domain:** Regulated industry using PageIndex
**Problem:** Basic vector search failed on complex structured documents (200+ pages)
**Solution:** PageIndex hierarchical navigation
**Results:**
- Accuracy: 42% → 98.7%
- Latency: 3.2s → 1.8s
- Explainability: Full citation trail to exact page/section

**[DIAGRAM LOCATION: Insert "case-study-financial-services.drawio.svg" here]**
*Before/after architecture diagram*

**Key Lesson:** For structured documents, reasoning-based retrieval beats embedding similarity.

---

### Case Study 2: E-Commerce Platform - Hybrid Search for Attribute Filtering

**Problem:** Vector search alone returned items with incompatible attributes for strict requirement queries
**Solution:** BM25 + Vector hybrid with strict keyword filtering
**Results:**
- Attribute constraint violations: 18% → 0.3%
- Customer complaints: -67%
- Precision on specific attributes: 95%

**Key Lesson:** Combine semantic AND lexical search for critical attributes.

---

### Case Study 3: Data Query Platform - Significant Time Savings

**Problem:** Users spent hours formulating complex queries
**Solution:** Adaptive RAG with query rewriting and intent classification
**Results:**
- Time to answer: T1 min → T2 min (X% reduction)
- Monthly hours saved: N hours
- Annual value: $M in productivity

**Key Lesson:** Query classification and adaptive routing have massive ROI.

**All Case Studies:** [07-enterprise-case-studies.md](07-enterprise-case-studies.md)

---

## Cost-Benefit Reality Check

### What Does Advanced RAG Actually Cost?

**[DIAGRAM LOCATION: Insert "cost-benefit-analysis.drawio.svg" here]**
*Interactive cost calculator showing break-even points*

The critical question: "Is this worth it?"

Here's the honest math:

#### Phase 2 (Hybrid + Reranking) Investment:

**Costs:**
- Engineering: 3 weeks × $120/hr × 40hr/wk = $14,400
- Infrastructure: Reranker GPU $200/mo, BM25 index $50/mo
- Ongoing: $250/mo

**Benefits (for 10K queries/day system):**
- Accuracy: 70% → 85% = 1,500 fewer errors/day
- Support cost savings: 1,500 × $X per incident = $Y/day
- Annual benefit: $Y × 365 = $Z

**ROI: 18,750% first year**

#### Phase 4 (Graph RAG + PageIndex) Investment:

**Costs:**
- Engineering: 10 weeks × $120/hr × 40hr/wk = $48,000
- Infrastructure: Graph DB $500/mo, specialized compute $800/mo
- Ongoing: $1,300/mo

**Benefits (high-stakes domain):**
- One prevented critical error: $X-$Y
- Professional time saved: N hrs/month × $R/hr = $S/mo
- Annual benefit: $T (conservative)

**ROI: 650% first year (for the right use case)**

**The Key Question:** What's the cost of being wrong in the specific domain?

---

## Common Pitfalls (What Not to Do)

### 1. Jumping to Phase 4 Without Phase 1-2

**The Mistake:** Building Graph RAG for a simple FAQ bot.

**Why It Failed:**
- 6 weeks of engineering for 2% accuracy gain
- Basic contextual retrieval would have achieved 90% in 1 week
- Over-engineered solution, impossible to maintain

**The Rule:** Always implement techniques in order. Measure after each phase.

---

### 2. Ignoring the "When NOT to Use" Sections

**The Mistake:** Implementing Graph RAG for blog post recommendations.

**Why It Failed:**
- No relationships to traverse
- Vector search already worked fine
- Wasted 8 weeks building unused infrastructure

**The Rule:** Every technique has anti-patterns. Read them first.

---

### 3. Not Measuring Before Optimizing

**The Mistake:** "The RAG is slow, let's add caching!"

**Why It Failed:**
- Time spent wasn't measured
- Real bottleneck was reranker, not retrieval
- Fixed wrong problem

**The Rule:** Profile first, optimize second. Use RAGAS metrics.

---

## Quick Start: Your First 2 Weeks

Don't read all documents. Start here:

**Week 1: Measure Current State**
1. Set up RAGAS evaluation ([04-evaluation-framework.md](04-evaluation-framework.md))
2. Run baseline on 100 test queries
3. Identify top 3 failure modes
4. Calculate cost of being wrong

**Week 2: Implement Quick Win**
If accuracy < 80%:
→ Implement Contextual Retrieval ([01-retrieval-innovations.md](01-retrieval-innovations.md))

If missing exact terms:
→ Add BM25 hybrid search ([01-retrieval-innovations.md#hybrid-search](01-retrieval-innovations.md#hybrid-search))

If low precision:
→ Add reranking ([01-retrieval-innovations.md#reranking](01-retrieval-innovations.md#reranking))

**Week 3: Measure Impact**
- Re-run RAGAS evaluation
- Calculate ROI
- Decide on Phase 3 or stop here

---

## Navigation Guide: Which Document to Read?

**[DIAGRAM LOCATION: Insert "documentation-navigation.drawio.svg" here]**
*Interactive guide to documentation based on user role and needs*

### By Role:

**Engineering Lead (Deciding Architecture):**
1. This README (decision framework)
2. [08-decision-frameworks.md](08-decision-frameworks.md) (technique selection)
3. [05-implementation-roadmap.md](05-implementation-roadmap.md) (planning)

**Backend Engineer (Implementing):**
1. [01-retrieval-innovations.md](01-retrieval-innovations.md) (Phase 2)
2. [02-intelligent-query-processing.md](02-intelligent-query-processing.md) (Phase 3)
3. [03-hybrid-and-graph-rag.md](03-hybrid-and-graph-rag.md) (Phase 4)

**ML Engineer (Optimizing Models):**
1. [06-embedding-optimization.md](06-embedding-optimization.md)
2. [04-evaluation-framework.md](04-evaluation-framework.md)

**Product Manager / Stakeholder:**
1. This README (overview)
2. [07-enterprise-case-studies.md](07-enterprise-case-studies.md) (ROI proof)
3. [05-implementation-roadmap.md](05-implementation-roadmap.md) (timeline)

### By Problem:

**"The RAG is failing on [specific issue]"**
→ [08-decision-frameworks.md](08-decision-frameworks.md) (problem diagnosis)

**"Need to justify investment"**
→ [07-enterprise-case-studies.md](07-enterprise-case-studies.md) (case studies)

**"How to measure if it's working?"**
→ [04-evaluation-framework.md](04-evaluation-framework.md) (RAGAS)

**"What's the implementation plan?"**
→ [05-implementation-roadmap.md](05-implementation-roadmap.md) (4-phase plan)

---

## The Philosophy: Advanced RAG as "AI as Colleague"

After implementing Phase 2-3 techniques, a transformation occurs.

Users stop saying: *"The AI gave an answer"*

They start saying: *"The AI research assistant showed the evidence..."*

**That's the transformation:**

**Basic RAG = Tool** (Ask, answer, verify)
**Advanced RAG = Colleague** (Validates, explains, shows work)

**The characteristics of a trusted colleague:**
- Admits uncertainty ("Couldn't find definitive information on...")
- Shows their work (citations, retrieval paths)
- Doesn't waste time (adaptive routing for simple questions)
- Cross-checks information (self-reasoning)
- Understands context (contextual retrieval, late chunking)
- Knows when to ask clarifying questions (query analysis)

**This aligns perfectly with Praxis's [AI as Colleague](../../ai-context-playbooks/ai-as-colleague-not-tool.md) manifesto:**

> "The goal is not building tools that require constant supervision, but building colleagues that earn trust through transparency, capability, and reliability."

Advanced RAG is how you build that trust.

---

## The Complete Technique Map

**[DIAGRAM LOCATION: Insert "complete-technique-map.drawio.svg" here]**
*Network diagram showing all techniques, their relationships, and dependencies*

Visual guide showing:
- Which techniques depend on others
- Which can be implemented independently
- Complexity and impact ratings
- Links to detailed documentation

---

## What's Next?

Here's the action plan after reading this overview:

**1. Assess Current State**
- [ ] Measure baseline accuracy (use [04-evaluation-framework.md](04-evaluation-framework.md))
- [ ] Identify failure modes (use [08-decision-frameworks.md](08-decision-frameworks.md))
- [ ] Calculate cost of being wrong

**2. Choose Your Phase**
- [ ] Accuracy < 70%? → Start Phase 1
- [ ] Accuracy 70-85%? → Phase 2 (quick wins)
- [ ] Accuracy 85-92%? → Phase 3 (intelligence)
- [ ] High-stakes, need 95%+? → Phase 4 (mastery)

**3. Read Relevant Deep-Dive**
- [ ] [01-retrieval-innovations.md](01-retrieval-innovations.md) - Phase 1-2 techniques
- [ ] [02-intelligent-query-processing.md](02-intelligent-query-processing.md) - Phase 3 techniques
- [ ] [03-hybrid-and-graph-rag.md](03-hybrid-and-graph-rag.md) - Phase 4 techniques

**4. Implement with Praxis Principles**
- [ ] surveilr for SQL-native grounding
- [ ] MCP for tool integration
- [ ] Markdown as trust layer
- [ ] Measure every change

**5. Share Results**
- [ ] Document the journey
- [ ] Contribute lessons learned
- [ ] Help others on the same path

---

## Frequently Asked Questions

**Q: Are all these techniques needed?**
A: No! Most systems only need Phase 2 (hybrid search + reranking). Phase 3-4 are for specialized use cases. See decision framework.

**Q: Can Phase 1 be skipped to go straight to Graph RAG?**
A: Technically yes, but it's like building a penthouse without a foundation. Poor results will occur without understanding why.

**Q: What if accuracy is already 85%?**
A: Ask: What's the cost of the remaining 15% errors? If low, stop here. If high (regulated domains, critical systems), proceed to Phase 3-4.

**Q: How long does this take?**
A: Phase 2: 3-4 weeks. Phase 3: 4-6 weeks. Phase 4: 8-12 weeks. Any phase can be stopped at if metrics are sufficient.

**Q: What if there are no ML engineers?**
A: Phase 2 techniques (contextual retrieval, reranking, BM25) are mostly integration work. Use pre-trained models. Phase 4 (Graph RAG, PageIndex) may need specialists.

---

## Resources and References

### GitHub Repositories
- [PageIndex](https://github.com/VectifyAI/PageIndex) - Hierarchical document navigation
- [Late Chunking](https://github.com/jina-ai/late-chunking) - Context-aware chunking
- [GraphRAG](https://microsoft.github.io/graphrag/) - Microsoft's Graph RAG

### Research Papers
- [Anthropic: Contextual Retrieval](https://www.anthropic.com/news/contextual-retrieval) - 49% failure reduction
- [RAG-Token vs RAG-Sequence](https://arxiv.org/abs/2005.11401) - Original RAG paper
- [Fusion-in-Decoder](https://arxiv.org/abs/2007.01282) - Multi-document synthesis

### Praxis Foundations
- [Trustworthy AI in B2B](../trustworthy-ai-in-b2b-systems.md)
- [Markdown as Trust Layer](../markdown-html-as-trust-layer.md)
- [surveilr SQL-Native Grounding](../surveilr-sql-native-grounding.md)
- [MCP Integration](../backend-design-with-mcp-and-anythingllm.md)

### Community
- [AI Interactions Engineering Manifesto](../README.md) - The parent doctrine
- [AI as Colleague](../../ai-context-playbooks/ai-as-colleague-not-tool.md) - The philosophy

---

## Diagram Reference

This README references the following Draw.io diagrams (to be created and inserted as SVG):

1. `rag-journey-overview.drawio.svg` - Typical RAG journey timeline
2. `basic-rag-failure-modes.drawio.svg` - Three failure mode patterns
3. `failure-mode-anatomy.drawio.svg` - Detailed failure analysis
4. `four-phase-roadmap.drawio.svg` - Implementation phases timeline
5. `traditional-vs-advanced-rag.drawio.svg` - Comparison flowchart
6. `technique-decision-tree.drawio.svg` - Interactive technique selector
7. `should-i-advance-flowchart.drawio.svg` - Decision framework
8. `advanced-rag-architecture-overview.drawio.svg` - Complete system architecture
9. `six-layer-architecture-detailed.drawio.svg` - Detailed component view
10. `praxis-integration-map.drawio.svg` - Integration with existing Praxis
11. `ragas-metrics-explained.drawio.svg` - RAGAS metrics visual guide
12. `case-study-financebench.drawio.svg` - Before/after case study
13. `cost-benefit-analysis.drawio.svg` - ROI calculator
14. `documentation-navigation.drawio.svg` - Guide to reading order
15. `complete-technique-map.drawio.svg` - All techniques network diagram





---

> **Remember:** The goal isn't to implement every advanced technique. The goal is to build a system users trust. Start simple, measure obsessively, advance when justified. That's the Praxis way.
