---
name: retrieval
description: 'Audits how a product gets knowledge into the model''s context — whether retrieval is the right shape at all, and whether it surfaces the right material or merely the nearest. Use when reviewing RAG pipelines, embeddings, chunking, vector stores, rerankers or query rewriting. Bundles a method catalog in references/.'
---

# Skill: retrieval

**Skill:** how this product gets knowledge into the model's context — and
whether that machinery actually surfaces the right material, or just the
nearest material. Applies only when the system reads from a corpus (docs,
tickets, code, transcripts, a knowledge base); if nothing retrieves, say so
and hand the run back to the gate.

## What to evaluate

1. **Is retrieval even the right shape?** The zeroth question. A corpus that
   fits in the window wants direct reading, not an embedding pipeline; a
   corpus queried three ways wants three strategies, not one index. The
   highest-leverage finding in this lane is often "delete the vector store,
   read the files" — or its inverse on a corpus that outgrew grep.
2. **Chunking.** Are documents split on meaning (sections, functions,
   turns) or on byte counts that cut sentences in half? Is there overlap
   where boundaries matter? Chunks that lose their heading lose their
   meaning — check what a retrieved chunk actually looks like alone.
3. **Search quality.** Pure dense similarity misses exact identifiers and
   rare terms; pure keyword misses paraphrase. Is there a hybrid, and
   reranking on top when the corpus is big enough to need it? Does the
   query that reaches the index resemble what users actually type, or does
   it need rewriting first?
4. **What reaches the window.** Top-k dumped raw, or filtered, deduplicated,
   and attributed? Retrieved-but-irrelevant context doesn't just waste
   tokens — it actively degrades answers (the model trusts what you hand
   it). Check whether anything measures that relevance.
5. **Freshness and coverage.** When the corpus changes, does the index
   follow? A stale index is a memory that lies. And can the system tell the
   difference between "no relevant material exists" and "retrieval missed
   it"?

The method catalog — chunking strategies, hybrid/reranking recipes, query
rewriting, and the direct-read decision table — lives in
`references/retrieval-methods.md`. Reason from it when comparing options;
recommend for THIS corpus's size and shape, not the fanciest pipeline.

## How to triage

Ground it statically — you cannot execute code, so evidence is a traced
path, not a live run. Take three to five representative queries from where
they already exist (logged samples, test fixtures, eval sets — and
`check_reality` for volumes) or from the product's obvious use, and trace
each BY HAND through the code: what gets rewritten, what the index is asked,
what comes back, what reaches the window — every hop cited `file:line`. A
retrieval finding without a traced query is taste; with one, it's evidence.
If the workspace has no way to know its retrieval quality at all — no logged
queries, no golden set, nothing to trace against — that absence IS the
finding: propose the measurement (a small golden set + recall@k) as the
concrete deliverable, and let the numbers drive the next run. Size the fix
to the corpus — a reranker for 200 documents is machinery without a problem.
