# DyRef-GraphRAG

DyRef-GraphRAG is a dynamic refinement framework for retrieval-augmented generation over evolving scientific corpora. It combines PMC Open Access (2021–2025) and arXiv cs.AI/cs.CL (2021–2025), cross-linked through Wikidata, to enable multi-hop, temporal-aware reasoning with adaptive retrieval budgets.

## Repository Structure

```
data_ingestion/   # Corpus downloads, parsing, metadata extraction
processing/       # Normalization and chunking pipelines
nlp/              # NER, linking, relation extraction modules
graph/            # Graph schema, Neo4j loaders, indexing scripts
retrieval/        # Hybrid seed retrieval, DyRef engine, scoring
services/         # Long-running services (ANN, reranking, summarization)
eval/             # Evaluation harness, baselines, metrics, plotting
scripts/          # CLI, orchestration, and HPC job scripts
notebooks/        # Analysis and exploratory investigations
configs/          # Configuration files for pipelines and services
docs/             # Design notes, specifications, and experimental logs
```

## Getting Started

1. Install Python 3.11 and create a fresh environment (conda or venv).
2. Install dependencies (`pip install -r requirements.txt` or `pip install -e .` once `pyproject.toml` is finalized).
3. Set `HF_HOME` and `TRANSFORMERS_CACHE` outside OneDrive (e.g., `C:\hf_cache` locally, `/storage/work/<user>/.cache` on Roar).
4. Review `configs/` for connection details, especially Neo4j and FAISS services.

> **Note**  
> This repository intentionally resides outside OneDrive to avoid synchronization conflicts. Historical prototypes live under `prelim-research/`.

## Roadmap

- Cross-corpus ingestion with Wikidata alignment
- Dynamic graph construction with temporal indexing
- Iterative DyRef engine with bandit scheduling
- Comprehensive evaluation versus strong RAG/GraphRAG baselines
- Reproducible artifacts for publication and artifact review

For planning status and task tracking, refer to the GitHub Project board (created post-initial commit).

