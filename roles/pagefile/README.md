# README for `pumphouse_p.windows.pagefile`

## Description

An Ansible Role that manages the Windows pagefile (virtual memory). This role allows you to configure the location and size of the Windows page file, which is used when physical RAM is exhausted.

## Requirements

* `community.windows`
* `ansible.windows`
* Windows Server 2012 or later, or Windows 8/10/11
* Administrator privileges on the target system

## Role Variables

Available variables are described in the table below.

|      Variable Name      | Default | Required | Type |                Description                |
|:-----------------------:|:-------:|:--------:|:----:|:-----------------------------------------:|
|    `pagefile_drive`     |   `C`   |    no    | str  | The drive letter for the pagefile (without colon) |
| `pagefile_initial_size` |  `None` |   yes    | int  | Initial size of the pagefile in megabytes |
|   `pagefile_max_size`   |  `None` |   yes    | int  | Maximum size of the pagefile in megabytes |

## Pagefile Sizing Guidelines

Microsoft's general recommendations for pagefile size:
* **Minimum**: 1.5 times the amount of physical RAM
* **Maximum**: 3 times the amount of physical RAM
* **System-managed**: Let Windows automatically manage size (not supported by this role)

For specific workloads:
* **Database servers**: Often benefit from smaller pagefiles or none at all
* **Application servers**: Typically 1x to 2x physical RAM
* **Workstations**: System-managed or 1.5x physical RAM

**Note**: This role will trigger a system reboot if the pagefile configuration is changed.

## Dependencies

None.

## Example Playbook

### Standard Configuration

```yaml
---
- name: Configure pagefile
  hosts: win_servers
  become: true

  roles:
  - role: pumphouse_p.windows.pagefile
    vars:
      pagefile_drive: "C"
      pagefile_initial_size: 2048
      pagefile_max_size: 4096
```

### Large Server Configuration

```yaml
---
- name: Configure pagefile for server with 32GB RAM
  hosts: sql_servers
  become: true

  roles:
  - role: pumphouse_p.windows.pagefile
    vars:
      pagefile_drive: "D"
      pagefile_initial_size: 49152   # 48 GB (1.5x 32GB)
      pagefile_max_size: 98304       # 96 GB (3x 32GB)
```

### Fixed Size Pagefile

```yaml
---
- name: Configure fixed-size pagefile
  hosts: win_servers
  become: true

  roles:
  - role: pumphouse_p.windows.pagefile
    vars:
      pagefile_drive: "C"
      pagefile_initial_size: 8192
      pagefile_max_size: 8192   # Same as initial for fixed size
```

## Notes

* The role sets `override: true`, which will replace any existing pagefile configuration
* A system reboot is required for pagefile changes to take effect (handled automatically by the role's reboot handler)
* Placing the pagefile on a separate physical drive from the OS can improve performance
* SSD drives benefit less from dedicated pagefile drives compared to traditional HDDs

## License

GPL

## Author Information

Devin Parrish ([GitHub](https://github.com/pumphouse-p))
