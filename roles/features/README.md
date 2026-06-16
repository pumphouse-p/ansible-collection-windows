# README for `pumphouse_p.windows.features`

## Description

An Ansible Role that manages Windows features and roles. This role can enable or disable Windows features such as IIS, Hyper-V, Active Directory, and other optional Windows components. It supports both simple list format and advanced dictionary format for fine-grained control over feature installation options.

## Requirements

* `ansible.windows` collection
* Windows Server 2012 or later, or Windows 8/10/11
* Administrator privileges on the target system
* PowerShell 5.1 or later
* WinRM configured and accessible

## Role Variables

### Feature Lists

|         Variable Name          |  Default  | Required |      Type       |                        Description                        |
|:------------------------------:|:---------:|:--------:|:---------------:|:---------------------------------------------------------:|
|      `features_enabled`        |   `[]`    |    no    | list(str/dict)  | List of Windows features to enable                        |
|      `features_disabled`       |   `[]`    |    no    | list(str/dict)  | List of Windows features to disable                       |

### Feature Installation Options

|            Variable Name             | Default | Required | Type |                          Description                          |
|:------------------------------------:|:-------:|:--------:|:----:|:-------------------------------------------------------------:|
|  `features_include_sub_features`     | `true`  |    no    | bool | Include all sub-features when enabling features              |
| `features_include_management_tools`  | `false` |    no    | bool | Include management tools when enabling features               |
|     `features_fail_on_missing`       | `true`  |    no    | bool | Fail if a specified feature is not found on the system        |

### Reboot Options

|       Variable Name       |  Default  | Required | Type |                     Description                      |
|:-------------------------:|:---------:|:--------:|:----:|:----------------------------------------------------:|
|  `features_auto_reboot`   |  `true`   |    no    | bool | Automatically reboot if required after changes       |
| `features_reboot_timeout` |  `3600`   |    no    | int  | Maximum time to wait for reboot (in seconds)         |

## Feature List Formats

### Simple Format (String List)

Use simple strings for basic feature installation with default options:

```yaml
features_enabled:
  - Web-Server
  - NET-Framework-45-Features
  - Hyper-V
```

### Advanced Format (Dictionary List)

Use dictionary format for per-feature control over installation options:

```yaml
features_enabled:
  - name: Web-Server
    include_sub_features: true
    include_management_tools: true
  - name: Hyper-V
    include_sub_features: false
```

### Mixed Format

You can mix both formats in the same list:

```yaml
features_enabled:
  - Web-Server  # Uses role defaults
  - name: Hyper-V
    include_sub_features: false
  - RSAT-AD-PowerShell  # Uses role defaults
```

## Dependencies

None.

## Example Playbooks

### Basic: Enable Features

```yaml
---
- name: Enable Windows features
  hosts: win_servers
  become: true

  roles:
  - role: pumphouse_p.windows.features
    vars:
      features_enabled:
        - Web-Server
        - Web-Asp-Net45
        - NET-Framework-45-Features
        - Telnet-Client
```

### Basic: Disable Features

```yaml
---
- name: Disable Windows features
  hosts: win_servers
  become: true

  roles:
  - role: pumphouse_p.windows.features
    vars:
      features_disabled:
        - SMB1
        - Windows-Defender
```

### Advanced: Enable and Disable with Custom Options

```yaml
---
- name: Manage Windows features with custom options
  hosts: win_servers
  become: true

  roles:
  - role: pumphouse_p.windows.features
    vars:
      features_include_sub_features: false
      features_include_management_tools: true
      features_auto_reboot: true
      features_enabled:
        - name: Web-Server
          include_sub_features: true
          include_management_tools: true
        - name: Hyper-V
          include_sub_features: true
        - RSAT-AD-PowerShell
      features_disabled:
        - SMB1
        - Windows-Defender-Gui
```

### Web Server Configuration

```yaml
---
- name: Configure IIS web server
  hosts: web_servers
  become: true

  roles:
  - role: pumphouse_p.windows.features
    vars:
      features_include_sub_features: true
      features_include_management_tools: true
      features_enabled:
        - Web-Server
        - Web-WebServer
        - Web-Common-Http
        - Web-Default-Doc
        - Web-Dir-Browsing
        - Web-Http-Errors
        - Web-Static-Content
        - Web-Asp-Net45
        - Web-Mgmt-Console
      features_disabled:
        - SMB1  # Security: Disable SMBv1
```

### Hyper-V Host Setup

