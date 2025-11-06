# DyRef-GraphRAG: Dynamic Refinement for Multi-hop and Temporal Reasoning over Evolving Scientific Corpora

## Scope and Objectives
- Build a cross-corpus knowledge graph from PMC Open Access (2021–2025) and arXiv cs.AI/cs.CL (2021–2025), cross-linked via Wikidata.
- Implement DyRef-GraphRAG: an iterative, budgeted retrieval-refinement engine with temporal constraints and cross-granularity routing (doc → section → paragraph → sentence).
- Evaluate on multi-hop QA and scientific claim verification against strong text RAG and GraphRAG baselines; report efficiency metrics and ablations.

## Datasets and Ingestion
- **PMC OA**: titles, abstracts, full text (where licensing permits), metadata (PMID/PMCID/DOI, dates, authors, affiliations).
- **arXiv cs.AI/cs.CL**: metadata, abstracts, parsed sections (API or S2ORC/parsed dumps), arXiv IDs, dates, version info.
- **Wikidata cross-linking**: SAME_AS links via DOI/PMID/arXiv IDs, ORCID/QIDs for authors, institutions, and concepts.
- Storage layout: `data/raw/{pmc,arxiv}`, normalized parquet in `data/processed/`, Neo4j graph store in `graph/neo4j` (or external).

## Graph Construction
- **Nodes**: `Paper`, `Author`, `Institution`, `Concept` (Wikidata), `Venue`, `Section`, `Paragraph`, `Sentence`, `Topic`.
- **Edges**: `MENTIONS`, `CITES`, `AUTHORED_BY`, `AFFILIATED_WITH`, `SAME_AS`, `CO_OCCURS`, `REFERS_TO`, with temporal properties (`valid_from`, `valid_at`).
- **NLP**: scispaCy (PMC) and generic NER/keyphrase extraction (arXiv) for concept detection; relation extraction via co-occurrence inside sentence/section and dependency patterns.
- **Temporal indexing**: `paper_date`, `citation_date`, arXiv revision metadata for time-aware retrieval.

## DyRef-GraphRAG Method
- Seed retrieval with hybrid BM25 + dense models (E5-large, BGE) over multiple granularities, projected onto the knowledge graph.
- Iterative refinement loop (budgeted steps):
  1. Expand frontier via `CITES`/`MENTIONS`/`CO_OCCURS` edges with beam/BFS; apply temporal filters when queries specify a time.
  2. Route between granularities (document → section → paragraph → sentence) based on uncertainty/confidence scores.
  3. Score candidate subgraphs using dense similarity, graph metrics (degree, recency), and a cross-encoder re-ranker.
  4. Summarize evidence into compact contexts with a compressive summarization module; cache reusable outputs.
  5. Multi-armed bandit policy decides whether to expand, prune, summarize, or stop, balancing accuracy and token/cost budgets.
- Temporal conflict handling detects contradictory evidence and surfaces time-stamped answers with provenance trails.

## Baselines
- Text RAG: BM25+LLM, dense retrieval (E5/BGE) + LLM, ColBERTv2 + LLM, each with cross-encoder re-ranking.
- Graph baselines: static GraphRAG neighborhood summarization, StepChain-style multi-step traversal, vanilla GraphRAG single-pass.
- Oracle recall baselines when gold evidence is available.

## Evaluation Tasks and Metrics
- Multi-hop QA: QASPER-style scientific QA, Musique/HotpotQA slices; report EM/F1 and evidence precision/recall/F1.
- Scientific claim verification: SciFact/SciFact-Open adaptations; label accuracy and evidence F1.
- Temporal QA set (1–2k queries from PMC/arXiv headlines with time constraints); temporal consistency rate.
- Efficiency metrics: latency, retrieved tokens, context tokens, and cost; distribution of refinement steps.

## Ablations
- Remove temporal filtering; fixed-depth traversal (no bandit); text-only RAG; graph-only retrieval; fixed granularity (document-only vs sentence-level); without re-ranking; without summarization cache.

## Compute Orchestration (A100 + 2×A6000 + Roar)
- **A100 (80GB)**: high-throughput embedding generation, FAISS/ColBERT index builds, large-batch cross-encoder sweeps.
- **A6000 (48GB×2)**: re-ranking service, summarization cache generation (8–13B models), GPU-accelerated NER batches.
- **Roar Collab (up to 4 nodes, 24 cores)**: distributed ingestion, spaCy/scispaCy multiprocessing, citation graph extraction, Neo4j bulk import prep.
- Neo4j tuning: heap/page cache sizing, constraints and composite indexes for `Paper`/`Author`/`Concept`, periodic commit batching.
- Concurrency: stagger FAISS builds with KG ingestion, cap simultaneous GPU jobs to avoid VRAM thrash, shard workloads.

