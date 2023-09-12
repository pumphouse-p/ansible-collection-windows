# README for `pumphouse_p.windows.disk_management`

## Description

An Ansible Role that manages disks configuration on Windows systems.

## Requirements

* `ansible.windows`
* `community.windows`

## Role Variables

Available variables are described in the table below. [Example](#example-playbook) is found further down in this document.

|      Variable Name      | Default | Required |    Type    |                          Description                          |
|:-----------------------:|:-------:|:--------:|:----------:|:-------------------------------------------------------------:|
|  `default_disk_style`   |  `gpt`  |    no    |    str     |     Default style of disk when initializing a new device.     |
|    `default_fs_type`    | `NTFS`  |    no    |    str     |        Default filesystem when formatting a partition         |
| `disk_management_disks` |  `[]`   |    no    | list(dict) | List of dictionaries containing desired storage configuration |

The list items in `disk_management_disks` define the desired storage configuration. Each
item supports the following parameters.

| Variable Name |       Default        | Required |    Type    |           Description           |
|:-------------:|:--------------------:|:--------:|:----------:|:-------------------------------:|
|   `number`    |         None         |   yes    |    int     |  Disk number being configured   |
|    `style`    | `default_disk_style` |    no    |    str     | Style of disk being initialized |
| `partitions`  |         None         |    no    | list(dict) | List of partitions to configure |

The list items in `partitions` define the desired partition configuration on the disk. Each
item supports the following parameters.

| Variable Name  |      Default      | Required | Type |                Description                 |
|:--------------:|:-----------------:|:--------:|:----:|:------------------------------------------:|
| `drive_letter` |       None        |   yes    | str  | Drive letter of partition being configured |
|  `size_in_gb`  |       None        |   yes    | int  |         Desired size of partition          |
|    `label`     |       None        |    no    | str  |        Label to apply to partition         |
|   `fs_type`    | `default_fs_type` |    no    | str  |          Desired filesystem type           |

## Example Playbook

```yaml
---
- name: Manage disks
  hosts: winservers
  gather_facts: true
  become: true

  roles:
  - role: pumphouse_p.windows.disk_management
    vars:
      disk_management_disks:
        - number: 1
          style: gpt
          partitions:
            - drive_letter: D
              size_in_gb: 10
              label: Data
              fs_type: NTFS
            - drive_letter: E
              size_in_gb: 40
              label: Media
              fs_type: NTFS
        - number: 2
          style: gpt
          partitions:
            - drive_letter: G
              size_in_gb: 5
              label: Share
```

## License

GPL

## Author Information

Devin Parrish ([GitHub](https://github.com/pumphouse-p))