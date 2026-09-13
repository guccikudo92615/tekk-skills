# Retrieval methods — the catalog

Reference for the retrieval skill. Not injected into runs today — it ships
alongside the skill and becomes loadable on demand when the `load_skill`
tool lands. Until then it is the authoring source the SKILL.md
distills, and the place to grow the catalog without bloating any prompt.

## Contents
- The zeroth decision: retrieve at all?
- Chunking strategies
- Search: lexical, dense, hybrid, reranking
- Query-side techniques
- Context assembly (what reaches the window)
- Index freshness
- Measuring retrieval

## The zeroth decision: retrieve at all?

| Corpus | Right shape |
|---|---|
| Fits comfortably in the window (≤ a few hundred KB of text) | Direct read — inject it, skip the pipeline |
| Medium, well-structured, code-like | Agentic search (grep/glob + targeted reads) — the model navigates; no index to go stale |
| Large, prose-heavy, paraphrase-queried | Embedding retrieval (this catalog) |
| Large AND identifier-heavy (code, SKUs, error strings) | Hybrid mandatory — dense-only will miss exact terms |

The most common failure is a pipeline built for the corpus the team expected,
not the one they have. Measure the corpus first.

## Chunking strategies

- **Structural** (best default): split on real boundaries — headings,
  functions/classes, conversation turns. Chunks keep their meaning.
- **Fixed-size with overlap**: fallback for unstructured text; 10–20%
  overlap so boundary sentences survive. Watch for mid-sentence cuts.
- **Parent-child (small-to-big)**: index small chunks for precision, return
  their parent section for context. Strong when answers need surroundings.
- **Contextualized chunks**: prepend each chunk with a short generated
  summary of where it sits ("From the billing FAQ, section on refunds…") —
  measurably lifts retrieval on ambiguous chunks; costs one LLM run at
  index time.
- Always store and return the chunk's provenance (doc title, section,
  URL/path) — a chunk without its source can't be cited or trusted.

## Search: lexical, dense, hybrid, reranking

- **Lexical (BM25/keyword)**: exact terms, identifiers, rare tokens. Cheap,
  no staleness beyond the index itself.
- **Dense (embeddings)**: paraphrase and concept matches. Pick a current
  embedding model and record which one — mixed-model indexes silently
  degrade. Re-embed when the model changes.
- **Hybrid**: run both, merge (e.g. reciprocal rank fusion). The default
  recommendation for any real corpus — each side covers the other's misses.
- **Reranking**: a cross-encoder (or LLM scorer) reorders the top ~50–100
  candidates by actual relevance to the query. Biggest single quality lift
  on large corpora; unnecessary machinery on tiny ones.

## Query-side techniques

- **Query rewriting**: user messages are rarely good queries — rewrite
  ("expand the pronouns, name the entities") before searching,
  conversation context included.
- **Decomposition**: multi-part questions → multiple targeted queries,
  merged results.
- **HyDE** (hypothetical answer embedding): embed a model-drafted answer
  instead of the question when question-vs-document phrasing diverges.
- **Metadata filtering**: scope by source, date, tenant BEFORE similarity —
  both a relevance and an isolation concern (cross-tenant retrieval is a
  security finding, not a quality one; hand that half to the security loop).

## Context assembly (what reaches the window)

- Deduplicate near-identical chunks; cap per-source so one document can't
  monopolize the window.
- Order deliberately (most relevant nearest the question, or grouped by
  source with headers) and attribute every chunk so the model can cite.
- Drop below a relevance floor rather than always filling k slots — padding
  the window with weak matches degrades answers.

## Index freshness

- Tie indexing to the write path or a short schedule; record per-document
  index time. An index that can silently lag its corpus is a memory that
  lies — surface "index age" somewhere a human can see.

## Measuring retrieval

- A tiny golden set (10–30 real queries → the doc/chunk that SHOULD come
  back) turns every claim in this file into a number: recall@k before and
  after a change. This is the retrieval half of the evals skill — build the
  measurement with the pipeline, not after it.
