I created the KB in [docs/kb/](docs/kb/) and haven't committed it. It has these notes:

- [README.md](docs/kb/README.md): index.
- [wsl/sudo-and-env.md](docs/kb/wsl/sudo-and-env.md): the `.env` format uses a space, not `=`, so you can't `source` it. The note has the awk one-liner and no password.
- [apt/perf-bind-mount.md](docs/kb/apt/perf-bind-mount.md): the cause and fix for your apt errors. The WSL kernel bind-mounts its own `perf` over `/usr/bin/perf`, which `dpkg` can't replace. I unmounted it, finished the upgrade, and remounted it.
- [upgrade/24.04-to-26.04.md](docs/kb/upgrade/24.04-to-26.04.md): backup locations, the exact command, and a post-upgrade checklist.

`.env` is gitignored.

The 26.04 upgrade is still running. I'll update the status log in the upgrade note when it finishes.
<!-- ------ -->
---
The 26.04 upgrade is running but not finished. The system is still on 24.04.5. I'll be notified when it exits and will check the result then.

- **Why `apt full-upgrade` failed:** the WSL kernel bind-mounts its own `perf` over `/usr/bin/perf`, and `dpkg` can't replace a mount point. I unmounted it, finished the upgrade, and remounted it. This will come back whenever `linux-tools-common` updates, and the same steps fix it.
- **The interrupted first run:** the session ending killed the upgrader. It had rewritten the apt sources to `resolute` and disabled the third-party repos, but installed nothing. I set the apt config aside, restored your original `/etc/apt` from my backup, and relaunched the upgrader detached from this session so it can't be killed that way again.
- **What the upgrade will change:**
  - The plan removes no packages.
  - Ubuntu 26.04 swaps in `coreutils-from-uutils` (Rust coreutils).
  - The third-party repos (CUDA, GitHub CLI, KiCad, deadsnakes, nvidia-container-toolkit, webmin, zotero) will be disabled during the upgrade. I'll check which have 26.04 builds afterward and re-enable or repoint them.
- **Backups:** your package list and apt config are in `~/ubuntu-24.04-backup`.
- **KB:** I added the interruption, recovery and detached-launch notes to `docs/kb/upgrade/24.04-to-26.04.md`. I haven't touched `docs/history.md`.
<!-- ------ -->
---