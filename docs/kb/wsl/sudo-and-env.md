# sudo (updated 2026-11-07)
- sudo is passwordless for echo_: /etc/sudoers.d/90-echo-nopasswd -> `echo_ ALL=(ALL) NOPASSWD: ALL` (0440, visudo-checked).
- SUDO_PASSWORD was REMOVED from repo .env at user request. Do not re-add it. Use `sudo -n <cmd>` (no prompt).
- Tradeoff: any process running as echo_ can become root. Accepted by user for this personal WSL box.
  To narrow later: replace the rule with a command list (e.g. apt, dpkg, flatpak, mount, umount).
- .env still holds CREALITY_* (printer SSH root creds). Only that file is gitignored; keep it that way.
