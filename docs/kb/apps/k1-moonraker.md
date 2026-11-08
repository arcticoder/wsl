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
