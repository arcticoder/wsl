# sudo / .env
- User authorised using SUDO_PASSWORD from repo-root `.env` for sudo.
- .env format is `SUDO_PASSWORD <value>` (SPACE separated, not `=`). `source .env` FAILS
  ("SUDO_PASSWORD: command not found"). Parse: `PW=$(awk '$1=="SUDO_PASSWORD"{print $2}' .env)`
  then `echo "$PW" | sudo -S -p '' <cmd>`.
- Never print/log the value or copy it into docs. Check .env is gitignored before committing anything.
- Env quirk: `systemctl is-system-running` = degraded (normal here).
