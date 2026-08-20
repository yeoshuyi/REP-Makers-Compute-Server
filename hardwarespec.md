# Hardware Specification

Verified: **20 August 2026**

## Motherboard
* Gigabyte X870 EAGLE WIFI7
* BIOS version F7 (released 07/16/2025)
* Required settings: Above 4G Decoding **enabled**, CSM **disabled** (UEFI only)
* Resizable BAR: **TODO — confirm current state in BIOS**

## CPU
* AMD Ryzen 7 7700 — 8 cores / 16 threads

## GPU
* NVIDIA GeForce RTX 4090, 24564 MiB
* Compute capability **8.9** (`sm_89`, Ada Lovelace)
* Driver 580.173.02
* No ECC, no MIG — consumer card, so GPU allocation is all-or-nothing per job

## RAM
* **32 GiB DDR5 installed** (2x 16 GiB @ 5200 MT/s), ~29 GiB usable
* **2 of 4 slots populated** — `DIMM 1` in both Channel A and Channel B
* `DIMM 0` empty in both channels -> **upgradeable to 64 GiB by adding a matched pair**
* Swap: 32 GiB (`lv_swap`)

## Storage
* 1x ADATA LEGEND 860 1 TB NVMe SSD (`nvme0n1`) — OS, home, tools, scratch

### Partition table
```
NAME                   MAJ:MIN RM   SIZE RO TYPE MOUNTPOINTS
nvme0n1                259:0    0 931.5G  0 disk
├─nvme0n1p1            259:1    0     1G  0 part /boot/efi
├─nvme0n1p2            259:2    0     2G  0 part /boot
└─nvme0n1p3            259:3    0 928.5G  0 part
  ├─vg_nvme-lv_root    252:0    0    80G  0 lvm  /
  ├─vg_nvme-lv_swap    252:1    0    32G  0 lvm  [SWAP]
  ├─vg_nvme-lv_var     252:2    0    60G  0 lvm  /var
  ├─vg_nvme-lv_tmp     252:3    0    40G  0 lvm  /tmp
  ├─vg_nvme-lv_opt     252:4    0   300G  0 lvm  /opt
  ├─vg_nvme-lv_home    252:5    0   200G  0 lvm  /home
  └─vg_nvme-lv_scratch 252:6    0   200G  0 lvm  /scratch
```

### Volume group
| | |
|---|---|
| VG name | `vg_nvme` |
| PVs / LVs | 1 / 7 |
| Total | 928.46 G |
| **Free (`VFree`)** | **16.46 G** |

The free extents are deliberately reserved for LVM snapshots (taken before driver/kernel upgrades) and `lvextend` headroom. **Do not allocate them.**

### Filesystem usage

| Mount | Size | Used | Backed up? | Purpose |
|---|---|---|---|---|
| `/` | 79 G | 13 G (18%) | config only | OS |
| `/var` | 59 G | 879 M (2%) | no | logs, apt cache |
| `/tmp` | 40 G | — | no | system temp |
| `/opt` | 295 G | 16 G (6%) | no | Vivado, shared tools, modulefiles |
| `/home` | 196 G | 692 M (1%) | **not yet** | user homes |
| `/scratch` | 197 G | — | **never** | venvs, caches, datasets, builds |

## Network

| Interface | State | MAC | Notes |
|---|---|---|---|
| `enp7s0` | UP | `30:56:0f:71:e7:b6` | School Ethernet |
| `wlp8s0` | DOWN | `50:ee:32:b8:6f:4c` | WiFi7, unused |
| `tailscale0` | UP | — | all user access |

* MagicDNS name: `ubuntu-makers`