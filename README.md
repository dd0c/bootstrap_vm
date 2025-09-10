# Bootstrap VM Ansible Role

Bootstrap Ansible role for newly deployed VMs with hostname, network, and user configuration.

## Overview
This Ansible role is designed to configure newly deployed RHEL VMs with the following features:
- Change hostname to Proxmox name (prompted during execution)
- Change IP address (prompted during execution)
- Create and configure a specified user with password from Ansible vault
- Set up authorized_keys for the user
- Configure network settings
- Install essential packages

## Role Structure
```
roles/bootstrap-vm/
├── defaults/
│   └── main.yml          # Default variables
├── handlers/
│   └── main.yml          # Handlers for service restarts
├── meta/
│   └── main.yml          # Role metadata
├── tasks/
│   └── main.yml          # Main role tasks
├── vars/
│   └── main.yml          # Role-specific variables
└── templates/            # Template files (if needed)
```

## Files Structure
```
bootstrap/
├── ansible.cfg              # Ansible configuration
├── inventory.yml            # Host inventory
├── example-playbook.yml     # Example playbook using the role
├── authorized_keys          # SSH public keys for user
├── vars/
│   └── vault.yml           # Encrypted sensitive variables
└── roles/
    └── bootstrap-vm/        # The Ansible role
```

## Role Variables

### Default Variables (defaults/main.yml)
- `ansible_user_name`: Username to create (default: "tycho")
- `ansible_user_groups`: Groups for the user (default: ["wheel"])
- `ansible_user_shell`: User shell (default: "/bin/bash")
- `netmask`: Network mask (default: "255.255.255.0")
- `dns1`, `dns2`: DNS servers (default: "8.8.8.8", "8.8.4.4")
- `required_packages`: List of packages to install
- `network_interface`: Network interface name (default: "eth0")

### Vault Variables (vars/vault.yml)
- `vault_ansible_user_password`: Encrypted password for the user
- `vault_ansible_user_authorized_keys`: SSH public keys for the user

## Setup Instructions

### 1. Configure the Vault
Edit the vault file with your sensitive data:
```bash
ansible-vault edit vars/vault.yml
```

Update the following variables:
- `vault_ansible_user_password`: Set the encrypted password for the user
- `vault_ansible_user_authorized_keys`: Add your SSH public keys

### 2. Update Inventory
Edit `inventory.yml` to match your target host's current IP address.

### 3. Customize Role Variables (Optional)
You can override default variables in your playbook or inventory:
```yaml
- name: Configure VM with custom user
  hosts: rhel_hosts
  become: yes
  vars:
    ansible_user_name: "admin"
    ansible_user_groups: ["wheel", "docker"]
  roles:
    - bootstrap-vm
```

## Usage

### Basic Usage
```bash
ansible-playbook example-playbook.yml --ask-vault-pass
```

### With Custom Variables
```bash
ansible-playbook example-playbook.yml --ask-vault-pass \
  -e "new_hostname=proxmox-vm-01" \
  -e "new_ip=192.168.1.100" \
  -e "ansible_user_name=admin"
```

### Using the Role in Your Own Playbook
```yaml
---
- name: Configure newly deployed VMs
  hosts: new_vms
  become: yes
  vars_files:
    - vars/vault.yml
  roles:
    - bootstrap-vm
```

## Prerequisites
- Ansible installed on the control machine
- SSH access to the target VM as root
- Target VM running RHEL/CentOS/Rocky Linux/AlmaLinux/Fedora

## Supported Operating Systems
- Red Hat Enterprise Linux 7, 8, 9
- CentOS 7, 8
- Rocky Linux 8, 9
- AlmaLinux 8, 9
- Fedora 35, 36, 37, 38, 39

## Security Notes
- The vault file contains sensitive information and should be encrypted
- SSH keys should be properly managed and rotated
- Consider using SSH key-based authentication instead of passwords for production use
- The role creates a user with sudo privileges without password

## Role Dependencies
None

## Example Playbook
See `example-playbook.yml` for a complete example of how to use this role.

## License
MIT