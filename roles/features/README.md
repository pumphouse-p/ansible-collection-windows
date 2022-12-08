# README for `windows.features`

## Description

An Ansible Role that manages Windows features.

## Requirements

* `ansible.windows`

## Role Variables

Available variables are described in the table below.

|    Variable Name    | Default | Required |   Type    |               Description                |
|:-------------------:|:-------:|:--------:|:---------:|:----------------------------------------:|
| `features_enabled`  | `None`  |    no    | list(str) | List of features that should be enabled  |
| `features_disabled` | `None`  |    no    | list(str) | List of features that should be disabled |


## Dependencies

None.

## Example Playbook

```yaml
---
- name: Play name
  hosts: win_servers
  become: true

  roles:
  - role: pumphouse_p.windows.features
```

## License

GPL

## Author Information

Devin Parrish ([GitHub](https://github.com/pumphouse-p))