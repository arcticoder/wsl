# OrcaSlicer (flatpak)
- User's earlier attempt (`flatpak install flathub com.orcaslicer.OrcaSlicer`, then run) never completed:
  run failed with "app/com.orcaslicer.OrcaSlicer/x86_64/master not installed".
- Fixed 2026-11-07: `sudo flatpak install -y --noninteractive flathub com.orcaslicer.OrcaSlicer`
  (flathub remote is *system* scope -> needs sudo; branch is `stable`, v2.5.0, runtime org.gnome.Platform 50).
  Ran detached via setsid with `echo $PW | sudo -S`. Installed OK (~266MB download).
- Verified: `flatpak run com.orcaslicer.OrcaSlicer` stays up under WSLg (DISPLAY=:0, WAYLAND=wayland-0), no errors.
- It auto-migrated config from legacy app id io.github.softfever.OrcaSlicer (~/.var/app/...) = user had used it before.
- `flatpak search orca` returns nothing (appstream cache), use `flatpak remote-info flathub <id>` to check IDs.
- Other flatpaks present: Bazaar (store UI), GNOME Platform 51, Mesa.
