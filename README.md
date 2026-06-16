# Ansible Collection - pumphouse_p.windows

A comprehensive Ansible collection for managing and configuring Windows systems. This collection provides roles and playbooks for common Windows administration tasks including system configuration, package management, storage management, and more.

## Version

1.0.0

## Author

Devin Parrish <deparris@redhat.com>

## Requirements

### Ansible Collections
* `ansible.windows`
* `community.windows`
* `chocolatey.chocolatey` (for package management)
* `community.aws` (for update reporting)

### Target Systems
* Windows Server 2012 or later
* Windows 8/10/11
* PowerShell 5.1 or later
* WinRM configured and accessible
* Administrator privileges

## Installation

Install from Ansible Galaxy:
```bash
ansible-galaxy collection install pumphouse_p.windows
```

Install from source:
```bash
git clone https://github.com/pumphouse-p/ansible-collection-windows.git
cd ansible-collection-windows
ansible-galaxy collection build
ansible-galaxy collection install pumphouse_p-windows-*.tar.gz
```

## Roles

### System Configuration

#### [disk_management](roles/disk_management/README.md)
Manages disk initialization, partitioning, and filesystem formatting on Windows systems.
* Initialize disks with GPT or MBR partition styles
* Create and format partitions with custom drive letters and labels
* Support for multiple disks and partitions

#### [network_interface](roles/network_interface/README.md)
Configures network interface IP addresses on Windows systems.
* Static IP address configuration
* DHCP configuration
* DNS server configuration
* Default gateway management

#### [pagefile](roles/pagefile/README.md)
Manages Windows pagefile (virtual memory) configuration.
* Configure pagefile location and size
* Support for custom initial and maximum sizes
* Automatic system reboot when configuration changes

#### [registry](roles/registry/README.md)
Manages Windows registry entries across all hives.
* Create, modify, or delete registry keys and values
* Support for all registry hives (HKLM, HKCU, HKCR, HKU, HKCC)
* Multiple data types (string, dword, qword, binary, etc.)
* Registry hive file loading support

### Security & Networking

#### [firewall_profiles](roles/firewall_profiles/README.md)
Manages Windows Firewall profiles (Domain, Private, Public).
* Enable or disable firewall for specific profiles
* Configure inbound and outbound traffic actions
* Support for multiple profile configuration

#### [features](roles/features/README.md)
Manages Windows features and roles installation.
* Enable Windows features (IIS, Hyper-V, AD, etc.)
* Disable features (SMB1, legacy components)
* Include sub-features automatically

### Software Management

#### [packages](roles/packages/README.md)
Manages software installation through multiple methods.
* Chocolatey package management (install, update, remove)
* MSI installer support (local and remote)
* Version-specific installations
* Custom package sources

#### [sqlserver](roles/sqlserver/README.md)
Installs and configures Microsoft SQL Server.
* Automated SQL Server installation
* Prerequisite installation (PowerShell modules, Windows features)
* Customizable instance and path configuration
* Support for different SQL Server editions

### Reporting

#### [update_report](roles/update_report/README.md)
Generates and publishes Windows Update reports.
* Scan for available Windows updates
* Generate HTML reports
* Publish reports to AWS S3 buckets
* Track update status across multiple systems

## Example Playbooks

The collection includes several example playbooks in the `playbooks/` directory:

### Network Configuration
* `network_interface_static.yml` - Configure static IP addresses
* `network_interface_dhcp.yml` - Configure DHCP

### Software Installation
* `chocolatey_install.yml` - Install Chocolatey package manager
* `sql_install.yml` - Install SQL Server
* `sql_install_win.yml` - Windows-specific SQL Server installation
* `sql_install_linux.yml` - Linux control node SQL Server deployment

### System Configuration
* `features_configure.yml` - Enable/disable Windows features
* `pagefile_configure.yml` - Configure pagefile settings
* `update_report.yml` - Generate Windows update reports

## Quick Start

### Basic Windows Server Configuration

```yaml
---
- name: Configure Windows Server
  hosts: windows_servers
  vars:
    ansible_connection: winrm
    ansible_winrm_transport: ntlm
    ansible_winrm_server_cert_validation: ignore
  
  roles:
    - role: pumphouse_p.windows.features
      vars:
        features_enabled:
          - Web-Server
          - NET-Framework-45-Features
    
    - role: pumphouse_p.windows.firewall_profiles
      vars:
        firewall_profile_state: enabled
        firewall_profile_inbound_action: block
        firewall_profile_outbound_action: allow
    
    - role: pumphouse_p.windows.packages
      vars:
        packages_chocolatey:
          - name: git
            state: present
          - name: vscode
            state: latest
```

### Configure Storage and Network

```yaml
---
- name: Configure storage and networking
  hosts: sql_servers
  vars:
    ansible_connection: winrm
  
  roles:
    - role: pumphouse_p.windows.disk_management
      vars:
        disk_management_disks:
          - number: 1
            style: gpt
            partitions:
              - drive_letter: D
                size_in_gb: 100
                label: SQL_Data
    
    - role: pumphouse_p.windows.network_interface
      vars:
        network_interface_name: "Ethernet0"
        network_interface_ip_address: "192.168.1.100"
        network_interface_prefix_length: 24
        network_interface_gateway: "192.168.1.1"
        network_interface_dns_servers:
          - "8.8.8.8"
          - "8.8.4.4"
    
    - role: pumphouse_p.windows.pagefile
      vars:
        pagefile_drive: "C"
        pagefile_initial_size: 8192
        pagefile_max_size: 16384
```

## WinRM Configuration

For Ansible to manage Windows hosts, WinRM must be configured. On the Windows target:

```powershell
# Quick setup (not for production)
winrm quickconfig

# For production, use HTTPS with certificate authentication
# See: https://docs.ansible.com/ansible/latest/os_guide/windows_setup.html
```

In your inventory:
```ini
[windows_servers]
server01.example.com

[windows_servers:vars]
ansible_connection=winrm
ansible_user=Administrator
ansible_password=SecurePassword
ansible_winrm_transport=ntlm
ansible_winrm_server_cert_validation=ignore
```

## Testing

Run tests using molecule (if configured):
```bash
molecule test
```

## Contributing

Contributions are welcome! Please submit pull requests to:
https://github.com/pumphouse-p/ansible-collection-windows

## Support

* **Issues**: https://github.com/pumphouse-p/ansible-collection-windows/issues
* **Documentation**: https://github.com/pumphouse-p/ansible-collection-windows

## License

GPL-2.0-or-later

## Links

* **GitHub Repository**: https://github.com/pumphouse-p/ansible-collection-windows
* **Ansible Galaxy**: https://galaxy.ansible.com/pumphouse_p/windows
