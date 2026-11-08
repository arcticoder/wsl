# wsl

Personal sysadmin notes and a small knowledge base for an Ubuntu 24.04 → 26.04 WSL2 setup. It covers the upgrade and its recovery steps, apt and dpkg fixes, OrcaSlicer, and a Creality K1 running Klipper and Moonraker, plus the MCP tooling used to drive them.

## Contents

- `docs/kb/` — the knowledge base, grouped by topic:
  - `upgrade/24.04-to-26.04.md` — release upgrade procedure, failures, and recovery.
  - `apt/perf-bind-mount.md` — why `apt` fails on WSL2 when `/usr/bin/perf` is bind-mounted, and how to work around it.
  - `wsl/sudo-and-env.md` — how `sudo` and the local `.env` file are handled (no secrets are stored here).
  - `apps/orcaslicer.md`, `apps/orcaslicer-mcp-fork.md` — OrcaSlicer install and the MCP control-API fork.
  - `apps/k1-moonraker.md` — installing Moonraker and the Mainsail and Fluidd web UIs on a rooted Creality K1.
- `docs/history.md` — raw chat log from the sessions that produced these notes.

## Conventions

- Credentials live in a local `.env` file, which is git-ignored and never committed. Notes refer to variable names only.
- Notes describe the environment as it was when written. Re-verify commands before reusing them.

## License

MIT. See [LICENSE](LICENSE).
