# Session context block (paste into the next chat session)

```context
Use .env SUDO_PASSWORD for any sudo needs. This is just wsl on my desktop, not a server, safe to work in. Just don't delete my code or installs. Never write to docs/history.md though feel free to read it as needed, that's our raw chat history (i just paste your completion messages into it).
```

## Working state (as of 2026-11-08)
- Repo: ~/Code/sysadmin/wsl (private GitHub arcticoder/wsl, MIT). KB index: docs/kb/README.md.
- OS: Ubuntu 26.04.1 LTS. python3 is 3.14; the 3.12 venvs use /usr/bin/python3.12 (noble build, held).
- Before any apt/dpkg op touching linux-tools/linux-perf: umount /usr/bin/perf, remount after (apt/perf-bind-mount.md).
- Git identity for this repo: Arcticoder <10162808+arcticoder@users.noreply.github.com>.

## Printer (Creality K1, 192.168.1.70)
- Rooted. Klipper + Moonraker v0.10.0 (py3.8 limit), Mainsail :8081, Fluidd :8082, boot hook S60moonraker.
- Printer is currently powered OFF by the user (2026-11-08). The user switches it on at the balcony before any MCP printer test; the user loads filament and adjusts the vents.
- Rules for this printer: never heat the extruder or start a print unless the user explicitly asks in that message.
  Homing and small non-extruding moves were approved.
- Nozzle-side filament sensor (filament_sensor_2, nozzle_mcu PA10) reported "filament detected" with nothing loaded.
  User believes it is stray filament. Not verified. Clear the path and re-check before any real print.
- Printer gcodes folder holds ~150 earlier user prints. Do not bulk delete.

## OrcaSlicer / MCP
- Fork build (MaxEllis mcp.14) installed per-user; `flatpak run com.orcaslicer.OrcaSlicer` runs it. Stock 2.5.0 is system-wide, untouched.
- Remote API is enabled; token in OrcaSlicer.conf ("remote_api_token"). MCP servers: `orcaslicer` (env ORCA_API_TOKEN, ORCA_PRINTER_URL) and `3d-printer` (klipper, SLICER_PROFILE = K1 0.4 nozzle, PLA).
- Verified: STL -> G-code (slice_stl), G-code upload with print=false. Not verified: 3MF project save (no such tool in the MCP build).
- Plate and layout behaviour through MCP is in `apps/orcaslicer-mcp-fork.md` (2026-11-08): `arrange_plate` and `add_plate` move objects between plates.
- orca-slicer wrapper: ~/.local/bin/orca-slicer (passes --filesystem=/tmp).

## Sudo / secrets
- sudo password lives only in .env (SUDO_PASSWORD, space separated). The user rotated the WSL password and put the value back.
- Passwordless sudo was tried and reverted at the user's request. Do not re-add it.
- .env holds CREALITY_* printer SSH credentials too. Never print or commit .env values.

## Printing sessions (2026-11-10)
Follow `../print-session-runbook.md` at the start of every session (pre-sliced G-code, connectivity check, no extra levelling, park before heating, Orca port-lock rule). Touchscreen print hours are the real total (131 h on 2026-08-06).
