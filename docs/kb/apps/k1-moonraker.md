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

## First print from the MCP (2026-11-08, plate 1 of the balance frame, 1 h 37 min, completed)
- Read-only state over HTTP works without the MCP: `GET http://192.168.1.70:7125/printer/objects/query?print_stats&heater_bed&extruder&filament_switch_sensor%20filament_sensor_2&bed_mesh&virtual_sdcard`. Gcode commands: `POST /printer/gcode/script?script=G28` (homed in a few seconds) and `BED_MESH_CALIBRATE` (over 2 minutes: the tool call goes to the background).
- `filament_sensor_2` reads detected with nothing loaded and with the spool loaded, so it does not tell the two apart; do not use it as the filament check.
- Bed mesh after the move to the balcony: three runs, the same steady 1.4 mm tilt front to back and almost none left to right. It repeats, so it is mechanical (bed not parallel to the gantry), not dust or noise. The stock `START_PRINT` (gcode_macro.cfg) runs CX_ROUGH_G28, ACCURATE_G28 and CX_PRINT_LEVELING_CALIBRATION itself, so every print re-levels; no separate mesh step is needed before a print.
- The Orca project built from loose STLs does NOT carry `balance.3mf`'s settings: `get_config` showed 2 walls, 15% infill, Generic PLA (bed 55). Apply the requirements with `set_config` (docs section 6 of `print_requirements.md`: 3 walls, 30%, 225/220 C, bed 60, tree supports, first-layer 30 mm/s) and re-slice before sending anything; the first slices of the session (5 h, 1 h 10 min) were at the wrong settings. At the right settings: plate 1 (8 parts) 1 h 36 min 36 g, gantry 3 h 58 min 70 g, plate 0 (9 parts) was 3 h 57 min at the wrong settings, base 3 h 25 min.
- Orca warns on every slice: bed temperature higher than the filament profile allows, and nozzle hardness lower than the profile requires. Both come from using the "Generic PLA @System" filament preset with a 60 C bed; harmless in this print.
- Flow: `save_gcode` then `upload_gcode` with `print=true` and `material=PLA` started the print; the call's `print_started: false` in its reply was wrong (the printer was printing seconds later), so check `print_stats.state`. A Monitor polling `print_stats` every 15 s caught the end (`complete`).
- Per-part mass from the G-code: sum the E moves between `EXCLUDE_OBJECT_START NAME=` and `EXCLUDE_OBJECT_END` times pi*(0.875 mm)^2*1.24 g/cm3 (the part masses sum to about 93% of the slicer's total, the rest is purge and supports). vane_holder: 0.93 g.

## 2026-11-09
- Total print time per Moonraker `server/history/totals`: 5805 s (1.61 h, one job) before tonight; user rule: when it reaches 300 h, ask the user to re-run the printer's equipment self-test.
- Bed mesh re-probed (G28 then BED_MESH_CALIBRATE, no heating): front row +1.14 mm to back row -0.28 mm, still a front-to-back tilt of about 1.4 mm. Which end is physically low is still unconfirmed.
- Do not park the head over the plate and then heat: after G28 and a mesh the nozzle sits at X110 Y110; move it to a corner (G1 X10 Y210) and lower the bed (G1 Z120) before M104, or filament oozes onto the plate.
- The K1 dropped off the network while idle at 225 C (no ping, no ports) and came back after the user reconnected its Wi-Fi on the touchscreen; WSL could reach it again then. A hot idle nozzle with no connection is a safety item: ask the user to check it.
- Upload and start without the MCP: `curl -F file=@x.gcode -F print=true http://192.168.1.70:7125/server/files/upload` returned `print_started: true` and the print began.

## 2026-11-10
- Print hours: Moonraker's `server/history/totals` undercounts (1.61 h, then 2.85 h after the 2026-11-09 evening print). The touchscreen counter showed 131 h on 2026-08-06, and more has been printed since; the touchscreen total is the one the 300 h self-test rule uses, and only the user can read it.
- Pre-sliced plate G-code lives in `~/.orcaslicer-mcp/gcode/` (`balance_p0_post_nn`, `balance_p1_posts_np_pn_pp`, `balance_p2_base`, `balance_p3_gantry`, `balance_p4_strip_guide`). `save_gcode` records `model_name` as the first object in the project, not the plate's parts; read `EXCLUDE_OBJECT_DEFINE` lines in the file instead.
- Orca MCP plate workflow that worked (mcp.14): `delete_object` for printed parts, `load_model` the 3MF, `add_plate(select=false)`, `move_object_to_plate` (lands at the plate centre), `transform_object translate` to space parts, `select_plate`, `slice_and_wait`, `save_gcode`. Plates in `list_plates` are laid out on a grid with 264 mm pitch; object offsets in `list_objects` use global coordinates.
- Session procedure: `print-session-runbook.md` (this folder's parent).

## 2026-11-11
- The K1 has no camera usable from WSL: `server/webcams/list` is empty and ports 8080 and 4408 are closed; the `3d-printer` MCP `camera_snapshot` cannot help (Klipper type has no snapshot URL). Visual checks are the user's, with a clock time each.
- From cold and unhomed, `G1` moves are refused: run `G28` first (safe cold), then `G90`, `G1 Z120 F600`, `G1 X10 Y210 F6000`, then `M104 S225`. Extruding 80 mm at 150-200 mm/min through the hot nozzle at the rear-left drops a strand onto the plate corner: the user lifts it off before the print.
- Upload-and-start with curl worked again (`print_started: true`; `print_stats.state` = `printing`). The first minutes show nozzle 135 °C and bed warming (the printer's own levelling) with `print_duration` 0; G-code `progress` is by file position, not time.
- Wi-Fi: repeated drops 7:19-7:50, none 7:50-8:52 then stable (see the repo's `docs/kb/printing_state.md`). The print continued on the printer through every drop.
- Print hours: `server/history/totals` still 2.85 h before today's gantry (10268 s).

## Bed-mesh reading convention (2026-11-11, from Klipper's `bed_mesh.py`, upstream master)
- `probed_matrix` row 0 is the minimum Y (the front on this printer); rows climb in Y, columns in X. A stored value is the probed Z minus the reference (`z_pos = result[2] - z_offset`); a positive value means the bed surface is lower than the reference there (the toolhead is raised by the value at that point). Klipper's docs do not state either point; the source does. Creality's fork is not checked for changes.
- Reading of 2026-11-10 (front row +1.14 mm, back row -0.28 mm): the front edge is the low end, by about 1.4 mm. The mesh compensates it on every print; no bed-screw step unless a first layer shows a front-to-back squish gradient.
