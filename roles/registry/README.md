# README for `windows.registry`

## Description

An Ansible Role that manages Windows registry entries.

## Requirements

* `ansible.windows`

## Role Variables

Registry entries are configured through the variable called `registry_entries`. 
Each item in the list supports the variables described in the table below.

| Variable Name | Default | Required |  Type  |                  Description                   |
|:-------------:|:-------:|:--------:|:------:|:----------------------------------------------:|
|    `name`     |  None   |    no    |  str   | Name of registry entry in the `path` parameter |
|    `path`     |  None   |   yes    |  str   |           Name of the registry path            |
|    `data`     |  None   |    no    |  any   |          Value of the registry entry           |
|    `hive`     |  None   |    no    |  path  |               Path to a hive key               |
|    `state`    |  None   |   yes    | string |            State of registry entry             |
|    `type`     |  None   |    no    | string |         Registry entry value data type         |


## Dependencies

None

## Example Playbook

```yaml
---
- name: Manage registry entries
  hosts: win_servers
  become: true

  roles:
  - role: pumphouse_p.windows.registry
    vars:
      registry_entries:
        - key: HKLM:\Software\Policies\Microsoft\Windows\WindowsUpdate\AU
          type: dword
          name: NoAutoUpdate
          data: 00000001
```

## License

GPL

## Author Information

Devin Parrish ([GitHub](https://github.com/pumphouse-p))