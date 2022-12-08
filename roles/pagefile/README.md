# README for `windows.pagefile`

## Description

An Ansible Role that manages the Windows pagefile.

## Requirements

* `community.windows`
* `ansible.windows`

## Role Variables

Available variables are described in the table below.

| Variable Name | Default | Required | Type | Description |
|:-------------:|:-------:|:--------:|:----:|:-----------:|
|     `pagefile_drive`     | `C` |    yes    | str  | The drive of the pagefile  |
| `pagefile_initial_size` | None | yes | int | Initial size of the pagefile in megabytes |
| `pagefile_max_size` | None | yes | int | Maximum size of the pagefile in megabytes |


## Dependencies

None.

## Example Playbook

```yaml
---
- name: Ensure pagefile is configured
  hosts: win_servers
  become: true

  roles:
  - role: pumphouse_p.windows.pagefile
    vars:
      pagefile_drive: "C"
      pagefile_initial_size: 2048
      pagefile_max_size: 4096
```

## License

GPL

## Author Information

Devin Parrish ([GitHub](https://github.com/pumphouse-p))