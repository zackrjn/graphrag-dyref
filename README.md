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
> This repository intentionally resides outside OneDrive (`C:\Users\saksh\Desktop\graphrag-dyref`) to avoid synchronization conflicts. Historical prototypes live under `prelim-research/`.

## Roadmap

- Cross-corpus ingestion with Wikidata alignment
- Dynamic graph construction with temporal indexing
- Iterative DyRef engine with bandit scheduling
- Comprehensive evaluation versus strong RAG/GraphRAG baselines
- Reproducible artifacts for publication and artifact review

For planning status and task tracking, refer to the GitHub Project board (created post-initial commit).

## Storage & Cache Setup

### Git LFS

1. Install Git LFS for this repository:
   ```
   git lfs install --local
   ```
2. Track the large artifact types we expect to check in (update patterns as needed when new formats appear):
   ```
   git lfs track "*.parquet"
   git lfs track "*.bin"
   git lfs track "*.pt"
   git lfs track "*.pkl"
   git add .gitattributes
   ```
3. Commit the updated `.gitattributes` so that future clones pick up the tracking rules.

### DVC Remote (Roar)

1. Log in to Roar and create the shared storage path once:
   ```
   mkdir -p /storage/work/sjr6223/dvc_remote
   ```
2. Inside this repository, register the remote and make it the default:
   ```
   dvc remote add -d roar-storage file:///storage/work/sjr6223/dvc_remote
   dvc remote modify roar-storage type local
   ```
3. Run `dvc push` after adding large artifacts to confirm the remote works.

### Roar Cache Directories & HF Caches

1. On Roar, prepare the cache directories used by Hugging Face, torch, and other artifacts:
   ```
   mkdir -p /storage/work/sjr6223/.cache/hf
   mkdir -p /storage/work/sjr6223/.cache/hf/datasets
   mkdir -p /storage/work/sjr6223/.cache/torch
   ```
2. Append the following exports to `~/.bashrc` (or the shell profile you use on Roar):
   ```
   export HF_HOME=/storage/work/sjr6223/.cache/hf
   export TRANSFORMERS_CACHE=$HF_HOME/transformers
   export HF_DATASETS_CACHE=$HF_HOME/datasets
   export HF_HUB_CACHE=$HF_HOME/hub
   export TORCH_HOME=/storage/work/sjr6223/.cache/torch
   ```
3. On Windows workstations, set the caches outside OneDrive so they align with the Roar layout (`setx HF_HOME C:\hf_cache` and `setx TRANSFORMERS_CACHE C:\hf_cache\transformers`).

After completing these steps, close the GitHub issues tracking Git LFS/DVC configuration and Roar cache setup (#6, #7, #8).

