# RAG / Retrieval Heuristics

Decision heuristics for retrieval systems — semantic search, RAG pipelines, hybrid search, reranking, contextual enrichment. Consult during Design (problem framing) and Build/Train (pipeline decisions). Forcing questions, not rules — skip what doesn't apply.

**Peer to `ml-heuristics.md` (training/architecture) and `production-thinking.md` (post-model production).** This covers the retrieval-specific middle: how to turn a document corpus into something an agent can search well. For eval methodology (proxy-vs-decisive gate, eval-set validity), see `ml-heuristics.md` Metrics — retrieval changes are *especially* prone to those traps.

---

## First: do you even need RAG?

1. **"Are the queries conceptual or literal?"** — For codebases and literal-identifier lookups, **grep often beats RAG** (Anthropic's Claude Code finding). RAG earns its cost when queries are conceptual ("how does batch normalization work") over prose the user won't phrase verbatim. A `grep`-style exact-match tool is a cheap *complement* to semantic search, not a thing you bolt on after.
2. **"Is the corpus small enough to stuff in context?"** — If the whole relevant set fits in the model's window, retrieval is premature optimization. RAG is for when it doesn't.
3. **"Single-hop lookup or multi-hop reasoning?"** — Structured corpus + hybrid search handles single-concept lookups. Graph approaches (GraphRAG/LightRAG) target multi-hop reasoning over unstructured docs and cost another full LLM pass to build — rarely worth it for lookup-style queries. Defer until a query pattern actually demands it.

---

## Chunking

**The chunk is the most consequential design decision in the pipeline — it sets the ceiling on everything downstream.**

### Match unit vs. context unit

The unit you **embed and retrieve** and the unit you **return to the consumer** should not be the same size.

- **Child = match unit:** small and focused (~one idea, e.g. ~250 tokens). Tight chunks → sharp embeddings → precise reranking.
- **Parent = context unit:** large, returned to the agent (~a full section). Carries enough surrounding context to actually be useful once retrieved.
- Index children, link each to its parent, **dedup by parent** at return time. Precise retrieval, rich results.

### Forcing questions

- **"Am I counting tokens or guessing?"** — Word-count estimates undercount real tokens (~1.65×). Count with the **actual tokenizer** (`tiktoken cl100k_base` is within ~2% of many embedding tokenizers). Oversized children silently blow past their target and blur the match.
- **"What's the match unit, what's the context unit?"** — If you only have one chunk size, you're trading precision for context or vice versa. Split the roles.
- **"Do my chunks respect structure?"** — Split on heading boundaries (H2/H3). Keep code blocks atomic. Hard-split monster paragraphs token-bounded (re-fence code, split prose by sentence) rather than emitting a 30K-token chunk.
- **"Do answers span chunk boundaries?"** — If answers straddle two sections, add small **overlap** (copy the last 1–2 sentences into the next chunk's start, within the same parent).

---

## What goes into the embedding

The embedded text is not the same as the displayed text. Be deliberate about what the vector "sees."

- **Prepend the heading path.** Section titles carry strong semantic signal ("MVCC > Snapshot Isolation"). Prepending `heading_path` to the chunk text before embedding is a cheap, reliable win.
- **Embed the match unit, not the context unit.** Embed child text (+ heading path), not the whole parent — otherwise the vector is diluted by unrelated sentences.
- **Keep the embed-input builder in one place.** If you have a standalone remote-indexing script and a local one, the embed-text construction *must* be identical in both, or local eval won't predict production.

---

## Hybrid retrieval

Dense alone misses exact terms; sparse alone misses paraphrase. Use both.

- **Dense (vectors)** handles semantic/paraphrase matching. **Sparse (BM25)** handles exact keywords, rare identifiers, acronyms.
- **Use a real BM25** (proper tokenization, stopwords, IDF) — not a hand-rolled `re.findall` + hash tokenizer (no stemming, hash collisions).
- **Fuse server-side if your store supports it.** Distribution-based fusion (DBSF) normalizes dense and BM25 score ranges automatically in one query; it's cleaner than client-side RRF you maintain yourself.
- **Query-side expansion belongs on the caller when the caller is an LLM.** HyDE (hypothetical-answer embedding) and multi-query reformulation traditionally need a local LLM. If your consumer *is* an agent (e.g. Claude via MCP), expose `hyde_passage` / `extra_queries` as tool parameters and let it generate them — zero added latency, zero inference dependency. Keep the BM25 leg on the original query for keyword fidelity.

---

## Reranking

A cross-encoder rerank over the fused top-N is usually the highest-ROI quality lever after chunking.

- **"What's my latency budget for N candidates?"** — Pick the reranker tier that fits. Tiny 2-layer models (TinyBERT) are the weak default in some libraries; a mid-size cross-encoder (e.g. a 150M ModernBERT-class / MiniLM-L-12) is far better for ~50–100ms on CPU. Don't ship the weakest tier by accident.
- **Don't get locked into a library's fixed model list.** A generic `CrossEncoder` (sentence-transformers) loads arbitrary models; some rerank libraries can only load their own catalog.
- **Adaptive cutoff: prefer an absolute score floor over gap detection.** Smooth reranker score distributions put the largest gap between #1 and #2, so gap-based cutoff truncates too aggressively. A calibrated absolute floor (+ a min-results guard) cleanly separates real hits from nonsense queries.

---

## Contextual enrichment (LLM-generated context per chunk)

Adding an LLM-written sentence of context to each chunk *can* help retrieval — but it is **not free and not guaranteed**. Treat it as an experiment, never a default.

- **The metric is extrinsic, never intrinsic.** Enrichment text is usually never read by anyone — it only shifts the embedding. So a "good summary" is irrelevant; the only question is whether retrieval ranking improves. Eval on `query → relevant-chunk` pairs, not `chunk → gold-summary` similarity.
- **It can be net noise.** Enriching *every* chunk with model-voiced boilerplate adds correlated text that **blurs inter-chunk distinctions** and pulls embeddings toward the enrichment's vocabulary instead of the chunk's actual content. Heading-path + raw chunk text may already carry the signal.
- **Gate it on the full index, not a proxy.** Enrichment is the canonical case where a cheap proxy lies (see `ml-heuristics.md` → Proxy vs. Decisive Metric). A small in-memory pool rewards any prompt that makes the positive *distinctive*; only enriching the whole corpus and re-evaluating exposes the real cost. **Pre-register: ships only if it beats the no-enrichment baseline on the hard gate.** It is a legitimate, common outcome that enrichment loses and you ship none.
- **If it ships, match the searcher's vocabulary.** Problem/symptom framing ("what would a dev type when they hit this?") beats keyword-stuffing — that's how real queries and good golden sets are phrased.

---

## Evaluation (retrieval-specific)

Read `ml-heuristics.md` → Eval-Set Validity and Proxy vs. Decisive Metric first; they apply with full force here. Retrieval-specific points:

- **Build the harness first — it's the real deliverable.** A tool-agnostic `query→relevant-chunk` eval harness lets you measure *any* future change (reranker swap, chunk tweak, embedding upgrade), not just the one you're working on. The optimizer/enricher rides on top of it.
- **Primary metrics:** NDCG@10 (rewards ranking positives high), Recall@10 (did we retrieve it at all), MRR (rank of first hit). Score at **parent granularity** if that's what you return.
- **Recall@50 ceiling (fusion only, no rerank) is a diagnostic lever-splitter.** The gap between the ceiling and delivered NDCG separates *"never retrieved"* (an embedding/recall problem — the lever enrichment/chunking targets) from *"retrieved but reranked out"* (a reranker problem). Knowing which lever a miss belongs to stops you tuning the wrong stage.
- **Keep two sets:** a hard (vocab-gap) set that is *the gate*, and an easy (chunk-vocab) set as a *regression guard* so a recall win doesn't quietly break the already-solved case.
- **Diversify query archetypes and report per stratum.** A set that's 88% why/how questions hides whole failure modes — when navigational/keyword/error-symbol archetypes were added to one corpus, navigational queries turned out to be the genuine weak spot (NDCG 0.76), invisible to the old set. Mix archetypes deliberately and break the score down by stratum, not just the aggregate. If an LLM judges relevance, validate the judge against human labels first (e.g. ±1 agreement, P/R) — an unvalidated judge is another unresolved instrument.

---

## Frontier (2026): adopt the consensus, distrust the add-ons

The 2026 production consensus is boring and cheap: **hybrid dense+BM25 → cross-encoder rerank, no LLM in the query path**, routing to anything heavier only for the queries that need it. The frontier mostly *confirms* that foundation and supplies measured negatives worth not re-litigating.

- **The cost dichotomy is the load-bearing lens.** Sort every technique into *per-query-LLM* (agentic loops, GraphRAG global search, RL-trained retrievers like Search-R1) vs *index-time / no-LLM-in-query-path* (hybrid+rerank, HippoRAG 2, LightRAG). Default to the second; reserve per-query-LLM for multi-hop / ambiguous / high-stakes.
- **Enrichment regresses on jargon/extractive corpora** (ConTEB, "Context is Gold"). Prepended LLM prose dilutes rare exact-term signal — worst for BM25 (COVID-QA −21 nDCG@10). Confirms "enrichment is an experiment, never a default," and names *when* it loses: technical/identifier-heavy corpora.
- **Late chunking** (embed whole doc, mean-pool per-chunk spans) buys context without per-chunk LLM or jargon dilution — but **lost measurably on a jargon corpus** (Δ−0.016, paired p=0.045) and **requires a mean-pooling embedder**. The strongest small open embedders (Qwen3-Embedding, Jina v5) are *last-token pooled* → "best embedder" and "late chunking" are mutually exclusive today. Pick per corpus; gate it.
- **Long context doesn't kill RAG for static corpora** (Chroma "Context Rot," 18 models): accuracy degrades with input length *even below the window limit* — a focused ~300-token excerpt beats stuffing 113K tokens. Retrieve-then-focus still wins.
- **You don't need RL retrieval** (FrugalRAG): a plain ReAct agent matches RL-trained Search-R1 on multi-hop. If your consumer is an agent, let *it* generate HyDE/multi-query (see Hybrid retrieval) — "agentic RAG" at zero server-side per-query-LLM cost; don't train a retriever.
- **Multi-hop, only if you actually have it:** HippoRAG 2 is the cheap graph option *when a query pattern demands multi-hop*; full GraphRAG is over-engineering for fact lookup.
- **MTEB rank ≠ your-corpus performance.** Top-MTEB embedders (Qwen3, Gemma) both lost to an incumbent (Jina v5) on the actual corpus. Re-rank embedders on *your* eval set, never on the leaderboard.

---

## The Meta-Principle (retrieval edition)

**Most retrieval quality comes from chunking and hybrid+rerank wiring, not from clever add-ons.** Get the match-unit/context-unit split, real BM25, a decent cross-encoder, and a trustworthy eval harness right *first*. Enrichment, graph indexes, learned sparse retrieval, and ANN compression are all things you add *only when the eval harness says the cheap foundation has run out of headroom* — and each must beat the no-frills baseline on a pre-registered gate to earn its place.
