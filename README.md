# Ansible Role: Zabbix Agent 2

[![License](https://img.shields.io/github/license/kevinmagni/zabbix-agent2-ansible-role)](LICENSE)
[![CI](https://github.com/kevinmagni/zabbix-agent2-ansible-role/actions/workflows/ci.yml/badge.svg)](https://github.com/kevinmagni/zabbix-agent2-ansible-role/actions/workflows/ci.yml)
[![Ansible Galaxy](https://img.shields.io/badge/galaxy-kevinmagni.zabbix__agent2-blue)](https://galaxy.ansible.com/ui/standalone/roles/kevinmagni/zabbix_agent2/)

An Ansible role that installs and configures **Zabbix Agent 2** from official Zabbix repositories across all major Linux distributions.

## Supported Platforms

| Distribution      | Versions         | Package Manager | Architectures     |
|-------------------|------------------|-----------------|-------------------|
| **Ubuntu**        | 20.04, 22.04, 24.04 | APT         | x86_64, aarch64   |
| **Debian**        | 11, 12           | APT             | x86_64, aarch64   |
| **RHEL**          | 8, 9, 10         | DNF             | x86_64, aarch64   |
| **CentOS**        | 8, 9             | DNF             | x86_64, aarch64   |
| **Rocky Linux**   | 8, 9, 10         | DNF             | x86_64, aarch64   |
| **AlmaLinux**     | 8, 9, 10         | DNF             | x86_64, aarch64   |
| **Oracle Linux**  | 8, 9             | DNF             | x86_64, aarch64   |
| **Fedora**        | 30+              | DNF             | x86_64, aarch64   |
| **Amazon Linux**  | 2023             | DNF             | x86_64, aarch64   |
| **SUSE/SLES**     | 15+              | Zypper          | x86_64, aarch64   |

## Requirements

- Ansible >= 2.10
- Root/sudo access on target hosts
- `community.general` collection (for SUSE/SLES and UFW firewall support)
- `ansible.posix` collection (for firewalld support, only if `zabbix_agent_firewall_manage: true`)

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
| `zabbix_agent_server_active` | _(same as server)_ | ServerActive address (useful with Zabbix Proxy) |
| `zabbix_agent_hostmetadata` | _(undefined)_ | Host metadata for auto-registration |
| `zabbix_agent_timeout` | `30` | Agent timeout in seconds |
| `zabbix_agent_logfile_size` | `10` | Max log file size in MB |
| `zabbix_agent_allow_key` | `"system.run[*]"` | Allowed item key pattern |
| `zabbix_agent_log_remote_commands` | `true` | Log remote commands execution |
| `zabbix_agent_listen_port` | `10050` | Agent listen port |

### TLS/PSK Encryption (optional)

All TLS settings are disabled by default. Set them only when encryption is needed.

| Variable | Default | Description |
|----------|---------|-------------|
| `zabbix_agent_tls_connect` | `""` | Outgoing connection type: `unencrypted`, `psk`, or `cert` |
| `zabbix_agent_tls_accept` | `""` | Incoming connection types |
| `zabbix_agent_tls_psk_identity` | `""` | PSK identity string |
| `zabbix_agent_tls_psk_value` | `""` | PSK secret (hex string) - deployed to PSK file |
| `zabbix_agent_tls_psk_file` | `"/etc/zabbix/zabbix_agent2.psk"` | Path to PSK file |
| `zabbix_agent_tls_ca_file` | `""` | Path to CA certificate |
| `zabbix_agent_tls_cert_file` | `""` | Path to agent certificate |
| `zabbix_agent_tls_key_file` | `""` | Path to agent private key |

### Custom UserParameters (optional)

| Variable | Default | Description |
|----------|---------|-------------|
| `zabbix_agent_userparameters` | `[]` | List of UserParameter config files to deploy |

### Extra Configuration (optional)

| Variable | Default | Description |
|----------|---------|-------------|
| `zabbix_agent_extra_config` | `{}` | Key-value pairs appended to the config file |

### Firewall Management (optional)

| Variable | Default | Description |
|----------|---------|-------------|
| `zabbix_agent_firewall_manage` | `false` | Automatically open agent port in firewalld/ufw |

## Usage

### Basic Playbook

```yaml
---
- name: Deploy Zabbix Agent 2
  hosts: all
  become: true
  roles:
    - role: kevinmagni.zabbix_agent2
      vars:
        zabbix_agent_server: "192.168.1.100"
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

### With Zabbix Proxy

```yaml
---
- name: Deploy Zabbix Agent 2 via Proxy
  hosts: all
  become: true
  roles:
    - role: kevinmagni.zabbix_agent2
      vars:
        zabbix_agent_server: "10.0.0.50"
        zabbix_agent_server_active: "10.0.0.50"
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

### Using Tags

Run only specific parts of the role:

```bash
# Only install the repository
ansible-playbook site.yml --tags repo

# Only update configuration
ansible-playbook site.yml --tags conf

# Only manage the service
ansible-playbook site.yml --tags service

# Only deploy UserParameters
ansible-playbook site.yml --tags userparameters

# Only manage firewall
ansible-playbook site.yml --tags firewall
```

```bash
# Uninstall Zabbix Agent 2 completely
ansible-playbook site.yml --tags zabbix-agent-uninstall
```

Available tags: `repo`, `install-agent`, `conf`, `service`, `tls`, `userparameters`, `firewall`, `zabbix-agent-uninstall`

## What This Role Does

1. **Repository setup** - Downloads and installs the official Zabbix repository for the detected OS
2. **Package installation** - Installs `zabbix-agent2` and all available plugins
3. **Configuration** - Deploys `/etc/zabbix/zabbix_agent2.conf` via Jinja2 template
4. **TLS/PSK** - Optionally deploys PSK file with correct ownership and permissions
5. **UserParameters** - Optionally deploys custom check files to `zabbix_agent2.d/`
6. **Firewall** - Optionally opens the agent port via firewalld (RHEL/SUSE) or ufw (Debian/Ubuntu)
7. **Service management** - Enables and starts the `zabbix-agent2` service
8. **Handler** - Automatically restarts the agent when configuration changes

## Directory Structure

```
.
├── defaults/main.yml              # Default variables
├── handlers/main.yml              # Service restart handler
├── meta/main.yml                  # Galaxy metadata
├── tasks/
│   ├── main.yml                   # OS detection, routing, and shared post-install tasks
│   ├── ubuntu.yml                 # Ubuntu-specific tasks
│   ├── debian.yml                 # Debian-specific tasks
│   ├── redhat.yml                 # RHEL-specific tasks
│   ├── centos.yml                 # CentOS-specific tasks
│   ├── rocky_linux.yml            # Rocky Linux-specific tasks
│   ├── alma_linux.yml             # AlmaLinux-specific tasks
│   ├── oracle.yml                 # Oracle Linux-specific tasks
│   ├── fedora.yml                 # Fedora-specific tasks
│   ├── amazon_linux.yml           # Amazon Linux-specific tasks
│   ├── suse.yml                   # SUSE/SLES-specific tasks
│   ├── preinstall_repo_apt.yml    # APT repo cleanup
│   ├── preinstall_repo_dnf.yml    # DNF/RPM repo cleanup
│   ├── preinstall_repo_zypper.yml # Zypper repo cleanup
│   ├── configure_tls.yml          # TLS PSK file deployment
│   ├── configure_userparameters.yml # Custom UserParameter files
│   ├── configure_firewall.yml     # Firewall rule management
│   └── uninstall.yml              # Complete removal tasks
├── molecule/                      # Molecule test scenarios
├── templates/
│   └── zabbix_agent2.conf.j2     # Agent configuration template
├── vars/main.yml                  # Internal variables
├── tests/                         # Test playbook and inventory
└── examples/                      # Example playbooks
```

## Testing

### Quick syntax check

```bash
ansible-playbook tests/test.yml -i tests/inventory.ini --syntax-check
```

### Dry run (check mode)

```bash
ansible-playbook tests/test.yml -i tests/inventory.ini --check
```

### Molecule (requires Docker)

```bash
pip install molecule molecule-docker
molecule test
```

## License

Apache-2.0

## Author

**Kevin Magni** - [GitHub](https://github.com/kevinmagni)
