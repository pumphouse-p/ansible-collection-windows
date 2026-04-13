# Windows Network Interface Configuration with Ansible

This guide explains how to use the `network_interface` role and playbooks to configure IP addresses on Windows systems.

## Prerequisites

### On the Control Node (where Ansible runs)

1. Install required Python packages:
```bash
pip install pywinrm
```

2. For Kerberos authentication (optional):
```bash
pip install pywinrm[kerberos]
```

### On Windows Target Systems

1. Enable WinRM (run in PowerShell as Administrator):

```powershell
# Quick setup for testing (HTTP, not recommended for production)
winrm quickconfig -q
winrm set winrm/config/service '@{AllowUnencrypted="true"}'
winrm set winrm/config/service/auth '@{Basic="true"}'

# Recommended setup with HTTPS
$cert = New-SelfSignedCertificate -DnsName "hostname" -CertStoreLocation Cert:\LocalMachine\My
winrm create winrm/config/Listener?Address=*+Transport=HTTPS "@{Hostname=`"hostname`";CertificateThumbprint=`"$($cert.Thumbprint)`"}"
winrm set winrm/config/service/auth '@{Basic="true"}'
winrm set winrm/config/service '@{AllowUnencrypted="false"}'

# Configure firewall
New-NetFirewallRule -DisplayName "WinRM HTTPS" -Direction Inbound -LocalPort 5986 -Protocol TCP -Action Allow
```

2. Verify WinRM is running:
```powershell
Test-WSMan
```

## Quick Start

### 1. Configure Static IP Address

Create an inventory file `inventory.yml`:

```yaml
all:
  children:
    windows_servers:
      hosts:
        myserver:
          ansible_host: 192.168.1.50
      vars:
        ansible_connection: winrm
        ansible_port: 5986
        ansible_winrm_transport: ntlm
        ansible_winrm_server_cert_validation: ignore
        ansible_user: Administrator
        ansible_password: YourPassword
```

Run the static IP playbook:

```bash
ansible-playbook -i inventory.yml network_interface_static.yml \
  -e "network_interface_name=Ethernet0" \
  -e "network_interface_ip_address=192.168.1.100" \
  -e "network_interface_gateway=192.168.1.1"
```

### 2. Configure DHCP

```bash
ansible-playbook -i inventory.yml network_interface_dhcp.yml \
  -e "network_interface_name=Ethernet0"
```

## Configuration Options

### Connection Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `ansible_connection` | - | Must be `winrm` |
| `ansible_port` | 5986 | 5985 for HTTP, 5986 for HTTPS |
| `ansible_user` | - | Windows username |
| `ansible_password` | - | Windows password (use vault!) |
| `ansible_winrm_transport` | ntlm | ntlm, kerberos, basic, certificate |
| `ansible_winrm_server_cert_validation` | validate | ignore, validate |

### Network Interface Variables

| Variable | Required | Default | Description |
|----------|----------|---------|-------------|
| `network_interface_name` | Yes | "Ethernet" | Interface name |
| `network_interface_ip_address` | Yes* | "" | IP address (*only if static) |
| `network_interface_prefix_length` | No | 24 | Subnet CIDR prefix |
| `network_interface_gateway` | No | "" | Default gateway |
| `network_interface_dns_servers` | No | [] | List of DNS servers |
| `network_interface_use_dhcp` | No | false | Use DHCP instead of static |
| `network_interface_remove_existing` | No | true | Remove existing IPs first |

## Advanced Usage

### Using Ansible Vault for Credentials

1. Create a vault file:
```bash
ansible-vault create group_vars/windows_servers/vault.yml
```

2. Add encrypted variables:
```yaml
---
vault_windows_admin_password: "YourSecurePassword"
```

3. Reference in your variables:
```yaml
ansible_password: "{{ vault_windows_admin_password }}"
```

4. Run playbook with vault:
```bash
ansible-playbook -i inventory.yml network_interface_static.yml --ask-vault-pass
```

### Multiple Network Interfaces

To configure multiple interfaces, use `include_role` with different variables:

```yaml
---
- name: Configure multiple network interfaces
  hosts: windows_servers
  tasks:
    - name: Configure primary interface
      include_role:
        name: network_interface
      vars:
        network_interface_name: "Ethernet0"
        network_interface_ip_address: "192.168.1.100"
        network_interface_gateway: "192.168.1.1"

    - name: Configure secondary interface
      include_role:
        name: network_interface
      vars:
        network_interface_name: "Ethernet1"
        network_interface_ip_address: "10.0.0.100"
        network_interface_prefix_length: 16
```

### Finding Network Interface Names

Run this ad-hoc command to discover interface names:

```bash
ansible windows_servers -i inventory.yml -m ansible.windows.win_shell \
  -a "Get-NetAdapter | Select-Object Name, InterfaceDescription, Status | ConvertTo-Json"
```

Or run on the Windows server directly:

```powershell
Get-NetAdapter | Format-Table Name, InterfaceDescription, Status
```

## Troubleshooting

### Cannot connect via WinRM

1. Test WinRM from control node:
```bash
nc -zv <windows-host> 5986
```

2. Verify firewall allows WinRM:
```powershell
Get-NetFirewallRule -DisplayName "*WinRM*" | Select-Object DisplayName, Enabled, Direction, Action
```

3. Check WinRM service status:
```powershell
Get-Service WinRM
```

### Network adapter not found

List all adapters:
```powershell
Get-NetAdapter | Select-Object Name, InterfaceDescription, Status, LinkSpeed
```

### IP address conflict

The role removes existing IP addresses by default. To keep existing IPs, set:
```yaml
network_interface_remove_existing: false
```

### Changes don't persist

Ensure you're not running on a network with DHCP override or Group Policy that resets network settings.

## Security Best Practices

1. **Always use HTTPS (port 5986)** instead of HTTP (port 5985)
2. **Use ansible-vault** to encrypt credentials
3. **Use certificate or Kerberos authentication** instead of basic auth when possible
4. **Validate certificates** in production (don't use `ignore` for cert validation)
5. **Limit WinRM access** via firewall rules to specific control nodes
6. **Use least-privilege accounts** instead of Administrator when possible

## Examples

### Example 1: Configure static IP with custom DNS

```bash
ansible-playbook -i inventory.yml network_interface_static.yml \
  -e "network_interface_name='Ethernet0'" \
  -e "network_interface_ip_address='10.0.1.100'" \
  -e "network_interface_prefix_length=16" \
  -e "network_interface_gateway='10.0.0.1'" \
  -e "network_interface_dns_servers=['10.0.0.10','10.0.0.11']"
```

### Example 2: Switch from static to DHCP

```bash
ansible-playbook -i inventory.yml network_interface_dhcp.yml \
  -e "network_interface_name='Ethernet0'"
```

### Example 3: Configure with inventory variables

Create `host_vars/myserver.yml`:
```yaml
network_interface_name: "Ethernet0"
network_interface_ip_address: "192.168.1.100"
network_interface_prefix_length: 24
network_interface_gateway: "192.168.1.1"
network_interface_dns_servers:
  - "192.168.1.10"
  - "192.168.1.11"
```

Run:
```bash
ansible-playbook -i inventory.yml network_interface_static.yml -l myserver
```

## Support

For issues or questions about this role, check the README.md in the role directory or refer to the Ansible Windows documentation.
