# Software Support

## Linux Distribution

Ubuntu Server LTS 26.04 (Linux 7.0 kernel)
Last APT update: 20 August 2026
---

## Using modules (read this first)

Toolchains are **not** on your `PATH` by default. You can select what you want per session with `module`.

```bash
module avail              # list everything available
module load cuda/13.1     # add a toolchain to this shell
module list               # what's currently loaded
module unload cuda/13.1   # remove one
module purge              # remove all
module spider java        # search, incl. versions not shown by avail
```

Modules apply to **the current shell only**. A new SSH session starts clean.

To load things automatically on login, add to your `~/.bashrc`:

```bash
# Example
module load cuda/13.1 java/21
```

Do this sparingly — the point of modules is that different projects can use different versions without interfering.

Submitting jobs via Slurm? Put the `module load` lines inside the job script, not just in your shell.

---

## Tailscale

All access to this machine is over Tailscale.
```bash
sudo tailscale up
tailscale status          # confirm ubuntu-makers is listed
ssh USER@ubuntu-makers
```

`ubuntu-makers` is the MagicDNS name — no IP address needed.

**TODO:** ACL policy is currently permissive (allow-all) to avoid lockout during setup.

---

## Lmod

Installed from the Ubuntu archive. System modulefiles live in `/opt/modulefiles`, added to `MODULEPATH` via `/etc/profile.d/01-modulepath.sh`.

To add a new toolchain, drop a `.lua` file in the appropriate subdirectory and `chmod a+rX` it. Users pick it up on next login.

---

## Python (uv)

Python 3.10, 3.12, and 3.14 are installed globally and managed via [`uv`](https://docs.astral.sh/uv/), which is on the default `PATH` — no module load needed.

### Standard workflow

```bash
cd /scratch/$USER
uv venv myproject --python 3.12
source myproject/bin/activate
uv pip install torch numpy pandas
```

### Where to put environments

**`/scratch/$USER`, never your home directory.** Package caches (pip, uv, conda, HuggingFace, torch) are already redirected to scratch automatically.

Reminder: `/scratch` is not backed up, and a 30-day purge is planned. Keep source code in `~` or git; keep environments and data on scratch.

### Available interpreters

```bash
uv python list
```

---

## Java

OpenJDK 21 and 25, both LTS.

```bash
module load java/21       # or java/25
java -version
echo $JAVA_HOME
```

Maven and Gradle caches are redirected to `/scratch/$USER/cache` automatically.

There is deliberately **no system-wide `JAVA_HOME`**
---

## C / C++

`build-essential` is installed and `gcc`/`g++` are on the default `PATH` at version 15.

Versions 13, 14, 15, and 16 are all available. The system default is **15**. To use another:

```bash
module load gcc/13
echo $CC $CXX
```

Or invoke directly: `gcc-14`, `g++-13`, etc.

Also available: `clang`, `cmake`, `ninja`, `meson`, `gdb`, `valgrind`, `ccache` (cache on scratch).

---

## CUDA 13.1

```bash
module load cuda/13.1
nvcc --version
nvidia-smi
```

**GPU:** NVIDIA GeForce RTX 4090 (24 GB, compute capability `sm_89`)
**Driver:** 580.173.02

### Host compiler

The CUDA 13.1 headers reject GCC newer than 15. The system default is exactly 15, so this works out of the box, and the `cuda/13.1` module pins `CUDAHOSTCXX=/usr/bin/g++-15` so a future default-compiler change won't silently break builds.

If you ever see `unsupported GNU version`, load the module, or compile with:

```bash
nvcc -ccbin g++-15 foo.cu -o foo
```

### Compiling for this GPU

```bash
nvcc -arch=sm_89 foo.cu -o foo
```
---

## Vivado

Vivado 2025.1.1, installed at `/opt/Xilinx`, free (BASIC) tier licence.

```bash
module load vivado/2025.1
vivado -version
```

Headless batch synthesis:

```bash
cd /scratch/$USER/myproject
vivado -mode batch -source build.tcl -log out/vivado.log -journal out/vivado.jou
```

Always pass `-log` and `-journal` explicitly, or Vivado litters the working directory.

**Projects belong on `/scratch/$USER`**
Cap threads so one synthesis run doesn't starve the box (8 cores total):
```tcl
set_param general.maxThreads 4
```

## Containers (Apptainer)

Installed to allow software without sudo. Runs unprivileged, reads Docker images directly, passes the GPU through with `--nv`:

```bash
## Example
cd /scratch/$USER
apptainer exec --nv docker://pytorch/pytorch:latest \
  python -c "import torch; print(torch.cuda.is_available())"
```

Images are large. `APPTAINER_CACHEDIR` points at `/scratch/$USER/cache/apptainer` automatically — verify with `echo $APPTAINER_CACHEDIR` before a big pull.
Building your own image:

```bash
apptainer build myimage.sif mydef.def
```