```yaml
---
- name: Configure Hyper-V host
  hosts: hyperv_hosts
  become: true

  roles:
  - role: pumphouse_p.windows.features
    vars:
      features_enabled:
        - name: Hyper-V
          include_sub_features: true
          include_management_tools: true
        - Hyper-V-PowerShell
        - RSAT-Hyper-V-Tools
      features_auto_reboot: true
```

### Disable Legacy/Insecure Features

```yaml
---
- name: Remove insecure features
  hosts: all_windows
  become: true

  roles:
  - role: pumphouse_p.windows.features
    vars:
      features_disabled:
        - SMB1                    # SMBv1 protocol
        - FS-SMB1                 # SMB 1.0/CIFS File Sharing Support
        - PowerShell-V2           # PowerShell 2.0 Engine
        - Windows-Defender-Gui    # Remove GUI but keep service
      features_fail_on_missing: false  # Don't fail if feature not present
```

## Tags

This role supports the following tags:

* `features` - All feature-related tasks
* `enable` - Only enable features
* `disable` - Only disable features
* `validation` - Only run variable validation
* `debug` - Display debug information

### Example: Only Enable Features

```bash
ansible-playbook playbook.yml --tags "features,enable"
```

### Example: Only Disable Features

```bash
ansible-playbook playbook.yml --tags "features,disable"
```

## Common Windows Features

Some commonly used Windows features include:

### Web Services (IIS)
* `Web-Server` - IIS Web Server
* `Web-WebServer` - Web Server (IIS) role
* `Web-Common-Http` - Common HTTP Features
* `Web-Asp-Net45` - ASP.NET 4.5/4.6/4.7/4.8
* `Web-Mgmt-Console` - IIS Management Console

### Active Directory
* `AD-Domain-Services` - Active Directory Domain Services
* `RSAT-ADDS` - AD DS Tools
* `RSAT-AD-PowerShell` - AD PowerShell Module
* `RSAT-AD-AdminCenter` - AD Administrative Center

### .NET Framework
* `NET-Framework-45-Features` - .NET Framework 4.5 Features
* `NET-Framework-Core` - .NET Framework 3.5 Features
* `NET-Framework-45-ASPNET` - ASP.NET 4.5/4.6/4.7/4.8

### Hyper-V
* `Hyper-V` - Hyper-V role
* `Hyper-V-PowerShell` - Hyper-V PowerShell Module
* `RSAT-Hyper-V-Tools` - Hyper-V Management Tools

### File Services
* `FS-FileServer` - File Server
* `FS-DFS-Namespace` - DFS Namespaces
* `FS-DFS-Replication` - DFS Replication
* `FS-Resource-Manager` - File Server Resource Manager

### Remote Desktop Services
* `RDS-RD-Server` - Remote Desktop Session Host
* `RDS-Licensing` - Remote Desktop Licensing
* `RDS-Gateway` - Remote Desktop Gateway

### Security & Legacy
* `SMB1` - SMB 1.0/CIFS (Legacy - should be disabled)
* `PowerShell-V2` - PowerShell 2.0 (Legacy - should be disabled)
* `Windows-Defender` - Windows Defender
* `BitLocker` - BitLocker Drive Encryption

## Finding Available Features

To list all available features on a Windows system:

```powershell
# List all features
Get-WindowsFeature

# List installed features only
Get-WindowsFeature | Where-Object {$_.Installed -eq $True}

# Search for specific features
Get-WindowsFeature -Name *Web*
Get-WindowsFeature -Name *Hyper*
```

## Best Practices

1. **Always disable SMBv1** - It's a security risk
2. **Include management tools** - Set `features_include_management_tools: true` for easier administration
3. **Test reboots** - Some features require a reboot; test with `features_auto_reboot: true` in non-production first
4. **Use tags** - Leverage tags to selectively run enable or disable operations
5. **Validate first** - Use `--check` mode to see what would change before applying

## Troubleshooting

### Feature Not Found

If you get an error about a feature not being available:

```yaml
features_fail_on_missing: false  # Continue even if feature is missing
```

### Reboot Required

If you want to control reboots manually:

```yaml
features_auto_reboot: false  # Don't auto-reboot
```

Then manually trigger reboots using:

```bash
ansible-playbook playbook.yml --tags features
# Review changes, then run reboot handler separately
```

## License

GPL-2.0-or-later

## Author Information

Devin Parrish ([GitHub](https://github.com/pumphouse-p))

