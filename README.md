# ansible-apps_squid_exporter

[![Galaxy Role](https://img.shields.io/badge/galaxy-apps_squid_exporter-purple?style=flat)](https://galaxy.ansible.com/lotusnoir/apps_squid_exporter)
[![Version](https://img.shields.io/github/release/lotusnoir/ansible-apps_squid_exporter.svg)](https://github.com/lotusnoir/ansible-apps_squid_exporter/releases/latest)
[![GitHub repo size](https://img.shields.io/github/repo-size/lotusnoir/ansible-apps_squid_exporter?color=orange&style=flat)](https://galaxy.ansible.com/lotusnoir/apps_squid_exporter)
[![downloads](https://img.shields.io/ansible/role/d/lotusnoir/apps_squid_exporter)](https://galaxy.ansible.com/lotusnoir/apps_squid_exporter)
[![License](https://img.shields.io/badge/license-Apache--2.0-brightgreen?style=flat)](https://opensource.org/licenses/Apache-2.0)

<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

## Description

Deploy [squid_exporter](https://github.com/boynux/squid-exporter) to expose squid metrics to prometheus.

This role deploys squid-exporter (https://github.com/boynux/squid-exporter), a Prometheus exporter exposing Squid proxy metrics. It downloads the pre-built binary release, installs it on the target, and manages it as a systemd service.

## Requirements
- Internet access to github.com from the Ansible controller (default) or target host
- ca-certificates package (installed automatically by the role)

## Role variables

See [variables](/defaults/main.yml) for the full list with raw defaults.

### Binary install

| Variable | Default | Description |
|----------|---------|--------------|
| `squid_exporter_version` | `"1.11.0"` | Version of squid-exporter to install, used to build `squid_exporter_url`. |
| `squid_exporter_url` | `"https://github.com/boynux/squid-exporter/releases/download/v{{ squid_exporter_version }}/squid-exporter-linux-amd64"` | Download URL for the binary. Override entirely if you host your own mirror or need a different architecture (only `linux-amd64` is provided upstream by default). |
| `squid_exporter_install_dir` | `"/usr/local/bin"` | Directory where the `squid-exporter` binary is installed on the target. |
| `squid_exporter_force_install` | `false` | If `true`, re-downloads and redeploys the binary even if one already exists at `squid_exporter_install_dir`. Required to pick up a `squid_exporter_version` bump — see notes below. |
| `squid_exporter_local_dl` | `true` | If `true` (default), the binary is downloaded on the Ansible **controller** (`localhost`) to `/tmp/squid-exporter`, then copied to the target — useful for targets without internet access. If `false`, the target downloads it directly. |

### Systemd / runtime options

| Variable | Default | Description |
|----------|---------|--------------|
| `squid_exporter_squid_host` | `"localhost"` | Hostname/IP of the Squid instance to monitor (`-squid-hostname` flag). Change it if the exporter runs on a different host than Squid itself. |
| `squid_exporter_squid_port` | `"3128"` | Port of the monitored Squid instance (`-squid-port` flag). |
| `squid_exporter_listen_port` | `"9103"` | Port the exporter listens on for Prometheus scraping (`-listen` flag). |
| `squid_exporter_extra_flags` | `"-extractservicetimes=false"` | Extra raw CLI flags appended to `ExecStart` in the systemd unit. Set to `""` to disable. |

### Framework variable (not in defaults/main.yml)

| Variable | Default | Description |
|----------|---------|--------------|
| `success_run_flag` | *(undefined)* | If defined and truthy, writes an idempotency timestamp file to `/var/cache/ansible/<role_name>` after a successful run (Debian/RedHat only). Not required for the role to work. |

### Notes

> Binary presence, not version, drives reinstalls: the install task only checks whether a file already exists at `squid_exporter_install_dir` — bumping `squid_exporter_version` alone does **not** trigger a redeploy. Set `squid_exporter_force_install: true` (at least for one run) whenever you change the version.
>
> With the default `squid_exporter_local_dl: true`, the binary is fetched once on the controller to `/tmp/squid-exporter` per play and then copied to every target in the batch — make sure the controller has outbound internet access and enough space in `/tmp`.
>
> The systemd unit always restarts (`Restart=always`) and reloads automatically (`ExecReload=/bin/kill -HUP $MAINPID`); changes to the unit file trigger the `Restart squid_exporter` handler.
>

## Generated files layout

```
{{ squid_exporter_install_dir }}/squid-exporter        # e.g. /usr/local/bin/squid-exporter
/etc/systemd/system/squid_exporter.service

```

## Tags

| Tag | Scope |
|-----|-------|
| `always` | Entire role: OS support check, OS-specific vars, binary install/download, systemd unit, success flag. No finer-grained tags are defined. |

## Handlers

| Handler | Description |
|---------|--------------|
| `Restart squid_exporter` | Full service restart, triggered when the systemd unit file changes. |

## Examples

Minimal example — installs squid-exporter with default settings, monitoring a local Squid instance on port 3128 and exposing metrics on port 9103.

```yaml
---
- hosts: apps_squid_exporter
  become: true
  become_method: sudo
  gather_facts: true
  roles:
    - role: ansible-apps_squid_exporter # or lotusnoir.base.apps_squid_exporter
```

Monitor a remote Squid instance, pin a specific exporter version with a forced reinstall, download directly on the target (air-gapped controller use case) and change the exporter's listen port.

```yaml
---
- hosts: apps_squid_exporter
  become: true
  become_method: sudo
  gather_facts: true
  roles:
    - role: ansible-apps_squid_exporter # or lotusnoir.base.apps_squid_exporter
vars:
    squid_exporter_version: "1.11.0"
    squid_exporter_force_install: true
    squid_exporter_local_dl: false

    squid_exporter_squid_host: "proxy01.internal.example.com"
    squid_exporter_squid_port: "3128"
    squid_exporter_listen_port: "9103"
    squid_exporter_extra_flags: "-extractservicetimes=true"
```

## License

This project is licensed under Apache License. See [LICENSE](/LICENSE) for more details.

## Author Information

- [Philippe LEAL](https://github.com/lotusnoir)
