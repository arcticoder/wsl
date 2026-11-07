I created the KB in [docs/kb/](docs/kb/) and haven't committed it. It has these notes:

- [README.md](docs/kb/README.md): index.
- [wsl/sudo-and-env.md](docs/kb/wsl/sudo-and-env.md): the `.env` format uses a space, not `=`, so you can't `source` it. The note has the awk one-liner and no password.
- [apt/perf-bind-mount.md](docs/kb/apt/perf-bind-mount.md): the cause and fix for your apt errors. The WSL kernel bind-mounts its own `perf` over `/usr/bin/perf`, which `dpkg` can't replace. I unmounted it, finished the upgrade, and remounted it.
- [upgrade/24.04-to-26.04.md](docs/kb/upgrade/24.04-to-26.04.md): backup locations, the exact command, and a post-upgrade checklist.

`.env` is gitignored.

The 26.04 upgrade is still running. I'll update the status log in the upgrade note when it finishes.
<!-- ------ -->
---