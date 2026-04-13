# Network Interface Role

This Ansible role configures network interface IP addresses on Windows systems. It supports both static IP configuration and DHCP.

## Requirements

- Windows Server 2016 or later, or Windows 10/11
- WinRM configured and accessible
- PowerShell 5.1 or later
- Administrator privileges on the target system

## Role Variables

### Required Variables (for static IP)

- `network_interface_ip_address`: The IP address to assign (e.g., "192.168.1.100")
- `network_interface_name`: The name of the network interface (e.g., "Ethernet", "Ethernet0")

### Optional Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `network_interface_prefix_length` | 24 | CIDR prefix length (subnet mask) |
| `network_interface_gateway` | "" | Default gateway IP address |
| `network_interface_dns_servers` | [] | List of DNS server IP addresses |
| `network_interface_use_dhcp` | false | Set to true to configure DHCP instead of static IP |
| `network_interface_remove_existing` | true | Remove existing IP configuration before applying new |
| `network_interface_validation_timeout` | 30 | Timeout for validation operations |

## Dependencies

None

## Example Playbook - Static IP Configuration

```yaml
---
- name: Configure Windows network interface with static IP
  hosts: windows_servers
  vars:
    ansible_connection: winrm
    ansible_winrm_transport: ntlm
    ansible_winrm_server_cert_validation: ignore
    
    network_interface_name: "Ethernet0"
    network_interface_ip_address: "192.168.1.100"
    network_interface_prefix_length: 24
    network_interface_gateway: "192.168.1.1"
    network_interface_dns_servers:
      - "8.8.8.8"
      - "8.8.4.4"
  
  roles:
    - network_interface
```

## Example Playbook - DHCP Configuration

```yaml
---
- name: Configure Windows network interface with DHCP
  hosts: windows_servers
  vars:
    ansible_connection: winrm
    ansible_winrm_transport: ntlm
    
    network_interface_name: "Ethernet0"
    network_interface_use_dhcp: true
  
  roles:
    - network_interface
```

## Finding Network Interface Names

To find the correct network interface name on your Windows system, run:

```powershell
Get-NetAdapter | Select-Object Name, InterfaceDescription, Status
```

Common interface names include:
- "Ethernet"
- "Ethernet0"
- "Local Area Connection"
- "Wi-Fi"

## License

MIT

## Author Information

Created for Windows system administration and automation.
