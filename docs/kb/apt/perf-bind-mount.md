# apt full-upgrade failing: /usr/bin/perf "Invalid cross-device link"
Symptom: `linux-tools-common` unpack fails:
  unable to make backup link of './usr/bin/perf' ... Invalid cross-device link
Cause: the WSL2 kernel (6.18.x-microsoft-standard-WSL2) bind-mounts its own perf
  (/dev/sdb[/<kver>/perf/bin/perf], ro) over /usr/bin/perf (source also at
  /usr/lib/linux-tools/<kver>/bin/perf). dpkg can't hardlink/replace a mountpoint.
Fix (non-destructive, done 2026-11-07):
  sudo umount /usr/bin/perf
  sudo dpkg --configure -a; sudo apt full-upgrade -y
  sudo mount --bind /usr/lib/linux-tools/<kver>/bin/perf /usr/bin/perf   # restore
Mount is re-created by WSL on restart anyway. Expect recurrence whenever linux-tools-common
  (or anything owning /usr/bin/perf) updates -> repeat the same steps.
Other apt notes: `apt update` was fine; third-party repos present: cuda-wsl, github-cli,
  kicad-9.0 PPA, deadsnakes PPA, cockpit-files PPA, nvidia-container-toolkit, webmin, zotero.
