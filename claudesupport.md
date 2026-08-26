# Using Claude on the Compute Server

This guide explains how to set up a local Claude CLI or SDK in your userspace and provides a context template to help Claude understand the `ubuntu-makers` system architecture.

## 1. Downloading Claude into Userspace

Since you do not have `sudo` privileges, you must install any Python-based Claude CLI tools or the Anthropic SDK into your personal scratch directory. 

We recommend using `uv` to manage a virtual environment:

```bash
# Navigate to your scratch directory
cd /scratch/$USER

# Create and activate a virtual environment
uv venv claude-env --python 3.12
source claude-env/bin/activate

# Install the Anthropic SDK or your preferred Claude CLI
uv pip install anthropic
```

Remember to export your API key in your session:
```bash
export ANTHROPIC_API_KEY="your-api-key-here"
```

## 2. Claude System Context Addon (`claude.md`)

When using Claude to generate scripts, run jobs, or debug code on this server, it needs to know how our infrastructure works. Provide the following template to Claude (as a system prompt, project knowledge file, or your first message) so it correctly instantiates jobs using Slurm, Apptainer, and Lmod.

---

### `claude.md` Template

```markdown
# System Context: ubuntu-makers Compute Server

You are assisting a user on `ubuntu-makers`, a shared Linux compute server. When providing commands, bash scripts, or debugging steps, you MUST adhere to the following infrastructure rules:

## 1. Toolchains & Modules (Lmod)
- Standard tools (like CUDA, Java, GCC, and Vivado) are not on the default PATH.
- You must instruct the user to use `module load` (e.g., `module load cuda/13.1`, `module load java/21`).
- Instruct the user to use `module avail` to check available software.

## 2. Job Scheduling (Slurm)
- **NEVER** suggest running heavy compute, model training, or synthesis directly on the login shell.
- All heavy tasks MUST be routed through the Slurm scheduler using `srun` or `sbatch`.
- **CPU Jobs:** `srun -p cpu -c 4 --mem=8G <command>`
- **GPU Jobs (RTX 4090):** The GPU is completely invisible without strict allocation. You MUST append `--gres=gpu:1`.
  Example: `srun -p gpu --gres=gpu:1 -c 4 --mem=16G python train.py`
- **Interactive GPU Debugging:** `srun -p interactive --gres=gpu:1 -c 4 --mem=8G --pty bash`

## 3. Containers (Apptainer)
- Docker is not available. The server uses **Apptainer** for rootless containers.
- To run a containerized app (e.g., from Docker Hub), use: `apptainer exec --nv docker://pytorch/pytorch:latest python script.py`
- The `--nv` flag is required for GPU passthrough, and the Apptainer command must be wrapped inside an `srun` GPU allocation.

## 4. Storage & Environments
- All Python virtual environments (venvs), datasets, cache files, and builds MUST be stored in `/scratch/$USER`.
- Do not use `/home/$USER` for heavy I/O or large dependencies.
```
