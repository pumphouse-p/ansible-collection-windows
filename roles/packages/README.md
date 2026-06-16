# README for `pumphouse_p.windows.packages`

## Description

An Ansible Role that manages software packages on Windows systems through multiple installation methods:
* **Chocolatey** - Popular Windows package manager for installing, updating, and removing software
* **MSI** - Direct installation from MSI installer files (local or remote URLs)

## Requirements

* `ansible.windows`
* `chocolatey.chocolatey`
* Windows Server 2012 or later, or Windows 8/10/11
* Administrator privileges on the target system
* Internet connectivity (for Chocolatey packages and remote MSI downloads)

## Role Variables

Available variables are described in the table below.

|          Variable Name          |    Default     | Required |    Type    |                Description                |
|:-------------------------------:|:--------------:|:--------:|:----------:|:-----------------------------------------:|
|     `packages_chocolatey`       |     `None`     |    no    | list(dict) | List of packages to manage via Chocolatey |
|        `packages_msi`           |     `None`     |    no    | list(dict) | List of MSI packages to install |
| `packages_msi_download_dir`     | `C:\Installers`|    no    |    str     | Directory to download MSI files to |
| `packages_msi_setup_filename`   | `setup.exe`    |    no    |    str     | Default filename for MSI downloads |

### Chocolatey Package Parameters

Each item in `packages_chocolatey` supports:
* `name` (required) - Package name from Chocolatey repository
* `state` (required) - `present`, `absent`, or `latest`
* `version` (optional) - Specific version to install
* `source` (optional) - Alternate Chocolatey source/feed

### MSI Package Parameters

Each item in `packages_msi` supports:
* `url` (required) - URL to the MSI file or local path
* `setup_filename` (optional) - Custom filename for the downloaded installer
* `product_id` (optional) - MSI product ID for idempotency checks
* `arguments` (optional) - Additional MSI installation arguments

## Dependencies

None. The role will automatically install Chocolatey if it's not already present when using `packages_chocolatey`.

## Example Playbook

### Install and Remove Chocolatey Packages

```yaml
---
- name: Manage Chocolatey packages
  hosts: win_servers
  become: true

  roles:
  - role: pumphouse_p.windows.packages
    vars:
      packages_chocolatey:
        - name: "git"
          state: "present"
        - name: "notepadplusplus"
          state: "latest"
        - name: "putty"
          state: "absent"
        - name: "python"
          state: "present"
          version: "3.11.0"
```

### Install MSI Packages

```yaml
---
- name: Install software from MSI files
  hosts: win_servers
  become: true

  roles:
  - role: pumphouse_p.windows.packages
    vars:
      packages_msi:
        - url: "https://example.com/software/app-installer.msi"
          setup_filename: "myapp.msi"
        - url: "https://github.com/petrkle/vim-msi/releases/download/v8.1.55/vim-8.1.55.msi"
          setup_filename: "vim.msi"
```

### Combined Chocolatey and MSI

```yaml
---
- name: Install packages via multiple methods
  hosts: win_servers
  become: true

  roles:
  - role: pumphouse_p.windows.packages
    vars:
      packages_chocolatey:
        - name: "git"
          state: "present"
        - name: "vscode"
          state: "latest"
        - name: "7zip"
          state: "present"
      packages_msi:
        - url: "https://example.com/custom-app.msi"
          setup_filename: "custom-app.msi"
      packages_msi_download_dir: "D:\\Installers"
```

### Install Specific Package Versions

```yaml
---
- name: Install specific versions
  hosts: win_servers
  become: true

  roles:
  - role: pumphouse_p.windows.packages
    vars:
      packages_chocolatey:
        - name: "nodejs"
          state: "present"
          version: "18.16.0"
        - name: "terraform"
          state: "present"
          version: "1.5.0"
```

## Popular Chocolatey Packages

Some commonly used packages:
* **Development**: `git`, `vscode`, `python`, `nodejs`, `golang`, `dotnetcore-sdk`
* **Tools**: `7zip`, `putty`, `winscp`, `notepadplusplus`, `sysinternals`
* **Browsers**: `googlechrome`, `firefox`, `microsoft-edge`
* **Utilities**: `powershell-core`, `curl`, `wget`, `jq`

Search for packages at: https://community.chocolatey.org/packages

## License

GPL

## Author Information

Devin Parrish ([GitHub](https://github.com/pumphouse-p))
