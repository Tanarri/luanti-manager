# Changelog

All notable changes to this project are documented in this file.

## Unreleased (Stand: 2026-09-28)

### Added

- Added process identity checks for directly started instances. PID files now
  contain the process ID, Linux process start time and boot ID.
- Added support for controlling installed systemd user and system services from
  `start`, `stop`, `restart` and `status`.
- Added validation for instance names and missing world directories.
- Added explicit reporting when UDP port inspection is unavailable.

### Changed

- Replaced TCP port inspection with UDP inspection, matching the Luanti server
  protocol.
- Replaced the root-owned system service template with a systemd user service
  at `etc/systemd/user/luanti@.service`.
- `start` now waits briefly before reporting success and returns an error when
  Luanti exits immediately.
- `check` now evaluates every configured instance and fails when any entry is
  invalid.
- Configuration parsing now trims surrounding whitespace, accepts indented
  comments and reads a final line without a trailing newline.
- `help` no longer creates runtime directories. `status` and `stop` no longer
  require the Luanti executable to be present.
- Consolidated configuration parsing, port inspection, systemd interaction and
  process shutdown logic.
- Expanded the README with requirements, path handling, logging behavior and
  systemd user service installation instructions.

### Fixed

- Fixed occupied Luanti UDP ports being reported as available.
- Fixed `check` returning success when an invalid entry was followed by a valid
  entry.
- Fixed stale PID files being able to target an unrelated process after PID
  reuse. Signals, including `SIGKILL`, are sent only while the stored process
  identity still matches.
- Fixed CLI commands and systemd managing separate process states without
  recognizing each other.
- Fixed successful start messages for processes that terminate immediately.
- Fixed the un-commented field header in the README configuration example.

### Migration from the previous system service

- Stop and disable every old system service instance before enabling the new
  user service, for example:

  ```bash
  sudo systemctl disable --now luanti@voxelibre.service
  ```

- Remove the previous template from `/etc/systemd/system/luanti@.service`, then
  reload the system manager:

  ```bash
  sudo rm /etc/systemd/system/luanti@.service
  sudo systemctl daemon-reload
  ```

- Install the new user service as described in the README and use
  `systemctl --user` to manage it.
- PID files created by previous versions contain only a PID. The new version
  refuses to signal a running process described by such a file because its
  identity cannot be verified. Stop the old process safely before removing the
  obsolete PID file.

### Compatibility notes

- Direct process tracking requires Linux `/proc`.
- UDP port inspection requires `ss`, normally provided by `iproute2`.
- The user service template expects the manager at `~/luanti-manager` and
  Luanti at `~/luanti` unless its paths are changed.
