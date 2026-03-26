# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [1.0.0] - 2026-03-26

### Added
- Multi-distribution support: Ubuntu, Debian, RHEL, CentOS, Rocky Linux, AlmaLinux, Oracle Linux, Fedora, Amazon Linux, SUSE/SLES
- Multi-architecture support: x86_64 and aarch64 (ARM)
- Zabbix Agent 2 installation from official Zabbix 7.0 repositories
- Jinja2-templated configuration with full variable control
- Optional TLS/PSK encryption support
- Optional custom UserParameters deployment
- Optional firewall management (firewalld and ufw)
- Separate Server and ServerActive configuration for Zabbix Proxy setups
- Extra configuration options via key-value dictionary
- Idempotent repository installation (skip if already present)
- Ansible `--check` mode compatibility
- Uninstall support via `zabbix-agent-uninstall` tag
- Molecule testing framework with Docker (Ubuntu 22.04, Debian 12, Rocky Linux 9)
- GitHub Actions CI pipeline (yamllint, ansible-lint, syntax check)
- Fedora-to-RHEL version mapping for repository compatibility
- Automatic DNF Python library bootstrapping on Fedora
- Pre-install cleanup of conflicting repository packages
