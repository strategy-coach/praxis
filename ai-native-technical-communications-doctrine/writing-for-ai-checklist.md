---
title: "Writing for AI Checklist"
description: "Quick-reference checklist of actionable guidelines for writing documentation with AI as the primary audience."
intended-audience: ["Engineers", "Product Managers", "Technical Writers", "LLM Prompt Engineers"]
version: "1.0"
ai-context:
  prompt-grounding: |-
    - Use descriptive, query-like headings.
    - One topic per section; keep paragraphs short.
    - Be explicit: avoid pronouns and layout-dependent references.
    - Provide examples with one-line explanations.
    - Use consistent terminology and code formatting.
    - Document defaults, limits, versions, and prerequisites.
    - Add clearly labeled notes and warnings near actions.
    - Include Q&A or “How to …” modules for common tasks.
    - Tag content via frontmatter (version, audience, product).
    - Cross-link related modules; avoid conflicting duplicates.
    - Test sections by asking likely questions an AI would receive.
    - Update docs promptly when gaps are found via AI feedback.
---

# Writing for AI Checklist

Finally, here’s a quick-reference **“Writing for AI” checklist** for individual authors and editors. These are concrete tips to apply whenever you create or update technical documentation:

* **Use Descriptive Headings:** Ensure every section has a clear, specific heading. Aim for headings that could match likely search queries (e.g., “Configure TLS Encryption” instead of just “Encryption”). This helps AI chunk and retrieve relevant sections accurately.

* **One Topic per Section:** Avoid covering multiple unrelated points in one paragraph or section. If a section or paragraph starts to answer two different questions, split it. This way, each chunk the AI sees is focused on a single subject, preventing mixed answers.

* **Be Explicit and Unambiguous:** Write sentences that don’t rely on external context. Replace pronouns with the nouns they refer to, especially when it’s not crystal clear. (Instead of “this improves performance”, say “Increasing the cache size improves performance of database reads.”).

* **Include Context in Each Module:** Assume the reader (AI or human) might only read that one section. If a prerequisite or context is needed to understand it, briefly mention it or link to it. For example, start an example with a note if it requires a prior step (“After installing the CLI, you can initialize the project...”). This helps the AI not miss implicit dependencies.

* **Format for Clarity:** Use bullet points, tables, and step-by-step lists where appropriate. Structured formatting not only helps humans but also gives the AI clear separators and relationships. For instance, if listing benefits or options, use a bullet list rather than burying them in a paragraph.

* **Avoid Vague References:** Never assume the AI (or user) has access to layout or figure numbers. Replace “the table below” with “Table 1 (Default Settings)” or just integrate the info into text. If you have an important diagram, describe its key point in text. Remember, *LLMs don’t see images* – provide textual alternatives for any information conveyed visually.

* **Keep Sentences and Paragraphs Short:** Long, complex sentences are more likely to be broken incorrectly by tokenization. Shorter sentences reduce ambiguity. Similarly, break long paragraphs into smaller ones focusing on a single idea (3-5 sentences max). This not only helps AI chunking but also aids human readability.

* **Use Consistent Terminology:** Pick one term for each concept and stick to it across the docs. Don’t alternate between synonyms in different sections (e.g., use “user account” everywhere rather than switching between “user account”, “profile”, “user record” unless they mean different things that you’ve defined). Consistency improves retrieval relevance.

* **Code and Commands in Proper Markup:** Always put code, commands, or config snippets in code blocks or inline code format. This ensures they’re tokenized as entities, and the AI will treat them distinctly (and can quote them exactly). Also, provide a brief description before or after code blocks so the AI knows what it is (it might or might not include the code, but it will use your description to decide when to present it).

* **Document Assumptions and Defaults:** If a behavior depends on a default setting or an environment assumption, mention that explicitly. For example, “By default, this command uses port 443 (unless `--port` is specified).” This prevents the AI from giving advice that only applies in certain cases without stating the conditions.

* **Embed Warnings and Notes with Clear Labels:** Use a consistent approach for important notes (e.g., start with “**Note:**” or “**Warning:**” as appropriate). This way, the AI can identify that text as a note and is likely to include it or at least understand its significance. Example: “**Warning:** This action is irreversible. Use with caution.” The assistant will then hopefully echo that warning in its answer about that action.

* **Provide Examples for Every Common Task:** Wherever applicable, pair explanations with examples (and vice versa). If you describe a command, show a sample invocation and output. If you explain a concept, consider a hypothetical scenario to illustrate it. For AI, examples act as ready-made answers (the AI can simply present the example to the user who asked “How do I do X?”). Make sure to explain the example too (so the AI doesn’t just drop a code block without context).

* **Use Frontmatter to Tag Content:** At the top of each doc page, include metadata fields like version, product, category, etc., as needed. Even if the AI pipeline doesn’t use them on day one, having them means you can later filter or instruct the AI using those tags. It’s future-proofing. For instance, `version: "2.0"` could later allow an AI prompt like “User is on 1.0, avoid content with version >1.0”.

* **Avoid Overlap and Duplication (Carefully):** Try not to describe the exact same thing in two different places in different words, to prevent conflicting info if both get retrieved. It’s better to reference one central explanation. However, some duplication of critical info (like a very important note) in multiple relevant sections is okay as long as it’s identical – consistency is key. If you must update it, update all places. This way, no matter which chunk is retrieved, the message is the same.

* **Test with Sample Questions:** After writing or updating a doc section, do a mental test (or actual, if you have an internal AI tool handy): “If someone asked an AI about this, would it be able to find and understand this section?” For example, if the section is titled “Connection Retries”, will a user ask that? Or might they ask “What if the connection fails?” – maybe include that phrasing. This helps ensure the doc is aligned with user thinking.

* **Keep a User-centric Tone:** While AI is the reader, the output still goes to a human. So write in a tone and level appropriate for the target audience (user, admin, developer, etc.). If your intended audience list says both “Engineers” and “Product Managers”, you might include clarifications for less technical readers (the AI can decide to omit details if the questioner seems to need a high-level answer, but the info is there if needed). The AI will often mirror the tone of the docs in responses, so make it clear, helpful and free of internal jargon or attitude.

* **Update Docs Promptly with New Info:** Whenever you answer a question that’s not in the docs – say on a forum or support ticket – consider adding that answer to the docs in some form. Over time, this populates the knowledge base so the AI (and other users) have it. An AI-first doc strategy thrives on comprehensive coverage of real user inquiries.

By following this checklist, every piece of documentation you write will be optimized for use by AI assistants *without sacrificing human usability*. It’s a set of habits that, once developed, will become a natural part of your technical writing process.
