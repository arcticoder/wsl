# OrcaSlicer MCP fork + orcaslicer-mcp (2026-11-07)
- Stock OrcaSlicer has NO control API. Needs MaxEllis fork build: github.com/MaxEllis/OrcaSlicer releases.
- Installed fork Flatpak per-user (`--user`), ref app/com.orcaslicer.OrcaSlicer/x86_64/master v2.4.2-mcp.14.
  Same app id as stock (system, stable 2.5.0, kept untouched). `flatpak info` resolves to the user install, so
  `flatpak run com.orcaslicer.OrcaSlicer` runs the fork. Remove fork: `flatpak uninstall --user com.orcaslicer.OrcaSlicer`.
  Bundle sha256 verified against release SHA256SUMS.txt (f0fc8584...).
- Fork API: 127.0.0.1:13130, token in ~/.var/app/com.orcaslicer.OrcaSlicer/config/OrcaSlicer/OrcaSlicer.conf
  ("remote_api_token"). The ENABLE switch is only in GUI: Preferences -> Remote API -> Enable Remote API.
  Not enabled as of this note -> port 13130 closed, get_status says "not reachable".
- orcaslicer-mcp 0.1.19 (PyPI) via uvx (uv installed to ~/.local/bin). Registered in ~/.claude.json as mcpServers.orcaslicer
  (env ORCA_API_TOKEN). Backup of ~/.claude.json: ~/ubuntu-24.04-backup/claude.json.bak-orca-*.
- orcaslicer-mcp MCP tools do NOT move the printer or send G-code (by design). Head moves were done directly via Moonraker.

## K1 checks 2026-11-07 (no print, heaters off)
- Heaters target 0, bed 22C, extruder 24C, chamber 22.4C, mcu_temp 32C.
- Homed xyz (G28), QUERY_ENDSTOPS: x/y/z open. Moved G1 X100 Y100 Z20 (no E). Position confirmed.
- filament_sensor (PC15, toolhead? main): filament_detected=false (matches "no filament").
- filament_sensor_2 (nozzle_mcu:PA10): filament_detected=TRUE while no filament loaded -> CHECK (stuck/wiring/misread).
- Probe last_query false; bed_mesh default profile present; all MCUs (main, nozzle, leveling) respond.

## Plates and layout over MCP (observed 2026-11-08, no slice, no save)
- `load_model` of a 3MF with five plates in its metadata loaded one plate (`list_plates` count 1). Most objects were reported `plate: -1` (off every plate), because their saved positions lie outside plate 0's 220 x 220 mm bed. Only objects placed on plate 0 are sliceable.
- `add_plate` creates a plate and moves every off-plate object onto it, so it is a way to gather stray objects, not a blank plate. The new plate's origin is offset by 264 mm in x or y from plate 0.
- `move_object_to_plate` puts the object at a default offset on the target plate. Several objects moved to the same plate land on top of each other until placed explicitly.
- `arrange_plate` is not limited to the current plate in this fork build: it redistributed objects across plates and removed empty ones. Do not use it on a multi-plate project without checking `list_plates` afterwards.
- `check_placement` reports positions relative to bed 0 only, so it flags objects on other plates as overflowing. Use `list_plates` and `list_objects` (their `plate` field) as the source of truth.
- The Orca project lives in memory only. Reloading the 3MF restores the file's contents; nothing is written back unless saved.

## K1 state (2026-11-08)
- The user reports the printer is switched off. Nothing was run against it in this session.

## 2026-11-09 session findings
- The per-user fork install had disappeared (only stock 2.5.0 system install left; stock has no Remote API, so `flatpak run com.orcaslicer.OrcaSlicer` started the wrong app). Reinstalled mcp.14 from the GitHub release (sha256 f0fc8584... verified) with `flatpak install --user --bundle`. Newer releases exist (mcp.16 on 2026-11-09); `open_project` needs mcp.15 or later, so use `load_model` on a 3MF with mcp.14 (it carries per-object settings and brings all 19 objects in; delete the unwanted ones, then `arrange_plate`).
- Toggling Preferences -> Remote API (or "allow LAN access") after the app is running leaves the listening socket held by a WebKit helper process: the log shows `remote-api: failed to bind port 13130: Address already in use`, the port accepts connections but never replies, and every tool call times out ("ui_timeout"). Fix: quit Orca completely (check `ss -ltnp | grep 13130` is empty), relaunch it, do not touch that preference. The saved settings (enabled, token, LAN) persist.
- `set_config` takes `{"changes": {...}}`. `get_config` is huge: avoid it.

## 2026-11-10
- A copy of the mcp.14 bundle and its SHA256SUMS now sits in `~/tools/orcaslicer-mcp/` (outside session folders) because the per-user install vanished once. Reinstall: check the checksum, then `flatpak install --user --bundle <file>`.
- Full session procedure and the list of failures with their checks: `docs/print-session-runbook.md`.
