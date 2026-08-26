# Changelog

## 26 Aug 2026

> Added support for slurm job GPU clock persistence. Added Claude CLI support guide to repo.

**Toolchains**
* Configured GPU peristence via Slurm prolog
* Added system UID package

**Documentation**
* Added claudesupport.md

## 24 Aug 2026

> Slurm scheduler installed, ready for beta testing.

**Toolchains**
* Added Slurm scheduler for GPU intensive tasks

## 20 Aug 2026

> OS install completed. Networking fixed. Storage reshuffled. Toolchains installed.

**Install / OS**
* Ubuntu Server 26.04 LTS installed (custom storage layout, LVM on `vg_nvme`)
* `systemd-networkd-wait-online` disabled

**Storage**
* LVM and partitions done.

**Access**
* Tailscale installed and joined as `ubuntu-makers`, tag `tag:compute`
* Tailscale SSH working from admin laptop
* OpenSSH server installed

**Toolchains**
* Lmod installed, `/opt/modulefiles` on `MODULEPATH`
* Python via `uv` (installed to `/opt/tools/uv`) — 3.10, 3.12, 3.14
* GCC/G++ 13, 14, 15, 16 (default 15)
* OpenJDK 21 and 25
* NVIDIA driver 580.173.02 + CUDA 13.1
  * Removed duplicate `nvidia-cuda-toolkit` (12.4) that was shadowing `nvcc`
  * Patched CUDA headers for the glibc `rsqrt` incompatibility (LP #2155233)
  * Verified end to end: kernel launch + result check on the 4090
* Vivado 2025.1.1 installed to `/opt/Xilinx`, exposed via `module load vivado/2025.1`
  * Headless install required `--batch` mode plus `AuthTokenGen` first
  * Default destination `/tools/Xilinx` had to be changed to `/opt/Xilinx`
* Apptainer installed

**Scratch**
* `/scratch/$USER` now auto-created on SSH login via `pam_exec` → `/usr/local/sbin/mkscratch`
  * Fixes silent fallback where caches landed in `/home` when the directory was missing
* Per-user scratch directories corrected to mode `700`

**Hardware Specification**
* Full spec verified and recorded in `hardwarespec.md`

**Docs**
* Repository documentation written up

## Outstanding

**Blocking onboarding*** [ ] `new-user` / `retire-user` scripts (must use `install -d -m 700` for scratch)
* [ ] Tailscale ACLs tightened
* [ ] ufw + sshd hardening (`AllowGroups compute admins`)
* [ ] Per-user systemd slice limits + `systemd-oomd` — see `hardwarespec.md` for sized values
* [ ] Test student account walked through the full workflow
* [ ] MOTD

**Infrastructure**
* [ ] Scratch purge timer (30 day)
* [ ] Slurm (GPU + FPGA as gres)
* [ ] Backups (`/home` + `/etc`) and a tested restore
* [ ] Monitoring