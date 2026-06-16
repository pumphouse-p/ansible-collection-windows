# README for `pumphouse_p.windows.features`

## Description

An Ansible Role that manages Windows features and roles. This role can enable or disable Windows features such as IIS, Hyper-V, Active Directory, and other optional Windows components.

## Requirements

* `ansible.windows`
* Windows Server 2012 or later, or Windows 8/10/11
* Administrator privileges on the target system
* PowerShell 5.1 or later

## Role Variables

Available variables are described in the table below.

|    Variable Name    | Default | Required |   Type    |               Description                |
|:-------------------:|:-------:|:--------:|:---------:|:----------------------------------------:|
| `features_enabled`  | `None`  |    no    | list(str) | List of Windows features that should be enabled with sub-features |
| `features_disabled` | `None`  |    no    | list(str) | List of Windows features that should be disabled |

## Dependencies

None.

## Example Playbook

### Enable Features

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

### Disable Features

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

### Enable and Disable Features Together

```yaml
---
- name: Manage Windows features
  hosts: win_servers
  become: true

  roles:
  - role: pumphouse_p.windows.features
    vars:
      features_enabled:
        - Hyper-V
        - RSAT-AD-PowerShell
      features_disabled:
        - SMB1
        - Windows-Defender-Gui
```

## Common Windows Features

Some commonly used Windows features include:
* **IIS**: `Web-Server`, `Web-WebServer`, `Web-Common-Http`
* **Active Directory**: `AD-Domain-Services`, `RSAT-ADDS`
* **.NET Framework**: `NET-Framework-45-Features`, `NET-Framework-Core`
* **Hyper-V**: `Hyper-V`, `Hyper-V-PowerShell`
* **File Services**: `FS-FileServer`, `FS-DFS-Namespace`
* **Remote Desktop**: `RDS-RD-Server`, `RDS-Licensing`

To list all available features on a Windows system, run:
```powershell
Get-WindowsFeature
```

## License

GPL

## Author Information

Devin Parrish ([GitHub](https://github.com/pumphouse-p))