## Repository and Workflow
- Repo lives at `C:\Users\saksh\Desktop\graphrag-dyref` (outside OneDrive to prevent sync conflicts).
- Branching: `main` (protected), `develop`, `feature/*`; pull requests with CI checks before merge.
- Directory structure (top-level):
  ```
  data_ingestion/   # Downloads, parsing, metadata extraction
  processing/       # Normalization, chunking
  nlp/              # NER, linking, relation extraction
  graph/            # Graph schema, loaders, Neo4j utilities
  retrieval/        # Hybrid retrieval, DyRef engine, scoring, temporal modules
  services/         # Long-running services (ANN, reranker, summarizer)
  eval/             # Evaluation harness, datasets, metrics, plots
  scripts/          # CLI orchestration, HPC Slurm scripts
  notebooks/        # Diagnostics, exploratory analysis
  configs/          # Config templates (Neo4j, retrieval, caching)
  docs/             # Design notes, experiment logs
  ```
- CI: GitHub Actions running lint (`ruff`), type-check (`mypy`), tests (`pytest`); pre-commit hooks for formatting and hygiene.
- Data/versioning: Git LFS for small binaries; DVC remote at `/storage/work/sjr6223/dvc_remote`; caching on Roar under `/storage/work/sjr6223/.cache`.
- GitHub Issues/Project board: single source of truth for tasks; labels (`Agent:A/B/C`, `Area:*`, `Status:*`, `Priority:*`); PR template uses “Closes #ID”.

## Multi-Agent Execution in Cursor (≤ 3 concurrent)
- **Agent A (Data/KG)**: ingestion, normalization, Wikidata linking, graph schema/import, Roar job scripts.
- **Agent B (Retrieval/Engine)**: hybrid retrieval, DyRef loop, scoring, temporal modules, services.
- **Agent C (Evaluation/Paper)**: baselines, evaluation harness, analysis, figures, paper writing.
- Coordination: CODEOWNERS per directory, small feature branches, daily sync notes, avoid simultaneous edits on same modules, shared configs in `configs/`.

## Figures and Tables (planned)
- System architecture diagram; KG statistics (node/edge counts, degree distributions).
- Accuracy vs refinement steps; recall@k curves; latency vs accuracy (Pareto).
- Ablation table; temporal conflict case studies; token/latency breakdown.

## Timeline (6–8 weeks)
- **Week 1**: Repo/CI hardening; ingestion + normalization for 100k–200k papers; schema/index design; Neo4j tuning.
- **Week 2**: NER/linking/relations; KG population; seed retrieval + FAISS index prototypes; sanity metrics.
- **Week 3**: DyRef MVP (beam/BFS + re-ranker); summarization cache; preliminary evaluation slices.
- **Week 4**: Temporal filter; granularity router; caching improvements; telemetry + tracing.
- **Week 5**: Bandit policy training; complete baselines; scale to 300k–400k papers.
- **Week 6**: Full evaluations; ablations; plots/tables; begin paper drafting.
- **Weeks 7–8 (buffer)**: Additional experiments, polishing, reproducibility pack, internal reviews.

## Risks and Mitigations
- **Scale / cost**: cap initial corpus to 300k–400k papers; shard KG and FAISS builds; batch Neo4j imports; monitor GPU utilization.
- **Linking noise**: rely on DOI/PMID/arXiv/ORCID IDs; conservative merges; manual spot checks on samples.
- **Licensing/compliance**: use PMC OA and arXiv metadata or derived features; release scripts and IDs, not restricted texts.
- **Coordination overhead**: enforce CODEOWNERS and PR reviews; maintain concise daily updates; keep GitHub issues as canonical status.

## Paper Outline (ACL/EMNLP Long)
1. **Introduction** – motivation, contributions, overview.
2. **Related Work** – GraphRAG, temporal RAG, agentic retrieval.
3. **Corpus & Knowledge Graph** – dataset construction, Wikidata linking, schema.
4. **Method: DyRef-GraphRAG** – retrieval loop, temporal reasoning, granularity control, bandit policy.
5. **Experimental Setup** – tasks, baselines, metrics, compute resources.
6. **Results** – main quantitative tables, efficiency analysis.
7. **Ablations & Analysis** – component studies, temporal case studies, error breakdown.
8. **Limitations & Ethics** – data coverage, computational cost, responsible use.
9. **Conclusion** – summary and future directions.
10. **Appendix** – implementation details, prompts, hyperparameters, dataset construction notes.

