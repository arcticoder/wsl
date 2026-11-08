# sudo / .env
- sudo uses SUDO_PASSWORD from repo-root .env (user's WSL password; user rotated it 2026-11-07 and put it back).
- .env format is `SUDO_PASSWORD <value>` (space separated, not `=`). `source .env` FAILS. Parse:
  `PW=$(awk '$1=="SUDO_PASSWORD"{print $2}' .env)` then `echo "$PW" | sudo -S -p '' <cmd>`.
- Passwordless sudo was tried 2026-11-07 and REVERTED at user request. No /etc/sudoers.d/90-echo-nopasswd.
- Never print/log the value or copy it into docs. .env is gitignored; user plans to make repo public, keep it that way.
- Env quirk: `systemctl is-system-running` = degraded (normal here).
