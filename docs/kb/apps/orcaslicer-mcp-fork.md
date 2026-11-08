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
