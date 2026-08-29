Title: Cutting RAG Costs: Do We Really Need to Send the Whole PDF to the LLM?
Date: 2026-08-29
Category: AI
Tags: rag, llm, cost-optimization, retrieval, chunking, embeddings, ai-engineering
Slug: cutting-rag-costs-do-we-really-need-to-send-the-whole-pdf

## The Problem Nobody Talks About Loudly

Retrieval-Augmented Generation — RAG — has quietly become the default way to make LLMs actually useful with real-world documents. You index a PDF, embed the chunks, retrieve the relevant ones at query time, and pass them to the model. It works. People ship it.

But there's a quiet inefficiency baked into nearly every RAG pipeline in production today.

When a 100-page PDF gets chunked and embedded, a huge share of those chunks are junk — or at least, near-junk. Table of contents pages. Repeated legal disclaimers. Boilerplate headers. Background paragraphs that exist to fill space, not convey information. And yet the pipeline treats every single chunk as equally worthy of being embedded, stored, retrieved, and eventually sent to the LLM as context.

Every token that reaches the LLM costs money. It also costs latency. If 30–40% of a typical document is low-value content, you're paying full price for tokens that contribute almost nothing to the quality of the answer. That's not a small rounding error — it's a structural inefficiency that compounds at scale.

## The Core Idea: Split Before You Index

The proposed fix is deceptively simple: instead of treating a document as a flat bag of chunks, split it into two buckets before it ever hits your vector store.

**Bucket A — high-importance content:** dense, factual material. The paragraphs that actually answer likely questions. The parts of the document that carry the semantic core — definitions, results, key claims, specific data.

**Bucket B — low-importance content:** repetitive, generic, or peripheral material. The stuff that exists in the document but contributes little marginal value to most queries. Disclaimers, transition paragraphs, boilerplate, tangential background sections.

Under normal operation, both buckets are available. But when cost becomes a constraint — high query volume, tight budgets, latency-sensitive applications — the system can skip Bucket B entirely and answer using only Bucket A.

The bet is that for the majority of real-world queries, this produces a negligible drop in answer quality while delivering a real reduction in token spend.

## Why This Isn't Just "Better Chunking"

You might be thinking: isn't this just more aggressive chunking or a higher similarity threshold at retrieval time? Not quite — and the distinction matters.

RAG pipelines already do a form of triage. Retrieval itself filters: you ask for the top-k chunks, and only those reach the model. But that triage happens at query time, based on semantic similarity to a specific question. It's dynamic, and it can miss things or over-retrieve depending on embedding quality.

The A/B split proposed here pushes triage earlier and makes it **explicit and controllable**. The document is pre-scored for "importance" at ingestion time — before any query exists. This turns cost-saving into a tunable knob rather than an emergent side effect of retrieval parameters. You can reason about it, audit it, and flip it on or off independently of your retrieval logic.

It also opens up something genuinely useful: **graceful degradation**. A system could serve full-fidelity answers using A+B when budget allows, and automatically fall back to A-only mode under load or budget pressure — without needing separate models, separate pipelines, or manual intervention. That's the kind of behavior production systems actually need.

## The Hard Questions

This idea is clean in theory. In practice, a few things need to hold up before it's publishable or deployable.

**How do you score importance?**

This is the hardest part. Options range from cheap heuristics — sentence length, keyword density, position in document, presence of named entities — to LLM-based scoring, where a small cheap model rates each chunk's informativeness before it gets indexed. You could also train a classifier on query-relevance data if you have enough signal. Each approach has a different cost profile and accuracy curve, and the right choice probably depends on your document type and query distribution.

**What's the right split ratio?**

A fixed 70/30 split across all documents would be a mistake. A legal contract has a very different "important content" distribution than a research paper or a product manual. The split ratio almost certainly needs to adapt per document type — and possibly per document. That's a tuning challenge, but it's a tractable one.

**How much quality is actually lost?**

This is the crux. You need to benchmark answer accuracy, faithfulness, and completeness on A-only retrieval versus full-document retrieval across a range of QA and summarization tasks. If the quality drop on A-only is consistently small — say, within a few percentage points on standard benchmarks — the cost savings are real and the tradeoff is defensible. If it falls apart on certain query types, you need to know which ones and why.

**Does the optimal split generalize?**

Or does every document type need its own scoring function and ratio? If the answer is "every domain needs separate tuning," the approach is still useful but narrower — more of a framework than a plug-and-play technique. That's still worth publishing, but it changes the framing.

## Why It's Worth Paying Attention To

Most RAG efficiency research focuses on the retrieval side: better embedding models, smarter chunk sizing, approximate nearest neighbor algorithms, reranking. Fewer papers tackle content-level triage as a first-class cost lever.

The implicit assumption in most pipelines is: if it's in the document, it deserves to be in the index. The A/B split challenges that assumption directly — and frames it as a design decision you should be making explicitly, not something you accidentally get right by tuning chunk overlap and top-k.

If the approach can be shown to preserve most answer quality while cutting a meaningful share of token costs, it's a practical, adoptable technique. Not just a theoretical contribution — something engineers can actually reach for when their RAG bill shows up at the end of the month.
