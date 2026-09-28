# 📦 Luanti Manager (`luantictl`)

A CLI manager for running multiple **Luanti** server worlds on one Linux host.

Designed for power users, homelabs and dedicated Linux servers.

---

## ✨ Features

- Manage multiple Luanti server instances
- Start / Stop / Restart individual worlds or all at once
- Process tracking using PID, start time and boot ID
- Per-instance log files for directly started servers
- UDP port collision detection
- Optional `--gameid` support per world
- Simple configuration file
- Optional systemd user service

---

## 📁 Project Structure

```
luanti-manager/
├── luantictl
├── luanti_instances.conf
├── etc/systemd/user/luanti@.service
├── run/          # PID files (ignored by git)
└── logs/         # Log files (ignored by git)
```

By default, the executable is expected at `~/luanti/bin/luanti`. The Luanti
installation and worlds remain separate from this repository:

```
~/luanti
```

This keeps the official Luanti repository clean.

The script requires Linux with Bash, `/proc`, `ss` (from `iproute2`) and `awk`.
The systemd service is optional. `start` and `run` stop with an error if `ss`
cannot check the UDP port.

---

## ⚙️  Configuration

Edit:
```
luanti_instances.conf
```
Format:
```
# NAME|WORLDREL|PORT|LOGREL|GAMEID
```

Example:
```
# NAME|WORLDREL|PORT|LOGREL|GAMEID
2025_09|worlds/2025_09|30000|logs/2025_09.log|
voxelibre|worlds/voxelibre|30001|logs/voxelibre.log|voxelibre
creative|worlds/creative|30002|logs/creative.log|minetest
```

### Field Description

| Field     | Description |
|-----------|------------|
| NAME      | Instance name; letters, digits, `_`, `.` and `-` only |
| WORLDREL  | World path relative to `LUANTI_DIR` |
| PORT      | UDP server port, from 1 to 65535 |
| LOGREL    | Log file path relative to `BASE_DIR` for direct starts |
| GAMEID    | Optional game ID (can be empty) |

Lines beginning with `#` are ignored. `./luantictl check` validates the fields
and checks that each world directory exists. It does not check for duplicate
names or ports.

---

## 🚀 Usage

From the repository directory:

```bash
./luantictl check
./luantictl start voxelibre
./luantictl start all
./luantictl status
./luantictl ports
./luantictl tail voxelibre
./luantictl restart 2025_09
./luantictl stop all
```

`ports` and the `listen` field in `status` report whether any process has the
configured UDP port open; they do not identify which process owns it. `?`
means the port could not be checked. `tail`
reads the log file from a direct start. For a systemd service, use `journalctl`
as shown below. `run <name>` runs Luanti in the foreground for systemd.

---

## 🔧 Installation

By default, `luantictl` expects this repository at `~/luanti-manager`. From the
root of a clone at another location, create a symlink and ensure the script is
executable:

```bash
ln -s "$(pwd)" "$HOME/luanti-manager"
chmod +x luantictl
```

Skip the symlink command if the repository is already at `~/luanti-manager`.
A direct start creates the `run/` and `logs/` directories automatically.
If Luanti is installed elsewhere, set `LUANTI_DIR` for direct commands and
adjust the service template before installing it.

Optionally add the command to your `PATH`:

```bash
sudo ln -s ~/luanti-manager/luantictl /usr/local/bin/luantictl
```

## Install as a systemd user service

The template runs as your user, without `root`. It expects the repository at
`~/luanti-manager` (or the symlink above) and Luanti at `~/luanti`. Edit its
paths before installation if these locations differ.

```bash
mkdir -p "$HOME/.config/systemd/user"
ln -s "$HOME/luanti-manager/etc/systemd/user/luanti@.service" "$HOME/.config/systemd/user/luanti@.service"
systemctl --user daemon-reload
```

Start a configured world and enable it for future user sessions:

```bash
systemctl --user enable --now luanti@voxelibre.service
```

The user service normally starts when you log in. To start it before login,
enable lingering with `sudo loginctl enable-linger "$USER"`.

Check status of the service:

```bash
systemctl --user status luanti@voxelibre.service
# or live logs
journalctl --user -u luanti@voxelibre.service -n 50
```

`luantictl start`, `stop`, `restart` and `status` use an installed systemd
service for that instance. Without one, `luantictl` starts and tracks the
process itself. A direct start writes to `logs/`; a systemd start writes to the
journal. The systemd `run` mode does not create a PID file.

PID files created by older versions contain only a PID. A running process with
such a file cannot be safely identified, so `luantictl stop` will refuse to
signal it. Stop that process through its existing service or after checking it
manually, then remove the obsolete PID file in `run/`.

If you installed the previous system-wide template under `/etc/systemd/system`,
disable its instances and remove that template before enabling the user service
for the same worlds. Otherwise, both services could try to start the same world.

## 🌍 Environment Variables (Optional)

For direct commands, you can override the default paths in your shell:

```bash
export LUANTI_DIR=~/luanti
export BASE_DIR=~/luanti-manager
export CONF_FILE=~/luanti-manager/luanti_instances.conf
```

Shell exports do not change an installed systemd user service. Adjust
`Environment=` and `ExecStart=` in its template or use a systemd override, then
run `systemctl --user daemon-reload`.

## 🔒 Git Safety
The following are ignored by .gitignore:

```
run/
logs/
*.pid
```
Runtime data is never committed.
