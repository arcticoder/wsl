# Creality K1 (K1-FA35, 192.168.1.70) -> Moonraker + Mainsail/Fluidd (2026-11-07)
Creds: CREALITY_IP_ADDRESS / CREALITY_ROOT_USERNAME / CREALITY_ROOT_PASSWORD in repo .env (value-only; never print).
SSH: root, password auth only. Use SSH_ASKPASS helper (no sshpass installed). Pipe stdin for file uploads
  (don't redirect </dev/null when piping data).

## Printer facts
- Creality Buildroot 2020.02, MIPS, kernel 4.4, Python 3.8.2 (/usr/bin/python3); Creality klippy at /usr/share/klippy-env.
- Klipper API socket: /tmp/klippy_uds. Klipper cfg: /usr/data/printer_data/config/printer.cfg
- /usr/data = mmc 6.5G (persistent). / = overlay with upper on /overlay (mmcblk0p9, 97MB, persistent). Keep big stuff in /usr/data.
- No compiler on the printer; GitHub unreachable from printer; PyPI reachable.
- Port 80 + 8000/9998/9999 used by Creality's own UI/services.

## What's installed
- Moonraker v0.10.0 (last release supporting py3.8; v0.11+ needs >=3.10). Source: /usr/data/moonraker
- venv: /usr/data/moonraker-env (created with --system-site-packages to reuse Creality's compiled PIL 7.0.0, streaming_form_data 1.8.1, pyserial 3.4)
- Extra pip pkgs needed (built --no-build-isolation): MarkupSafe 2.1.5, dbus-fast 2.24.4, libnacl 2.1.0 (uses /usr/lib/libsodium.so)
- Skipped: apprise (PyYAML won't build here) -> notifications unavailable.
- Config: /usr/data/printer_data/config/moonraker.conf (trusted_clients 192.168.1.0/24 = NO API KEY on LAN)
- Mainsail: http://192.168.1.70:8081  (python http.server, /usr/data/www/mainsail)
- Fluidd:   http://192.168.1.70:8082  (python http.server, /usr/data/www/fluidd)
- Control: /usr/data/printer_data/moonraker-ctl.sh {start|stop|status}
- Boot hook: /etc/init.d/S60moonraker -> moonraker-ctl.sh start (boot NOT yet tested by reboot)
- Pre-install backup of printer config: /usr/data/backup-preinstall-20261107/config

## Gotchas
- Moonraker "pip recovery" fires on any missing module and tries to pip-install full requirements; it fails
  on MIPS. Fix = install the missing module manually into the venv, then restart.
- Moonraker logs append; old tracebacks persist in logs/moonraker.log (look at tail).
- `pgrep -f moonraker.py` matches the ssh shell itself; use `ps | grep "[m]oonraker/moonraker.py"`.

## MCP side
- ~/.claude.json mcpServers["3d-printer"]: PRINTER_TYPE=klipper, PRINTER_HOST=192.168.1.70, PRINTER_PORT=7125,
  SLICER_TYPE=orcaslicer, SLICER_PATH=~/.local/bin/orca-slicer (wrapper -> flatpak OrcaSlicer).
- Verified get_printer_status returns state ready.
- Print/temperature tools NOT tested (by design).

## Follow-up 2026-11-07 (after power cycle)
- Printer hung/needed power cycle at least twice (moonraker.log "Unsafe Shutdown Count" 1->3). Cause not found in logs.
  Kernel logged "Possible SYN flooding on port 8082": python http.server had listen backlog 5.
- Fix: /usr/data/printer_data/serve.py = ThreadingHTTPServer, request_queue_size=64. ctl script now uses it.
- Mainsail: /usr/data/www/mainsail/config.json "port" null -> 7125 (Mainsail otherwise talks to the page port 8081).
- Moonraker: [update_manager] enable_system_updates: False -> removes buildroot warning; PolKit warning also gone.
- Verified: 7125/8081/8082 = 200, 40-way concurrent burst OK, /websocket 101 from Mainsail origin, MCP get_printer_status ready.
- Boot hook /etc/init.d/S60moonraker confirmed working (servers came back after power cycle).
- Python upgrade: NOT recommended. Printer Python is Creality's bundled 3.8 (klippy-env + system), no compiler,
  GitHub unreachable, rootfs 97MB. Moonraker v0.10.0 works on 3.8; newer Moonraker needs 3.10.

## Pre-power-down prep (2026-11-07)
- MCP env: 3d-printer gets SLICER_PROFILE = K1 machine;process|PLA (Creality K1 0.4 nozzle, 0.16mm Optimal, Creality Generic PLA @K1).
  orcaslicer gets ORCA_PRINTER_URL=http://192.168.1.70:7125, ORCA_PRINTER_ID=creality-k1. Restart Claude Code to load.
- Tested STL -> G-code with 3d-printer slice_stl on a 20 mm cube (slicer = fork CLI via wrapper). Wrapper now passes
  --filesystem=/tmp so the sandboxed slicer can write MCP output dirs in host /tmp.
  confirm_temperatures: extruder 220, bed 55 (PLA). Output ~291 KB.
- Uploaded with upload_gcode print=false -> printed nothing. Test file deleted from printer afterwards.
- Printer gcodes folder holds ~150 prior jobs; DO NOT bulk delete.
- orcaslicer MCP get_printer_status works via ORCA_PRINTER_URL (idle, homed xyz).
- load_model works in fork. save_project NOT available: orcaslicer-mcp 0.1.19 has no such tool and fork mcp.14 lacks
  project-file tools (README says mcp.15). So the STL->3MF project step is not automated yet; the STL->G-code path does not need it.
- Test object removed from fork plate (delete_object takes object_id).
- Stray-filament assumption for filament_sensor_2 (nozzle_mcu PA10) per user. Not verified physically.
- Background sensor logger stopped.
