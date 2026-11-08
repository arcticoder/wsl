# KB index (for Claude; user doesn't read this)
Host: WSL2 Ubuntu on user's desktop "hercules". Not a server; safe to work in. Never delete user code or installs.
- wsl/sudo-and-env.md   how to get sudo (.env format quirk)
- apt/perf-bind-mount.md  why `apt full-upgrade` failed (linux-tools-common) + fix
- upgrade/24.04-to-26.04.md  release upgrade log, backups, status, post-checks
- Lesson: long-running system ops must be launched detached (setsid nohup), not as the tool's background task.
- Current OS: Ubuntu 26.04.1 LTS (since 2026-11-07). python3=3.14; 3.12 held at noble build; see upgrade note.
- Before ANY apt/dpkg op touching linux-tools/linux-perf: umount /usr/bin/perf, then remount (apt/perf-bind-mount.md).
- apps/orcaslicer.md  OrcaSlicer flatpak install + notes
- apps/k1-moonraker.md  Creality K1 root + Moonraker/Mainsail/Fluidd install, MCP printer wiring
- apps/orcaslicer-mcp-fork.md  OrcaSlicer fork + orcaslicer-mcp + K1 sensor/move checks
- session-context.md  paste-ready context block for the next chat + working state (printer, MCP, sudo rules)
