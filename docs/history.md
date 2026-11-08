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
Your WSL is now on Ubuntu 26.04.1 LTS (Resolute Raccoon), and `apt update && apt full-upgrade` runs cleanly. Nothing from your original manual-install list is missing, `~/Code` is intact, and conda and the GPU (`nvidia-smi`) still work. I haven't rebooted or tested Docker, KiCad or webmin.

- **Why apt was failing:** the WSL kernel mounts its own `perf` over `/usr/bin/perf`, and `dpkg` can't replace a mount point. It hit again during the release upgrade and aborted it near the end. I unmounted `perf` to finish and then remounted it. It will recur whenever a `perf` package updates, so unmount `/usr/bin/perf` first and remount it after.
- **Python 3.12:** Ubuntu 26.04 doesn't ship it, and the deadsnakes build clashes with the old noble packages. Your venvs link to it and need 3.12.3.
  - **What I did:** I kept the noble 3.12.3 packages and held them, so `apt upgrade` will list 5 packages as "kept back". That is expected.
  - **My mistake:** apt's repair step removed `python3.12-venv` and `python3.12-dev`. I restored both from the noble archive with checksums verified.
- **Venvs:** 26.04 made `python3` 3.14, which silently switched 10 of your 3.12 venvs to 3.14 and hid their packages. I repointed their interpreter links to `/usr/bin/python3.12`. Under `~/Code` I changed `vid`, `su2-node-matrix-elements`, `cosmological-constant-workbench`, `su2-3nj-unified-framework`, and two venvs in `warp-bubble-qft`. The other four are in `~/.venv`, `~/.venvs` and `~/refact-server`. The old link targets are in `~/ubuntu-24.04-backup/venv-symlink-changes.txt`.
- **Third-party repos:** CUDA, GitHub CLI, nvidia-container-toolkit, webmin, zotero, KiCad and deadsnakes are enabled again and updating. I left the cockpit-files PPA disabled because it has no 26.04 release, and the two Microsoft repos stay disabled as before.
- **`autoremove`:** I haven't run it, and I'd skip it. Its list includes `python3.12`, `openjdk-21` and old `g++-13`, which you may use.

Run `wsl --shutdown` from Windows and reopen the terminal. The upgrade replaced the dbus daemon, which only restarts that way.

The KB in `docs/kb/` has the details, and I didn't touch `docs/history.md`. It already shows as modified in git, so I haven't committed anything.
<!-- ------ -->
---
OrcaSlicer 2.5.0 is installed and starts correctly under WSLg. I ran it for 25 seconds with no errors, until my timeout stopped it. I didn't open a model or slice anything.

Your earlier install never finished, so the run failed with "not installed". The Flathub remote is system-wide, so the install needed sudo. I installed the stable branch along with the GNOME 50 runtime it requires.

To run it, use `flatpak run com.orcaslicer.OrcaSlicer`. It will probably also be in your Windows Start menu after `wsl --shutdown`, but I haven't checked.

On first launch it found settings from an older install (`io.github.softfever.OrcaSlicer`) and migrated them, so your profiles should carry over.

I added a short note at `docs/kb/apps/orcaslicer.md` and an entry in the KB index. The two background tasks in that notification were only waiters whose work had already finished, so nothing is lost.
<!-- ------ -->
---
Your K1 is on stock firmware and isn't rooted yet, so I'll hold off on configuring a printer backend. Before I pick one, I need to confirm how you want to proceed.

Your options:

1. **Wait until the K1 is rooted with Klipper and Moonraker.** Then I'd configure `PRINTER_TYPE=klipper` on port 7125, using the printer's IP address and an API key only if Moonraker requires one. This is the path the guide documents best for K1-class printers. Until then, the server is already registered with OrcaSlicer, so you can slice from your agent. Printing stays off.
2. **Try the `creality` backend now** against stock firmware. The guide marks it unverified, with no hardware reports, and I don't know whether the stock K1 exposes its API on port 80. I'd try it only if you want to test that.

Which one do you want? Please also send the printer's IP address (or hostname) so I can set it up later. Don't paste any API keys or access codes into chat. I'll tell you where to put those when the time comes.

Once you choose, I'll update the `3d-printer` entry in `~/.claude.json`. You'll also need to restart Claude Code so it loads the server.
<!-- ------ -->
---
Moonraker is running on your K1, Mainsail and Fluidd are being served from it, and the MCP server connects to the printer and reports it as ready.

- **Mainsail:** http://192.168.1.70:8081
- **Fluidd:** http://192.168.1.70:8082
- **Moonraker API:** http://192.168.1.70:7125

Klipper and Moonraker both report ready. The MCP server reads the status, and I didn't start a print or change any temperatures.

**Before you use the web UIs:**
- **No password on the LAN.** Moonraker trusts anything on 192.168.1.0/24 without an API key, so anyone on your network can control the printer. Tell me if you want that tightened.
- **Restart Claude Code** so it loads the updated printer settings.
- **Boot persistence is untested.** A boot hook is installed, but I haven't rebooted the printer. Reboot it when no print is running and check that the web UIs come back.

**Things that differ from a standard install:**
- Moonraker is v0.10.0, not the latest. Newer releases need Python 3.10, and the K1 has 3.8.
- Notifications (apprise) are not installed, because PyYAML won't build on this printer.
- The printer's Python already had the compiled Pillow and file-upload libraries, so I reused them instead of building them.
- The web UIs run on Python's built-in static file server. That's fine for your LAN, but don't expose those ports to the internet.

**Changes on the printer:**
- The config was backed up to `/usr/data/backup-preinstall-20261107/config` before I changed anything. I didn't edit `printer.cfg`.
- Creality firmware updates may remove the boot hook or the web servers.

Details, the control script, and gotchas are in `docs/kb/apps/k1-moonraker.md`, which has no credentials. I haven't committed anything.
<!-- ------ -->
---