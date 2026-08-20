# REP-Makers-Compute-Server

> Documentation for the REP Makers shared compute server (`ubuntu-makers`).
Access is over Tailscale only; users do not have sudo.

---

## Start here

* **Using the machine?** → [`quickstart.md`](quickstart.md)
* **What's installed and how to load it?** → [`sofwaresupport.md`](sofwaresupport.md)
* **What's the machine?** → [`hardwarespec.md`](hardwarespec.md)
* **Something behaving oddly?** → [`systemquirks.md`](systemquirks.md)

---

## Latest changelog

### 20 Aug 2026
> OS install completed. Networking fixed, storage reshuffled, toolchains installed.
> Tailscale, Lmod, Python (uv), GCC, OpenJDK, CUDA 13.1. Vivado in progress.

Full history in [`CHANGELOG.md`](CHANGELOG.md).

---

## Current support

| | Status |
|---|---|
| Ubuntu Server 26.04 LTS | ✅ |
| Tailscale access | ✅ (ACLs still permissive) |
| Lmod modules | ✅ |
| Python 3.10 / 3.12 / 3.14 via uv | ✅ |
| C / C++ (GCC 13–16, clang) | ✅ |
| Java (OpenJDK 21, 25) | ✅ |
| CUDA 13.1 + RTX 4090 | ✅ |
| Vivado 2025.1.1 | ✅ |
| Onboarding scripts | ✅ |
| Slurm scheduler | ⬜ not started |
| Apptainer | ✅ |
| Backups | ⬜ not started |
| Monitoring | ⬜ not started |

---

## At a glance

| | |
|---|---|
| Hostname | `ubuntu-makers` (Tailscale MagicDNS) |
| OS | Ubuntu Server 26.04 LTS, kernel 7.0 |
| CPU | AMD Ryzen 7 7700 (8C/16T) |
| RAM | 32 GiB DDR5 (2 of 4 slots used — upgradeable) |
| GPU | NVIDIA RTX 4090 24 GB (`sm_89`) |
| Storage | 1 TB NVMe — `/home` 200 G, `/scratch` 200 G (purged, not backed up) |
| Access | Tailscale only, SSH keys, no user sudo |
---

## Repository directory

```
.
├── README.md            this file — index and status
├── CHANGELOG.md         dated log of changes, plus outstanding work
├── quickstart.md        for users: connecting, where files go, loading toolchains
├── sofwaresupport.md    what's installed, how to use each toolchain
├── hardwarespec.md      machine specs, partition layout, known constraints
└── systemquirks.md      patches and workarounds — read before rebuilding/upgrading
```

---

## Conventions

* **Admin-installed software goes in `/opt`**, exposed via Lmod modulefiles in `/opt/modulefiles`. Never in a home directory.
* **Every non-obvious fix gets an entry in `systemquirks.md`.**
* **Changelog entries are dated** and note what's outstanding, not just what was done.
