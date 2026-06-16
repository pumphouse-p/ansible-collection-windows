# README for `pumphouse_p.windows.disk_management`

## Description

An Ansible Role that manages disk configuration on Windows systems. This role handles disk initialization, partitioning, and filesystem formatting for new or existing disks.

## Requirements

* `ansible.windows`
* `community.windows`
* Windows Server 2012 or later, or Windows 8/10/11
* Administrator privileges on the target system

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

## Dependencies

None.

## Example Playbook

### Configure Multiple Disks with Partitions

```yaml
---
- name: Configure storage disks
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

### Single Disk Configuration

```yaml
---
- name: Configure single data disk
  hosts: sql_servers
  gather_facts: true
  become: true

  roles:
  - role: pumphouse_p.windows.disk_management
    vars:
      disk_management_disks:
        - number: 1
          style: gpt
          partitions:
            - drive_letter: S
              size_in_gb: 100
              label: SQL_Data
              fs_type: NTFS
            - drive_letter: L
              size_in_gb: 50
              label: SQL_Logs
              fs_type: NTFS
```

### MBR Disk Configuration

```yaml
---
- name: Configure disk with MBR partition style
  hosts: legacy_servers
  gather_facts: true
  become: true

  roles:
  - role: pumphouse_p.windows.disk_management
    vars:
      default_disk_style: mbr
      disk_management_disks:
        - number: 1
          partitions:
            - drive_letter: D
              size_in_gb: 50
              label: Data
```

## Disk Identification

To identify disk numbers on Windows systems, run:
```powershell
Get-Disk | Select-Object Number, FriendlyName, OperationalStatus, Size
```

## Partition Styles

* **GPT (GUID Partition Table)** - Modern partition style supporting disks larger than 2TB, recommended for Windows Server 2012+ and UEFI systems
* **MBR (Master Boot Record)** - Legacy partition style with 2TB disk limit, used for older systems or BIOS compatibility

## Tags

This role supports the following tags:
* `disk_init` - Initialize disks only
* `disk_partition` - Create partitions only
* `disk_write_fs` - Format filesystems only

## Notes

* **Destructive operations**: This role initializes and formats disks. Ensure you have backups before running.
* Disk initialization will **erase all data** on the specified disk
* The role is **idempotent** - running it multiple times will not recreate existing partitions
* Drive letters must be unique across the system
* Partition sizes should not exceed the physical disk capacity

## License

GPL

## Author Information

Devin Parrish ([GitHub](https://github.com/pumphouse-p))
