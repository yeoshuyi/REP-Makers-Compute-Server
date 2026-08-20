# System Quirks

Non-obvious patches and workarounds keeping this machine running. **If you rebuild or upgrade, check this list** — several of these are undone by package updates and will fail confusingly.

---

## 1. CUDA header patched for glibc compatibility

**Symptom:** Any `.cu` file fails to compile:

```
/usr/include/x86_64-linux-gnu/bits/mathcalls.h(206): error: exception specification
is incompatible with that of previous function "rsqrt"
```

**Cause:** Known Ubuntu 26.04 bug (LP #2155233). 26.04's glibc declares these math functions `noexcept(true)`; CUDA 13.1's headers declare them without it. Affects the archive `cuda-toolkit` package on a default install. Not specific to this machine.

**Fix:** patched `/usr/local/cuda-13.1/targets/x86_64-linux/include/crt/math_functions.h` to add `noexcept(true)` to the `rsqrt` and `rsqrtf` declarations. Original preserved as `.orig` alongside.

```bash
sudo sed -i 's/rsqrt(double x);/rsqrt(double x) noexcept(true);/'  "$H"
sudo sed -i 's/rsqrtf(float x);/rsqrtf(float x) noexcept(true);/' "$H"
```

**⚠️ This is overwritten by any CUDA toolkit update.** Mitigations in place:

- `/usr/local/sbin/patch-cuda-headers` — re-applies the patch (**TODO: confirm created**)
- `apt-mark hold cuda-toolkit-13-1` (**TODO: confirm applied**)

**Scope:** only affects compiling `.cu` source. Prebuilt PyTorch/TensorFlow wheels are unaffected.

---

## 2. `systemd-networkd-wait-online` disabled

Added 120 seconds to every boot while networking was broken, and provides nothing on this machine.

```bash
systemctl disable --now systemd-networkd-wait-online.service
```

---

## 3. `sudo -E` doesn't work

`sudo: preserving the entire environment is not supported, '-E' is ignored`

Use `sudo env VAR=value command` instead. Relevant for the Vivado installer, which needs `TMPDIR` redirected away from the 40 GB `/tmp`.

---

## 4. Vivado installer specifics

- **Headless:** must use `--batch` mode; the GUI installer aborts with `No X11 DISPLAY variable was set`
- **Default destination is `/opt/Xilinx`
- **libtinfo5 / libncurses5 symlinks** Backwards symlink for libtinfo6

---

## 5. AppArmor profile required for Apptainer

**Symptom:** any `apptainer` pull or exec dies during extraction with a bare:

```
FATAL: ... root filesystem extraction failed: extract command failed:
ERROR : Failed to create container process: Operation not permitted
```

**Cause:** Ubuntu restricts unprivileged user namespaces via AppArmor. Confirm with:

```bash
sysctl kernel.apparmor_restrict_unprivileged_userns    # returns 1
```

Apptainer needs userns to run rootless containers, so it's blocked by default.

**Fix:** `/etc/apparmor.d/apptainer`

```
abi <abi/4.0>,
include <tunables/global>

profile apptainer /usr/bin/apptainer flags=(unconfined) {
  userns,
  include if exists <local/apptainer>
}
```

```bash
sudo apparmor_parser -r /etc/apparmor.d/apptainer
sudo systemctl reload apparmor
sudo aa-status | grep -i apptainer
```

**Notes:** the path in the profile must match `which apptainer`. This file is not created by the package and **will not survive a rebuild** — the error message gives no hint that AppArmor is involved, so it costs an hour to rediscover.

---

## 6. Scratch directories auto-created via PAM

**Symptom (before the fix):** `/etc/profile.d/00-scratch-env.sh` guards on `[ -d "/scratch/$USER" ]`. If that directory doesn't exist, **the entire block is skipped silently** — no error, and every cache (`TMPDIR`, pip, uv, HuggingFace, torch, Apptainer) lands in the user's home directory instead. Discovered after a 3.4 GB PyTorch container pull went to `~/.apptainer`.

**Fix:** `/usr/local/sbin/mkscratch`, called from PAM on login.

```sh
#!/bin/sh
[ -n "$PAM_USER" ] || exit 0
case "$PAM_USER" in root|nobody|daemon|sshd) exit 0 ;; esac
id -u "$PAM_USER" >/dev/null 2>&1 || exit 0
[ "$(id -u "$PAM_USER")" -ge 1000 ] || exit 0
d="/scratch/$PAM_USER"
[ -d "$d" ] && exit 0
install -d -m 700 -o "$PAM_USER" -g "$PAM_USER" "$d" "$d/tmp" "$d/cache"
exit 0
```

`chmod 700`, owned by root. Then in `/etc/pam.d/sshd`, at the end of the session block:

```
session optional pam_exec.so /usr/local/sbin/mkscratch
```

**Notes:**

- The UID ≥ 1000 guard keeps system accounts out of `/scratch`.
- `optional` is deliberate — a scratch failure must never block a login.
- **Only fires for sshd sessions.** `su -`, `sudo -i`, and console logins do not trigger it. Add the same line to `/etc/pam.d/login` and `/etc/pam.d/su` if that coverage is wanted.
- This lives in the login path, which is not an obvious place to look. If scratch directories stop appearing, check here.

---

## 7. Per-user scratch directories must be mode 700

`/scratch/makers` was initially created `775`, letting every user on the box read it. Corrected to `700`.

`/scratch` itself stays `1777` (sticky, like `/tmp`). It's the per-user directories underneath that need locking down. The `mkscratch` script and any future `new-user` script must use `install -d -m 700`.

---

## Template for new entries

```markdown
## N. Short title

**Symptom:** what you see when it's broken

**Cause:** why

**Fix:** exact commands / file contents

**Notes:** does it survive updates? what undoes it?
```
