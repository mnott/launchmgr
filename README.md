# launchmgr

# launchmgr


A systemd-like service launcher for macOS login items.
Launches apps in configurable tiers with delays after boot.

## Overview

`launchmgr` manages macOS application startup by organizing apps into tiers
with configurable delays. A LaunchAgent runs `launchmgr start` at login,
which launches apps tier by tier with absolute delays from boot time.

## Configuration

Config file: `~/.config/launchmgr/config.yaml`
Log file: `~/.config/launchmgr/launch.log`

### Config Structure

```yaml
tiers:
  1:
    delay: 30
    name: "Menu Bar Apps"
  2:
    delay: 120
    name: "Sync & Containers"
  3:
    delay: 300
    name: "Heavy Sync"
services:
  myapp: {app: "MyApp", tier: 1, enabled: true}
```

Tier `delay` values are **absolute seconds from launch start**:
- Tier 1 starts after 30s
- Tier 2 starts after 120s (90s after tier 1)
- Tier 3 starts after 300s (180s after tier 2)

## Commands

### start

Launch services with tier delays.

```
launchmgr start [--tier N] [--service NAME] [--force] [--no-delay]
```

- No args: run all enabled tiers sequentially with configured delays
- `--tier N`: start only that tier's services immediately (no delay)
- `--service NAME`: start one service immediately
- `--force`: bypass per-boot lock file
- `--no-delay`: skip all tier delays for a full run

A per-boot lock file at `/tmp/launchmgr-{boot_time}.lock` prevents double-runs
on the same boot. Only the full start (no --tier/--service) creates/checks it.

### stop

Kill a running service by name.

```
launchmgr stop SERVICE
```

Finds the PID via `pgrep` and sends SIGTERM.

### status

Show a Rich table with columns: Tier | Service | App | Enabled | Running | PID | Delay

```
launchmgr status
```

Also shows lock file status at the bottom.

### list

List services grouped by tier with colored status dots.

```
launchmgr list
```

Green dot = enabled, red dot = disabled.

### enable

Enable a service in the config.

```
launchmgr enable SERVICE
```

### disable

Disable a service in the config.

```
launchmgr disable SERVICE
```

### add

Add a new service to the config.

```
launchmgr add NAME --tier N --app "App Name"
```

Creates the tier if it does not exist (default delay = tier_num * 60).

### remove

Remove a service from the config.

```
launchmgr remove SERVICE
```

### logs

Show the launch log.

```
launchmgr logs [--tail N] [--follow]
```

Default: last 50 lines. `--follow` streams new lines with `tail -f`.

### config

Print config path, log path, and current config contents.

```
launchmgr config
```

### doc

Re-generate this documentation as markdown.

```
launchmgr doc [--title TITLE] [--toc]
```

### install

Install launchmgr to the system (run from the dev directory).

```
cd ~/Cloud/Development/scripts/dev.launchmgr
./launchmgr install
```

Copies the binary to `/usr/local/bin/`, the LaunchAgent plist to
`~/Library/LaunchAgents/`, and creates the default config if none exists.

## Installation

### Prerequisites

```bash
pip install -r requirements.txt
# or
pip install typer rich pyyaml doc2md
```

### Install

```bash
git clone <repo-url> dev.launchmgr
cd dev.launchmgr
./launchmgr install
```

This installs:

| File | Destination |
|------|-------------|
| `launchmgr` | `/usr/local/bin/launchmgr` |
| `org.mnsoft.launchmgr.plist` | `~/Library/LaunchAgents/org.mnsoft.launchmgr.plist` |
| `config.yaml` | `~/.config/launchmgr/config.yaml` (only if missing) |

The LaunchAgent is loaded automatically and will run `launchmgr start`
at every login.
