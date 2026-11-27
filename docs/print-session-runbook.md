# Print-session runbook (Creality K1 + OrcaSlicer MCP fork + Moonraker)

Audience: Claude, at the start of every printing session. Goal: the user plugs in, says go, and the print starts within minutes, with no troubleshooting or repeated levelling inside the print window. Built from the 2026-11-09 evening session, where about 30 minutes of a 2.5 h window went to a wrong Orca build, a stuck API port, a hot nozzle over the plate, two bed-level runs and two Wi-Fi drops.

Hard rules (user): nothing heats or prints without the user's explicit request in that message; PLA only; filament choice is mine to advise before the window; keep the riser vents closed unless told. Machine and paths: K1 at 192.168.1.70 (Moonraker :7125, Mainsail :8081); Orca Remote API 127.0.0.1:13130; connection notes in `kb/apps/k1-moonraker.md` and `kb/apps/orcaslicer-mcp-fork.md`.

## A. Before the window (no printer, no user needed)

1. Slice every plate that is still to print, save each G-code under a name that says the plate and the parts (`save_gcode filename=balance_p1_posts_....gcode`; files land in `~/.orcaslicer-mcp/gcode/`). Do this the day before, never inside the window.
2. For each file, check the header with grep before trusting it: `; nozzle_temperature = 220`, `nozzle_temperature_initial_layer = 225`, `hot_plate_temp = 60`, `wall_loops`, `sparse_infill_density`, `enable_prime_tower = 0`, supports, and the `EXCLUDE_OBJECT_DEFINE NAME=` lines (they list the parts that are really in the file). The `model_name` field in the save reply is only the first object in the project; ignore it.
2b. Added 2026-11-21: compare the `EXCLUDE_OBJECT_DEFINE` polygons of the file: objects meant to be separate must not overlap (the two calibration riders came off joined at a corner), keep at least 3 mm apart; and name each object of the plate (and which STL/3MF previews it) in the message to the user.
3. Work out for each file: print time plus about 12 min (heating and the printer's own levelling), filament mass, and which spool it needs. A job that does not fit the window plus 10 % margin is not offered.
4. Record in the project TODO the window each file fits, and tell the user which spool to have loaded (mass left on the opened spool against the job's mass).
5. Orca must be the fork: `flatpak list | grep -i orca` shows a `user` entry 2.4.2 (the stock 2.5.0 system install has no API). The bundle and its checksum are kept in `~/tools/orcaslicer-mcp/` (a copy outside any session folder) because the per-user install has vanished once; reinstall with `flatpak install --user --bundle <file>` after checking `sha256sum -c SHA256SUMS.txt`. Later fork releases exist (mcp.15+ adds `open_project`/`save_project`); upgrading is optional. Update 2026-11-13/14: the user flatpak is now 2.5.0 (`flatpak run com.orcaslicer.OrcaSlicer`) and its API answers on 13130 (`get_status`: 'OrcaSlicer with the Remote API built in'); a fresh project has stock settings, so reapply the profile in `open-torsion-balance/docs/kb/printing_state.md` (2026-11-13 entry), and `open_project` of a 3MF with its own G-code is refused by the API policy.
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
The fast start (`PRINT_PREPARED`, step 9 and `kb/printing_state.md` in open-torsion-balance) crashes Klipper into Shutdown when no full `START_PRINT` has run since Klipper booted (`CX_PRINT_DRAW_ONE_LINE` has no `extruder_temp`): the first print of a session uses the normal start (upload with `print=true` only; about 8-10 min of start-up). Recover from a shutdown with `curl -X POST http://192.168.1.70:7125/printer/firmware_restart`. `G28` cools the nozzle to about 140 C: do any hand purge after `M109`. Check the `START_PRINT` line of each G-code for `BED_TEMP=` (the header can disagree).

## F. Added 2026-11-14 (first full session run by the runbook)
- Before the window: verify each G-code's `START_PRINT ... BED_TEMP=` and `EXCLUDE_OBJECT_DEFINE` lines (done for p7, p8, p10), and name the spool (bagged on a roller; the strand feeds through the bag).
- Loading: park the head at about X200 Y110, Z120 (not the rear-left corner), `M104 S225`, the user hand-feeds, then `M83`, `G1 E40 F150`, `G1 E40 F200`; the heating request counts when the user says to decide or to go in that message.
- Starting: the first print uses the normal start (upload with `print=true`); later prints in the same boot use the fast start sequence in `kb/apps/k1-moonraker.md`. Start the monitor right after the upload; give the first-layer look time (about 8-10 min after start on the normal start, 2-6 min on the fast start) and one mid-print look.
- After a print: confirm both heater targets read 0 (the printer turns them off), report the bed temperature and the time to under 40 C (about 2.5 C/min), and only then ask for the plate to be cleared and for a go.
- Monitor notices that arrive as system events are not the user's messages: the first line after a re-arm is a state read.

## G. Added 2026-11-14: Orca MCP preflight (before blaming a dialog)
`get_status` timing out is usually a process problem, not a window dialog: more than one `uvx orcaslicer-mcp` bridge, or two Orca instances on port 13130. Preflight at the start of any slicing: `ss -ltn | grep 1313` (one listener on 13130), `pgrep -af orcaslicer` (one Orca, bridges at most one or two), `get_status` answers within a few seconds. Fix: kill the extra bridges (they restart on the next call); if Orca itself is doubled or the port is stale, quit Orca fully and relaunch it with `nohup flatpak run com.orcaslicer.OrcaSlicer`. Do not toggle Preferences > Remote API while it runs. Only after these checks ask the user to look for a dialog. Orca's saved preferences hold `remote_api_enabled`, port and token across restarts; nothing in the app needs changing for this. `new_project` needs `if_unsaved: "discard"` when the current project is dirty and keeps the print/filament edits.

## H. Added 2026-11-22
- A lifted-and-replaced plate needs a warm re-mesh before the fast start (see `kb/apps/k1-moonraker.md`, session of 2026-11-21/22); compare the mesh columns: one column off by more than about 0.5 mm from its neighbours means reseat the plate.
- Park the head at the rear-left (X10 Y210) before any hand-load push or test extrusion; the head at X200 Y110 drips onto the plate.
- Do not offer a plate whose start (cool-down, re-mesh, heat) leaves under 10 % margin before the window ends; the cool-down from 58 C to under 40 C takes about 8 minutes.
- Slicing walls with a recessed face: lay the flat (outside) face down; check `bottom_surface` about 20 % of time and no `bridge` role in `slice_and_wait` before saving.

## I. Added 2026-11-25 (first print after a power-on, then a second plate on the same boot)
- Load sequence that worked with the spool already threaded and the black strand already out of the extruder: `G28`, `G90`, `G1 Z120 F600`, `G1 X200 Y110 F6000`, `M104 S225` (225 C in about 1 min 20 s); the user seats and locks the strand at 225 C; then `M83`, `G1 E40 F150`, `G1 E40 F200` (80 mm). With an empty extruder there was nothing to clear.
- Clock times: the normal start takes about 8 min from the upload to the first layer (upload 19:32:38, first layer about 19:41). Give the first-layer look as upload + 9-11 min, not + 1-2. The fast start (after a warm re-mesh, `M104 S225`, `M190 S60`, `M109 S225`, `PRINT_PREPARED`, upload) reached extrusion in about 1 min: first-layer look at upload + 3-5 min.
- Warm re-mesh after the user lifted and replaced the plate: bed to 60, `G28`, `M190 S60`, 60 s soak, `BED_MESH_CALIBRATE`; 20:52-20:57 took 4.5 min of mesh; accept spread about 1.5 mm (front-to-back tilt) with column means within about 0.2 mm; then start without another `G28`.
- A long G-code posted through `/printer/gcode/script` blocks the HTTP call: use `curl -m 400` in a background shell (the harness blocks foreground `sleep` and moves long commands to the background) and read the output file when the completion notice arrives. `G28` posted with `-m 60` returned an empty body but homed fine: check `homed_axes`.
- Monitor on `print_stats` (15 s poll, 30 min cap) must be re-armed twice for a 63 min print; its first event after each re-arm is a state read, not news. Do not claim notes are written before they are.
- Two plates in one window worked: `px_a` (63 min, normal start) then `px_b` (43 min, fast start); the 10 % margin rule decided it, with start slip beyond 21:10 as the drop condition.
