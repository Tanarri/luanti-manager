# 📦 Luanti Manager (`luantictl`)

A lightweight multi-instance CLI manager for running multiple **Luanti** server worlds on a single host.

Designed for power users, homelabs and dedicated Linux servers.

---

## ✨ Features

- Manage multiple Luanti server instances
- Start / Stop / Restart individual worlds or all at once
- PID-based process tracking
- Per-instance logging
- Port collision detection
- Optional `--gameid` support per world
- Simple configuration file
- No systemd required (but fully compatible)

---

## 📁 Project Structure

luanti-manager/
├── luantictl
├── luanti_instances.conf
├── run/ # PID files (ignored by git)
└── logs/ # Log files (ignored by git)

scss
Code kopieren

Your Luanti installation remains separate:

~/luanti

yaml
Code kopieren

This keeps the official Luanti repository clean.

---

## ⚙️ Configuration

Edit:

luanti_instances.conf

makefile
Code kopieren

Format:

NAME|WORLDREL|PORT|LOGREL|GAMEID

makefile
Code kopieren

Example:

NAME|WORLDREL|PORT|LOGREL|GAMEID
2025_09|worlds/2025_09|30000|logs/2025_09.log|
voxelibre|worlds/voxelibre|30001|logs/voxelibre.log|voxelibre
creative|worlds/creative|30002|logs/creative.log|minetest

yaml
Code kopieren

### Field Description

| Field     | Description |
|-----------|------------|
| NAME      | Instance name |
| WORLDREL  | Path relative to Luanti directory |
| PORT      | Server port |
| LOGREL    | Log file path relative to manager directory |
| GAMEID    | Optional game ID (can be empty) |

---

## 🚀 Usage

### Start all worlds

luantictl start all

### Start a single world

luantictl start voxelibre

### Stop everything

luantictl stop all

### Restart one instance

luantictl restart 2025_09

### Show status

luantictl status

### Follow logs

luantictl tail voxelibre

### Check port usage

luantictl ports

### Validate configuration

luantictl check

---

## 🔧 Installation

```bash
mkdir -p ~/luanti-manager/{run,logs}
chmod +x luantictl
```

Optional: add to PATH

```bash
sudo ln -s ~/luanti-manager/luantictl /usr/local/bin/luantictl
```

## 🌍 Environment Variables (Optional)
You can override default paths:

```
export LUANTI_DIR=~/luanti
export BASE_DIR=~/luanti-manager
export CONF_FILE=~/luanti-manager/luanti_instances.conf
```

##� ��� Git Safety
The following are ignored by .gitignore:

```
run/
logs/
*.pid
```
Runtime data is never committed.

