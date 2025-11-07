## Environment Setup Verification

Baseline environment configuration for DyRef-GraphRAG was validated on 2025-11-07 across Windows (local workstation), Roar, and the i4 GPU server. Each environment now shares a consistent Git, Git LFS, and DVC setup to support multi-agent development.

### Windows Workstation (`C:\Users\saksh\Desktop\graphrag-dyref`)
- Repository synced from `main`; Python 3.11 environment installs `requirements.txt` cleanly.
- Git LFS initialized (`git lfs install --local`) with tracked patterns in `.gitattributes`.
- Local cache directories configured (`HF_HOME=C:\hf_cache`, `TRANSFORMERS_CACHE=C:\hf_cache\transformers`) outside OneDrive.
- DVC ready to interact with Roar remote once data artifacts are pushed.

### Roar Cluster (`/storage/work/sjr6223/projects/graphrag-dyref`)
- Repository cloned with Git LFS enabled; `.bashrc` exports point HF and torch caches to `/storage/work/sjr6223/.cache`.
- DVC remote `roar-storage` registered at `file:///storage/work/sjr6223/dvc_remote`; `dvc pull/push` confirms access.
- Python 3.11 environment created and able to install `requirements.txt`; GPU access verified via `nvidia-smi`.

### i4 GPU Server (`/home/sjr6223/projects/graphrag-dyref`)
- Project directory created under `/home/sjr6223/projects`; repository cloned with Git LFS enabled.
- Cache directories configured to mirror Roar layout, ensuring consistent Hugging Face and torch usage.
- Python dependencies install cleanly; `nvidia-smi` reports access to RTX A6000 (×2) and A100 GPUs for future workloads.

### Next Steps
- Large artifacts should be staged via DVC on Roar, then mirrored to i4 as needed (`dvc pull`).
- Document deviations or additional environment needs in this file to keep multi-agent workflows aligned.

