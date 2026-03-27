# Ansible Role: Zabbix Agent 2

[![License](https://img.shields.io/github/license/kevinmagni/zabbix-agent2-ansible-role)](LICENSE)
[![CI](https://github.com/kevinmagni/zabbix-agent2-ansible-role/actions/workflows/ci.yml/badge.svg)](https://github.com/kevinmagni/zabbix-agent2-ansible-role/actions/workflows/ci.yml)
[![Ansible Galaxy](https://img.shields.io/badge/galaxy-kevinmagni.zabbix__agent2-blue)](https://galaxy.ansible.com/ui/standalone/roles/kevinmagni/zabbix_agent2/)

An Ansible role that installs and configures **Zabbix Agent 2** from official Zabbix repositories across all major Linux distributions and Windows.

## Supported Platforms

### Linux

| Distribution      | Versions             | Package Manager |
|-------------------|----------------------|-----------------|
| **Ubuntu**        | 20.04, 22.04, 24.04 | APT             |
| **Debian**        | 11, 12               | APT             |
| **RHEL**          | 8, 9, 10             | DNF             |
| **CentOS**        | 8, 9                 | DNF             |
| **Rocky Linux**   | 8, 9, 10             | DNF             |
| **AlmaLinux**     | 8, 9, 10             | DNF             |
| **Oracle Linux**  | 8, 9                 | DNF             |
| **Fedora**        | 30+                  | DNF             |
| **Amazon Linux**  | 2023                 | DNF             |
| **SUSE/SLES**     | 15+                  | Zypper          |

### Windows

| Platform    | Method | Architecture |
|-------------|--------|--------------|
| **Windows** | MSI    | amd64        |

Windows support includes automatic detection and removal of previous installations, MSI-based deployment from the official Zabbix CDN, and template-driven configuration with Windows-native paths.

## Requirements

- Ansible >= 2.10
- Root/sudo access on target hosts (Linux), Administrator on Windows
- `community.general` collection (for SUSE/SLES and UFW firewall support)
- `ansible.posix` collection (for firewalld support)
- `ansible.windows` collection (for Windows targets)

```bash
ansible-galaxy collection install community.general ansible.posix ansible.windows
```

## Installation

### From Ansible Galaxy

```bash
ansible-galaxy install kevinmagni.zabbix_agent2
```

### From GitHub

```bash
ansible-galaxy install git+https://github.com/kevinmagni/zabbix-agent2-ansible-role.git,main
```

## Role Variables

### Required

| Variable | Description | Example |
|----------|-------------|---------|
| `zabbix_agent_server` | Zabbix Server/Proxy address | `"192.168.1.100"` |

### Optional

| Variable | Default | Description |
|----------|---------|-------------|
| `zabbix_agent_version` | `"7.0"` | Zabbix repository version |
| `zabbix_agent_server_active` | _(same as server)_ | ServerActive address |
| `zabbix_agent_hostmetadata` | _(undefined)_ | Host metadata for auto-registration |
| `zabbix_agent_timeout` | `30` | Agent timeout in seconds |
| `zabbix_agent_logfile_size` | `10` | Max log file size in MB |
| `zabbix_agent_allow_key` | `"system.run[*]"` | Allowed item key pattern |
| `zabbix_agent_log_remote_commands` | `true` | Log remote commands execution |
| `zabbix_agent_listen_port` | `10050` | Agent listen port |

### TLS/PSK Encryption

All TLS settings are disabled by default. Works on both Linux and Windows.

| Variable | Default | Description |
|----------|---------|-------------|
| `zabbix_agent_tls_connect` | `""` | Outgoing connection type: `unencrypted`, `psk`, or `cert` |
| `zabbix_agent_tls_accept` | `""` | Incoming connection types |
| `zabbix_agent_tls_psk_identity` | `""` | PSK identity string |
| `zabbix_agent_tls_psk_value` | `""` | PSK secret (hex string) |
| `zabbix_agent_tls_psk_file` | `"/etc/zabbix/zabbix_agent2.psk"` | PSK file path (Linux) |
| `zabbix_agent_tls_ca_file` | `""` | Path to CA certificate |
| `zabbix_agent_tls_cert_file` | `""` | Path to agent certificate |
| `zabbix_agent_tls_key_file` | `""` | Path to agent private key |

### Custom UserParameters

| Variable | Default | Description |
|----------|---------|-------------|
| `zabbix_agent_userparameters` | `[]` | List of UserParameter config files to deploy |

### Extra Configuration

| Variable | Default | Description |
|----------|---------|-------------|
| `zabbix_agent_extra_config` | `{}` | Key-value pairs appended to the config file |

### Firewall Management (Linux only)

| Variable | Default | Description |
|----------|---------|-------------|
| `zabbix_agent_firewall_manage` | `false` | Automatically open agent port in firewalld/ufw |

### Windows Settings

| Variable | Default | Description |
|----------|---------|-------------|
| `zabbix_agent_win_version` | `"7.0.0"` | Full MSI version to download |
| `zabbix_agent_win_download_path` | `C:\Temp` | MSI download location |
| `zabbix_agent_win_install_path` | `C:\Program Files\Zabbix Agent 2` | Installation directory |

On Windows, previous installations are automatically detected via `Get-CimInstance Win32_Product` and removed before installing the new version.

## Usage

### Basic Playbook (Linux)

```yaml
---
- name: Deploy Zabbix Agent 2
  hosts: linux
  become: true
  roles:
    - role: kevinmagni.zabbix_agent2
      vars:
        zabbix_agent_server: "192.168.1.100"
```

### Windows Playbook

```yaml
---
- name: Deploy Zabbix Agent 2 on Windows
  hosts: windows
  roles:
    - role: kevinmagni.zabbix_agent2
      vars:
        zabbix_agent_server: "192.168.1.100"
        zabbix_agent_win_version: "7.0.0"
```

### With PSK Encryption

```yaml
---
- name: Deploy Zabbix Agent 2 with PSK
  hosts: all
  become: true
  roles:
    - role: kevinmagni.zabbix_agent2
      vars:
        zabbix_agent_server: "zabbix.example.com"
        zabbix_agent_tls_connect: "psk"
        zabbix_agent_tls_accept: "psk"
        zabbix_agent_tls_psk_identity: "myhost-psk-01"
        zabbix_agent_tls_psk_value: "abc123def456..."
```

### With Custom UserParameters

```yaml
---
- name: Deploy Zabbix Agent 2 with custom checks
  hosts: all
  become: true
  roles:
    - role: kevinmagni.zabbix_agent2
      vars:
        zabbix_agent_server: "192.168.1.100"
        zabbix_agent_userparameters:
          - name: custom_checks
            params:
              - "UserParameter=custom.cpu.usage,/usr/local/bin/cpu_check.sh"
              - "UserParameter=custom.mem.free,free -m | awk '/Mem/ {print $4}'"
          - name: app_monitoring
            params:
              - "UserParameter=app.status,systemctl is-active myapp"
```

### With Firewall and Extra Config

```yaml
---
- name: Deploy Zabbix Agent 2 with firewall
  hosts: all
  become: true
  roles:
    - role: kevinmagni.zabbix_agent2
      vars:
        zabbix_agent_server: "192.168.1.100"
        zabbix_agent_firewall_manage: true
        zabbix_agent_extra_config:
          BufferSize: 1000
          RefreshActiveChecks: 60
```

### Mixed Linux + Windows

```yaml
---
- name: Deploy Zabbix Agent 2 everywhere
  hosts: all
  become: "{{ 'true' if ansible_facts['os_family'] != 'Windows' else 'false' }}"
  roles:
    - role: kevinmagni.zabbix_agent2
      vars:
        zabbix_agent_server: "zabbix.example.com"
        zabbix_agent_hostmetadata: "auto-registration"
```

## Tags

Run specific parts of the role:

```bash
# Only update configuration
ansible-playbook site.yml --tags conf

# Only manage the service
ansible-playbook site.yml --tags service

# Only deploy UserParameters
ansible-playbook site.yml --tags userparameters

# Only manage firewall
ansible-playbook site.yml --tags firewall

# Uninstall Zabbix Agent 2 completely
ansible-playbook site.yml --tags zabbix-agent-uninstall
```

Available tags: `repo`, `install-agent`, `conf`, `service`, `tls`, `userparameters`, `firewall`, `zabbix-agent-uninstall`

## What This Role Does

1. **Repository setup** - Downloads and installs the official Zabbix repository for the detected OS (idempotent, skips if already present)
2. **Package installation** - Installs `zabbix-agent2` and all available plugins
3. **Configuration** - Deploys the agent config via Jinja2 template (Linux: `/etc/zabbix/zabbix_agent2.conf`, Windows: `C:\Program Files\Zabbix Agent 2\zabbix_agent2.conf`)
4. **TLS/PSK** - Deploys PSK file with correct ownership and permissions (Linux and Windows)
5. **UserParameters** - Deploys custom check files to `zabbix_agent2.d/` (Linux)
6. **Firewall** - Opens the agent port via firewalld (RHEL/SUSE) or ufw (Debian/Ubuntu)
7. **Service management** - Enables and starts the agent service
8. **Uninstall** - Complete removal of packages, configuration, logs, and repository (all platforms)

On Windows, the role additionally handles automatic detection and removal of previous Zabbix Agent 2 versions before installing the new one.

## Directory Structure

```
.
├── defaults/main.yml                # Default variables
├── handlers/main.yml                # Service restart handlers (Linux + Windows)
├── meta/main.yml                    # Galaxy metadata
├── tasks/
│   ├── main.yml                     # OS detection and routing
│   ├── ubuntu.yml                   # Ubuntu
│   ├── debian.yml                   # Debian
│   ├── redhat.yml                   # RHEL
│   ├── centos.yml                   # CentOS
│   ├── rocky_linux.yml              # Rocky Linux
│   ├── alma_linux.yml               # AlmaLinux
│   ├── oracle.yml                   # Oracle Linux
│   ├── fedora.yml                   # Fedora
│   ├── amazon_linux.yml             # Amazon Linux
│   ├── suse.yml                     # SUSE/SLES
│   ├── windows.yml                  # Windows (MSI)
│   ├── configure_tls.yml            # TLS/PSK (Linux)
│   ├── configure_tls_win.yml        # TLS/PSK (Windows)
│   ├── configure_userparameters.yml # Custom UserParameters
│   ├── configure_firewall.yml       # Firewall rules
│   └── uninstall.yml                # Complete removal
├── templates/
│   ├── zabbix_agent2.conf.j2        # Linux configuration
│   └── zabbix_agent2_win.conf.j2    # Windows configuration
├── molecule/                        # Molecule test scenarios
├── tests/                           # Test playbook and inventory
└── examples/                        # Example playbooks
```

## Testing

The CI pipeline runs on every push and pull request using GitHub Actions with [uv](https://docs.astral.sh/uv/) for fast dependency management:

- **Lint** - yamllint + ansible-lint
- **Syntax Check** - `ansible-playbook --syntax-check`
- **Molecule** - Integration tests on Ubuntu 22.04, Debian 12, and Rocky Linux 9 (Docker)

### Run locally

```bash
# Syntax check
ansible-playbook tests/test.yml -i tests/inventory.ini --syntax-check

# Molecule (requires Docker)
pip install molecule 'molecule-plugins[docker]'
molecule test

# Test a specific distro
MOLECULE_DISTRO=rockylinux9 molecule test
```

## License

Apache-2.0

## Author

**Kevin Magni** - [GitHub](https://github.com/kevinmagni)
