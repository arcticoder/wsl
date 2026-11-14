# Print-session runbook (Creality K1 + OrcaSlicer MCP fork + Moonraker)

Audience: Claude, at the start of every printing session. Goal: the user plugs in, says go, and the print starts within minutes, with no troubleshooting or repeated levelling inside the print window. Built from the 2026-11-09 evening session, where about 30 minutes of a 2.5 h window went to a wrong Orca build, a stuck API port, a hot nozzle over the plate, two bed-level runs and two Wi-Fi drops.

Hard rules (user): nothing heats or prints without the user's explicit request in that message; PLA only; filament choice is mine to advise before the window; keep the riser vents closed unless told. Machine and paths: K1 at 192.168.1.70 (Moonraker :7125, Mainsail :8081); Orca Remote API 127.0.0.1:13130; connection notes in `kb/apps/k1-moonraker.md` and `kb/apps/orcaslicer-mcp-fork.md`.

## A. Before the window (no printer, no user needed)

1. Slice every plate that is still to print, save each G-code under a name that says the plate and the parts (`save_gcode filename=balance_p1_posts_....gcode`; files land in `~/.orcaslicer-mcp/gcode/`). Do this the day before, never inside the window.
2. For each file, check the header with grep before trusting it: `; nozzle_temperature = 220`, `nozzle_temperature_initial_layer = 225`, `hot_plate_temp = 60`, `wall_loops`, `sparse_infill_density`, `enable_prime_tower = 0`, supports, and the `EXCLUDE_OBJECT_DEFINE NAME=` lines (they list the parts that are really in the file). The `model_name` field in the save reply is only the first object in the project; ignore it.
3. Work out for each file: print time plus about 12 min (heating and the printer's own levelling), filament mass, and which spool it needs. A job that does not fit the window plus 10 % margin is not offered.
4. Record in the project TODO the window each file fits, and tell the user which spool to have loaded (mass left on the opened spool against the job's mass).
5. Orca must be the fork: `flatpak list | grep -i orca` shows a `user` entry 2.4.2 (the stock 2.5.0 system install has no API). The bundle and its checksum are kept in `~/tools/orcaslicer-mcp/` (a copy outside any session folder) because the per-user install has vanished once; reinstall with `flatpak install --user --bundle <file>` after checking `sha256sum -c SHA256SUMS.txt`. Later fork releases exist (mcp.15+ adds `open_project`/`save_project`); upgrading is optional. Update 2026-11-13/14: the user flatpak is now 2.5.0 (`flatpak run com.orcaslicer.OrcaSlicer`) and its API answers on 13130 (`get_status`: 'OrcaSlicer with the Remote API built in'); a fresh project has stock settings, so reapply the profile in `spurious-force-balance/docs/kb/printing_state.md` (2026-11-13 entry), and `open_project` of a 3MF with its own G-code is refused by the API policy.
6. The Orca project is held in memory only. If the app was closed, reload `balance.3mf` with `load_model`, delete what is printed, rebuild plates, and re-save the G-code. Keep the plate-to-file table in the project TODO so this is quick.

## B. When the user says the printer is on

7. Connectivity first, one command, no assumption: `curl -s -m 3 http://192.168.1.70:7125/server/info`. Silence means the printer or its Wi-Fi is down. Ask for exactly one thing: open `http://192.168.1.70:8081` on the Windows side; if it loads, the problem is WSL's route; if it does not, reconnect Wi-Fi from the touchscreen's network tab (that worked twice on 2026-11-09). Do not run anything hot until it answers.
8. Read-only state, no motion: `print_stats`, `heater_bed`, `extruder`, `idle_timeout`, `server/history/totals` (print hours; see the project TODO for which counter counts), spool in use. Write the print-hours reading in the notes.
9. Do not home or run a bed mesh. `START_PRINT` homes and levels on every print; an extra G28 and `BED_MESH_CALIBRATE` before it cost about 10 min and added nothing on 2026-11-09. Only run a mesh when the user asks a mesh question.
10. Filament loading is the only thing that needs heat before the print. Before any heater command: raise the bed (`G90`, `G1 Z120 F600`), park the head at the rear-left (`G1 X10 Y210 F6000`), only then `M104 S225`. A head left at X110 Y110 dripped a glob onto the plate on 2026-11-09. After loading, let the nozzle idle at the park position and tell the user.
11. Orca health, before slicing or sending anything: `ss -ltnp | grep 13130` and an MCP `get_status`. If the port is open but calls time out, the user toggled Remote API/LAN after the app started (bind "Address already in use" in the log under `~/.var/app/com.orcaslicer.OrcaSlicer/`): ask the user to quit Orca completely (File > Quit), confirm the port is free, relaunch with `flatpak run com.orcaslicer.OrcaSlicer`, and not to open that preference. If `get_status` says not reachable and nothing listens, check it is the fork (step 5), and that Remote API is enabled once in Preferences. Never ask the user to guess which build they ran: check `flatpak ps` and `ss` myself.

## C. Starting and watching the print

12. Only on the user's explicit request in that message. Upload and start without Orca or the MCP: `curl -F file=@<gcode> -F print=true http://192.168.1.70:7125/server/files/upload`; the reply `print_started: true` (the MCP reply field `print_started` is unreliable; confirm with `print_stats.state`).
13. Monitor with the Monitor tool on `print_stats` every 15 s; it catches `complete`, `error`, `paused` and a lost connection. The monitor must be re-armed every 30 min.
14. The Wi-Fi dropped twice on 2026-11-09 (idle at 225 C, and 90 s into a print). The print continues on the printer. A hot idle nozzle with no connection is the safety case: ask the user to look at the printer. Keep the monitor running through reconnection.
15. After the print: read `server/history/totals` and the last job, then update the TODO files and the notes; ask the user to inspect the parts when the bed is cool.

Step 7a (added 2026-11-11): there is no camera and the user is in an office away from the balcony with the door shut. When the user must look at the printer, give a clock time and exactly what to see; they cannot hear the extruder. First-layer look about 8-10 min after the start (corners flat, lines continuous); one mid-print look. Armed Monitor on `print_stats` (15 s poll, offline 10 min alert); re-arm every 30 min. Expect Wi-Fi flaps in the first half hour: a short drop is not an emergency and the print continues.

## D. Things that went wrong and the check that prevents each

| Problem | Check |
|---|---|
| Wrong Orca (stock 2.5.0 has no API) | `flatpak list`, user entry present; `get_status` answers |
| Fork install missing | reinstall from `~/tools/orcaslicer-mcp/` after the checksum |
| API port held after a toggle | do not toggle; quit fully, `ss -ltnp` clear, relaunch |
| Opening the 3MF by hand not needed | `load_model` over the API |
| `set_config` rejected | body is `{"changes": {...}}` |
| Slice at the wrong defaults | slice the pre-saved plates only; headers checked in step 2 |
| Hot nozzle over the plate | step 10 order of commands |
| Double levelling | step 9 |
| Window overrun | step 3 fit table |
| Spool too light | step 4 mass check; plan a pause, swap, resume instead of discarding a part spool |

## E. Added 2026-11-12: first print after a power-on or restart
The fast start (`PRINT_PREPARED`, step 9 and `kb/printing_state.md` in spurious-force-balance) crashes Klipper into Shutdown when no full `START_PRINT` has run since Klipper booted (`CX_PRINT_DRAW_ONE_LINE` has no `extruder_temp`): the first print of a session uses the normal start (upload with `print=true` only; about 8-10 min of start-up). Recover from a shutdown with `curl -X POST http://192.168.1.70:7125/printer/firmware_restart`. `G28` cools the nozzle to about 140 C: do any hand purge after `M109`. Check the `START_PRINT` line of each G-code for `BED_TEMP=` (the header can disagree).
