# JLCONE (JLCPCB desktop client)
- Installed from deb. 2026-11-13: upgraded 1.0.72 -> 1.0.73 with `apt-get install ./<deb>` (sudo via .env SUDO_PASSWORD).
- Binary /opt/JLCONE/jlcone, alternatives link /usr/bin/jlcone. User wrapper ~/.local/bin/jlcone adds --force-device-scale-factor=1.33 (WSLg renders Electron small).
- Launch for automation: `jlcone --remote-debugging-port=9222 &`. Verify with `curl -s http://127.0.0.1:9222/json/version` (Electron 35.7.5, reports jlcone/1.0.73).
- Launch detached (setsid nohup), not as a foreground tool background task.
