# systemdcustom

Personal collection of systemd **user** units. Each subdirectory is one
self-contained job (unit + timer + any helpers). Files are kept here in
git and symlinked into `~/.config/systemd/user/` so `git pull` is enough
to pick up upstream changes — no re-copying.

## Jobs

- **syncp/** — KeePass vault backup pipeline (Borg → cloud remote), wrapped
  by `syncp.timer` and an `OnFailure=` notifier (`syncp-failure.service`).
  See the header comment in `syncp/syncp.service` for prerequisites
  (`~/.local/bin/scripts/syncp.py`, `~/.secrets/borg.env`).

## Install

The repo is intended to live at `~/systemdcustom`. The unit files
reference `%h/...` paths that assume that location.

```sh
# 1. Clone into the expected path
git clone <repo-url> ~/systemdcustom

# 2. Make sure the user units directory exists
mkdir -p ~/.config/systemd/user

# 3. Symlink each unit you want to enable. Example for syncp:
ln -s ~/systemdcustom/syncp/syncp.service         ~/.config/systemd/user/
ln -s ~/systemdcustom/syncp/syncp.timer           ~/.config/systemd/user/
ln -s ~/systemdcustom/syncp/syncp-failure.service ~/.config/systemd/user/

# 4. Reload and enable
systemctl --user daemon-reload
systemctl --user enable --now syncp.timer
```

To install everything at once:

```sh
ln -s ~/systemdcustom/*/*.service ~/systemdcustom/*/*.timer ~/.config/systemd/user/
systemctl --user daemon-reload
```

Check per-job header comments before enabling — some units need secrets
or scripts that aren't in this repo.

## Updating

```sh
cd ~/systemdcustom && git pull
systemctl --user daemon-reload
```

Because the units are symlinks, the pull is the update. Restart any
affected timers if their schedule changed.

## Removing a unit

```sh
systemctl --user disable --now syncp.timer
rm ~/.config/systemd/user/syncp.{service,timer}
rm ~/.config/systemd/user/syncp-failure.service
systemctl --user daemon-reload
```

## Useful commands

```sh
systemctl --user list-timers           # see when each timer next fires
systemctl --user status syncp.timer    # inspect a unit
journalctl --user -u syncp.service -e  # tail logs for one unit
systemctl --user start syncp.service   # run a oneshot job by hand
```

## License

Dual-licensed under [CC BY-NC-SA 4.0](LICENSE-CC-BY-NC-SA) and
[PolyForm Noncommercial 1.0.0](LICENSE-POLYFORM). Free for personal,
educational, and charitable use; commercial use requires a separate
license — contact me.
