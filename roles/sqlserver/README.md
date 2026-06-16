# README for `pumphouse_p.windows.sqlserver`

## Description

An Ansible Role that installs and configures Microsoft SQL Server on Windows systems. This role handles prerequisites, downloads the installer, extracts media, and performs a complete SQL Server installation with customizable instance and path configuration.

## Requirements

* `ansible.windows`
* `community.windows`
* Windows Server 2016 or later
* PowerShell 5.1 or later
* Administrator privileges on the target system
* Internet connectivity for downloading SQL Server installer

## Role Variables

Available variables are described in the table below.

| Variable Name | Default | Required | Type | Description |
|:-------------:|:-------:|:--------:|:----:|:-----------:|
| `sqlserver_installer_url` | `https://go.microsoft.com/fwlink/p/?linkid=866662` | no | str | URL to SQL Server installer (default: SQL Server 2019 Developer) |
| `sqlserver_download_path` | `C:\tmp` | no | str | Path to download setup files |
| `sqlserver_install_path` | `C:\SQLServer` | no | str | Installation path for SQL Server media |
| `sqlserver_instance_name` | `DefaultDB` | no | str | Name of the SQL Server instance |
| `sqlserver_drive` | `C` | no | str | Drive letter for database files |
| `sqlserver_port` | `1433` | no | int | SQL Server port number |
| `sqlserver_install_shared_dir` | `C:\Program Files\Microsoft SQL Server` | no | str | Shared installation directory |
| `sqlserver_install_sharedwow_dir` | `C:\Program Files (x86)\Microsoft SQL Server` | no | str | Shared WOW64 installation directory |
| `sqlserver_instance_dir` | `C:\Program Files\Microsoft SQL Server\{{ sqlserver_instance_name }}` | no | str | Instance installation directory |
| `sqlserver_install_sql_data_dir` | See defaults | no | str | SQL data directory |
| `sqlserver_userdb_dir` | See defaults | no | str | User database directory |
| `sqlserver_userdb_log_dir` | See defaults | no | str | User database log directory |
| `sqlserver_tempdb_dir` | `C:\TempDBFiles\Data\{{ sqlserver_instance_name }}` | no | str | TempDB data directory |
| `sqlserver_tempdb_log_dir` | `C:\TempDBFiles\Logs\{{ sqlserver_instance_name }}` | no | str | TempDB log directory |
| `sqlserver_features` | `SQLENGINE,FULLTEXT,CONN` | no | str | SQL Server features to install |
| `sqlserver_collation` | `SQL_Latin1_General_CP1_CI_AS` | no | str | SQL Server collation |
| `sqlserver_startup_type` | `Automatic` | no | str | SQL Server service startup type |
| `sqlserver_sysadmin_accounts` | `ansible` | no | str | System administrator accounts |
| `sqlserver_suppress_reboot` | `false` | no | bool | Suppress automatic reboot after installation |

## Dependencies

This role automatically installs the following dependencies:
* PowerShell modules: `SQLServerDSC`, `dbatools`
* Windows Features: `NET-Framework-45-Features`, `WAS`

## Example Playbook

```yaml
---
- name: Install SQL Server
  hosts: win_servers
  become: true

  roles:
  - role: pumphouse_p.windows.sqlserver
    vars:
      sqlserver_instance_name: "MYINSTANCE"
      sqlserver_sysadmin_accounts: "DOMAIN\\SQLAdmins"
      sqlserver_features: "SQLENGINE,FULLTEXT,CONN,IS"
      sqlserver_collation: "SQL_Latin1_General_CP1_CI_AS"
```

## Custom Installation Example

```yaml
---
- name: Install SQL Server with custom paths
  hosts: sql_servers
  become: true

  roles:
  - role: pumphouse_p.windows.sqlserver
    vars:
      sqlserver_instance_name: "PRODDB"
      sqlserver_drive: "D"
      sqlserver_userdb_dir: "D:\\SQLData\\UserDB"
      sqlserver_userdb_log_dir: "D:\\SQLData\\UserLogs"
      sqlserver_tempdb_dir: "E:\\SQLData\\TempDB"
      sqlserver_tempdb_log_dir: "E:\\SQLData\\TempLogs"
      sqlserver_sysadmin_accounts: "DOMAIN\\DBA-Team"
```

## Tags

This role supports the following tags:
* `prereqs` - Install prerequisites only (PowerShell modules and Windows features)
* `install` - Perform SQL Server installation only

## License

GPL

## Author Information

Devin Parrish ([GitHub](https://github.com/pumphouse-p))
