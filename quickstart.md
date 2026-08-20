# Quickstart Guide

`ubuntu-makers` — REP Makers shared compute server.

---

## Onboarding

### What you need to do

**1. Get a Tailscale account.**
Get an invite link from maintainer. Install Tailscale on the machine you'll connect from:

| Platform | Install |
|---|---|
| macOS / Windows | Download from tailscale.com/download |
| Linux | `curl -fsSL https://tailscale.com/install.sh \| sh` |
| iOS / Android | App Store / Play Store |

Then `tailscale up` and sign in with your Tailscale account.

**2. Send an admin your SSH public key.**

```bash
ssh-keygen -t ed25519          # if you don't already have one
cat ~/.ssh/id_ed25519.pub      # send this — never send the file without .pub
```

**3. Wait for confirmation**, then test:

```bash
ssh USER@ubuntu-makers
```

### What an admin does

**TODO — the `new-user` script is not yet written.** Intended:

```bash
sudo /usr/local/sbin/new-user <username> '<ssh-pubkey>' '<Full Name>'
```

Which will: create the account with no password (keys only), add to `compute` and `fpga` groups, apply the 20 GB `/home` quota, install the SSH key, and create `/scratch/<username>`.

Outstanding before onboarding can start:
- [ ] `new-user` / `retire-user` scripts written
- [ ] Tailscale ACLs tightened (currently allow-all)
- [ ] `AllowGroups compute admins` in sshd, plus the rest of the SSH hardening
- [ ] ufw enabled (`default deny incoming`, `allow in on tailscale0`)
- [ ] Per-user resource limits (systemd slice)
- [ ] Scratch purge timer
- [ ] MOTD with the house rules

---

## General usage

### Connecting

```bash
sudo tailscale up
tailscale status              # confirm ubuntu-makers is online
ssh USER@ubuntu-makers
```

`ubuntu-makers` is a MagicDNS name — no IP needed. If it doesn't resolve, check that Tailscale is actually up on your machine.

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

**Scratch is deleted after 30 days of no access and is never backed up.** Keep source in git. Keep anything you'd cry about losing off this machine entirely.

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

### Python

```bash
cd /scratch/$USER
uv venv myproject --python 3.12
source myproject/bin/activate
uv pip install numpy torch
```

`pip install` outside a venv fails with `externally-managed-environment` — that's expected, not broken.

### GPU

**TODO — Slurm not yet configured.**

```bash
nvidia-smi
```

Once Slurm is up, GPU work goes through the scheduler rather than run directly.

### Need software we don't have?

1. **Python packages** — install in your own venv, no admin needed
2. **Containers** — Apptainer (TODO: not yet installed) runs Docker images unprivileged
3. **Use SUDO Account** — if you know what you're doing, please don't break the server

---
