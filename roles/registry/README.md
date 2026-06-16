# README for `pumphouse_p.windows.registry`

## Description

An Ansible Role that manages Windows registry entries. This role allows you to create, modify, or delete registry keys and values across all registry hives.

## Requirements

* `ansible.windows`
* Windows Server 2012 or later, or Windows 8/10/11
* Administrator privileges on the target system (for HKLM and system-wide changes)

## Role Variables

Registry entries are configured through the variable called `registry_entries`. 
Each item in the list supports the variables described in the table below.

| Variable Name | Default | Required |  Type  |                  Description                   |
|:-------------:|:-------:|:--------:|:------:|:----------------------------------------------:|
|    `name`     |  `None` |    no    |  str   | Name of registry value in the `path` parameter (omit to work with the default value) |
|    `path`     |  `None` |   yes    |  str   | Full path to the registry key (e.g., `HKLM:\Software\MyApp`) |
|    `data`     |  `None` |    no    |  any   | Value of the registry entry |
|    `hive`     |  `None` |    no    |  path  | Path to a registry hive file to load (e.g., `C:\Users\Default\NTUSER.dat`) |
|    `state`    |  `None` |   yes    | string | State of registry entry (`present` or `absent`) |
|    `type`     |  `None` |    no    | string | Registry value data type (`string`, `dword`, `qword`, `binary`, `expandstring`, `multistring`) |

## Registry Hives

Common registry hives include:
* **HKLM** (`HKEY_LOCAL_MACHINE`) - Machine-wide settings
* **HKCU** (`HKEY_CURRENT_USER`) - Current user settings
* **HKCR** (`HKEY_CLASSES_ROOT`) - File associations and COM objects
* **HKU** (`HKEY_USERS`) - All user profiles
* **HKCC** (`HKEY_CURRENT_CONFIG`) - Hardware profile information

## Registry Value Types

* `string` - Regular string value (REG_SZ)
* `expandstring` - Expandable string value (REG_EXPAND_SZ) with environment variables
* `binary` - Binary data (REG_BINARY)
* `dword` - 32-bit number (REG_DWORD)
* `qword` - 64-bit number (REG_QWORD)
* `multistring` - Multiple string values (REG_MULTI_SZ)

## Dependencies

None

## Example Playbook

### Disable Windows Auto Update

```yaml
---
- name: Manage registry entries
  hosts: win_servers
  become: true

  roles:
  - role: pumphouse_p.windows.registry
    vars:
      registry_entries:
        - path: HKLM:\Software\Policies\Microsoft\Windows\WindowsUpdate\AU
          name: NoAutoUpdate
          data: 1
          type: dword
          state: present
```

### Multiple Registry Entries

```yaml
---
- name: Configure multiple registry settings
  hosts: win_servers
  become: true

  roles:
  - role: pumphouse_p.windows.registry
    vars:
      registry_entries:
        - path: HKLM:\Software\MyCompany\MyApp
          name: InstallPath
          data: "C:\\Program Files\\MyApp"
          type: string
          state: present
        - path: HKLM:\Software\MyCompany\MyApp
          name: EnableFeature
          data: 1
          type: dword
          state: present
        - path: HKCU:\Control Panel\Mouse
          name: MouseTrails
          data: 0
          type: dword
          state: present
```

### Remove Registry Entry

```yaml
---
- name: Remove registry entry
  hosts: win_servers
  become: true

  roles:
  - role: pumphouse_p.windows.registry
    vars:
      registry_entries:
        - path: HKCU:\Software\MyCompany
          name: ObsoleteValue
          state: absent
```

### Load and Modify Registry Hive

```yaml
---
- name: Modify default user registry hive
  hosts: win_servers
  become: true

  roles:
  - role: pumphouse_p.windows.registry
    vars:
      registry_entries:
        - path: HKLM:\ANSIBLE\Control Panel\Mouse
          name: MouseTrails
          data: 10
          type: string
          hive: C:\Users\Default\NTUSER.dat
          state: present
```

## License

GPL

## Author Information

Devin Parrish ([GitHub](https://github.com/pumphouse-p))
