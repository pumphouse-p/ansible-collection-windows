# README for `windows.packages`

## Description

An Ansible Role that manages packages (MSIs and via Chocolatey) on Windows systems.

## Requirements

* `ansible.windows`
* `chocolatey.chocolatey`

## Role Variables

Available variables are described in the table below.

|     Variable Name     | Default | Required |    Type    |                Description                |
|:---------------------:|:-------:|:--------:|:----------:|:-----------------------------------------:|
| `packages_chocolatey` |  None   |    no    | list(dict) | List of packages to manage via Chocolatey |
|    `packages_msi`     |  None   |    no    | list(dict) |          List of MSIs to install          |

Refer to the example below to see how to use these variables.

## Dependencies

None.

## Example Playbook

```yaml
---
- name: Ensure packages
  hosts: win_servers
  become: true

  roles:
  - role: pumphouse_p.windows.packages
    vars:
      packages_chocolatey:
        - name: "git"
          state: "present"
        - name: "putty"
          state: "absent"
        - name: "notepadplusplus"
          state: "latest"
      packages_msi:
        - url: "https://github.com/petrkle/vim-msi/releases/download/v8.1.55/vim-8.1.55.msi"
          setup_filename: "vim.msi"
```

## License

GPL

## Author Information

Devin Parrish ([GitHub](https://github.com/pumphouse-p))