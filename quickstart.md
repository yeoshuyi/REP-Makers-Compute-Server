# Quickstart Guide

`ubuntu-makers` — REP Makers shared compute server.

---

## Onboarding
Authentication is by **Tailscale identity**

### For users

**1. Install Tailscale** on whatever machine you'll connect from, and get invited to the network:

| Platform | Install |
|---|---|
| macOS / Windows | Download from [tailscale.com/download](https://tailscale.com/download) |
| Linux | `curl -fsSL https://tailscale.com/install.sh \| sh` |
| iOS / Android | App Store / Play Store |

**2. Sign in** with your school account:

```bash
tailscale up
tailscale status          # ubuntu-makers should appear in the list
```

**4. Connect.**

```bash
ssh YOUR_USERNAME@ubuntu-makers
```

**5. Check it worked:**

```bash
echo $TMPDIR              # should be /scratch/YOUR_USERNAME/tmp
module avail              # should list cuda, java, gcc, vivado
```

### For Account Creation
> Requires Sudo Account

```bash
sudo new-user alice 'Alice Tan'
```

Then in the Tailscale admin console: invite the user, and add them to `group:students` in the ACL policy.

Verify before handing over:

```bash
sudo -u alice bash -lc 'echo $TMPDIR; module avail'
```

**The Unix username must match** what Tailscale derives from their login if the ACL uses `${dst_user}` mapping. Check this with a test account before onboarding a group.

### Offboarding

```bash
sudo retire-user alice
```

Locks the account, kills their processes, archives `/home/alice` to `/opt/archive`, and deletes their scratch. **Does not delete the account** — verify the tarball first, then:

```bash
sudo userdel -r alice
```

Also remove them from the tailnet, or disable their school SSO account if that's the identity source.

## General usage

### Connecting

```bash
sudo tailscale up
tailscale status              # confirm ubuntu-makers is online
ssh USER@ubuntu-makers
```

`ubuntu-makers` is a MagicDNS name — no IP needed, and no SSH key. If it doesn't resolve, check that Tailscale is actually up on your machine.

### Where to put things

| | Path | Quota | Backed up | Purged |
|---|---|---|---|---|
| Code, configs, notes | `~` (`/home/USER`) | 20 GB *(planned, not enforced)* | not yet | no |
| Environments, data, builds | `/scratch/USER` | none | **no** | after 30 days *(planned)* |

```bash
cd /scratch/$USER
```

Check your usage:

```bash
du -sh ~/* | sort -h | tail
quota -s          # once quotas are enabled
```

**Scratch is deleted after 30 days of no access and is never backed up.** 
Package caches (pip, uv, conda, HuggingFace, torch, ccache, Gradle, Maven) and `$TMPDIR` are already pointed at scratch for you — you don't need to configure that. Your `/scratch/USER` directory is created automatically the first time you SSH in.

### Loading toolchains

Nothing is on your `PATH` by default except Python/uv and the default GCC. Use `module`:

```bash
module avail                  # what's here
module load cuda/13.1
module load java/21
module list
```

Per-shell, per-session. See the software support doc for details on each toolchain.

### Claude
Claude has to understand the Slurm + Apptainer workflow. Refer to claudesupport.md.

### Python

```bash
cd /scratch/$USER
uv venv myproject --python 3.12
source myproject/bin/activate
uv pip install numpy torch
```

`pip install` outside a venv fails with `externally-managed-environment`

Setting up the venv is fine on the login shell. Running your code goes through Slurm:

```bash
srun -p cpu -c 4 --mem=8G python script.py
srun -p gpu --gres=gpu:1 -c 4 --mem=16G python train.py
```

### GPU

One RTX 4090, shared. **GPU work goes through Slurm** — a job without `--gres=gpu:1` cannot see the card at all.

```bash
srun -p gpu --gres=gpu:1 -c 4 --mem=16G python train.py     # run one command
srun -p interactive --gres=gpu:1 -c 4 --mem=8G --pty bash   # interactive shell
sbatch job.sh                                                # submit and walk away
```

Check what's happening:

```bash
sinfo                     # node and partition state
squeue                    # everything queued and running
squeue -u $USER           # just yours
scancel 1234              # kill a job
```

Current load is also shown when you log in. Partitions: `cpu` (default), `gpu`, `interactive` — see the software support doc.

### Need software we don't have?

You don't have sudo — this is deliberate on a shared machine. Options:

1. **Python packages** — install in your own venv, no admin needed
2. **Containers** — Apptainerruns Docker images unprivileged
3. **Use SUDO** — if you know what you're doing

---