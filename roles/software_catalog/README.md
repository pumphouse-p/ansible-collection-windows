# README for `pumphouse_p.windows.software_catalog`

## Description

Maintains approved software on Windows hosts from a **single central catalog**,
[`vars/software_catalog.yml`](vars/software_catalog.yml).

* The catalog defines every approved application, its approved versions, and
  *how* each is installed (installer URL, checksum, silent switches, detection).
* **Inventory groups decide what each host gets.** A host in the
  `notepadplusplus` and `java_runtime` groups gets those two apps.
* An AAP survey (or extra vars) can override the groups for one-off runs.
* Anything not in the catalog, or an unapproved version, fails the play.

Supported install methods (no package manager required):

| Method | Detection / idempotency                                           |
|:------:|-------------------------------------------------------------------|
| `msi`  | ProductCode read from the cached MSI                              |
| `exe`  | `creates_path` (+ optional `creates_version`); `product_id` for removal |

Installers are downloaded once to `software_catalog_download_dir` and reused.

## Requirements

* `ansible.windows` >= 2.0.0
* Administrator rights on the target
* Network access to installer URLs (or point `url` at an internal share/repo)

## How a host's software is selected

1. **Explicit list** – if `software_catalog_apps` is non-empty (AAP survey or
   extra vars), it is used and groups are ignored.
2. **Inventory groups** – otherwise, every group the host is in whose name
   matches a catalog id. Parent groups count, so bundle groups work.
3. **Version** – an explicit `name@version` > `software_catalog_pins` >
   catalog `default_version`.
4. **Removal** – with `software_catalog_remove_unlisted: true` (group mode
   only), every catalog app/version the host isn't selected for is removed.

## Role Variables

|            Variable Name             |                    Default                    |    Type     | Description |
|:------------------------------------:|:---------------------------------------------:|:-----------:|-------------|
|       `software_catalog_apps`        |                     `[]`                      | list / str  | Explicit selection (survey). Overrides groups when non-empty. List, or newline-separated string. Items: `name`, `name@version`, or dict with `name`/`version`/`state`. |
|     `software_catalog_use_groups`    |                    `true`                     | bool        | Select apps from inventory group membership when no explicit list is given. |
|       `software_catalog_pins`        |                     `{}`                      | dict        | Per-app version pins, e.g. `{nodejs: "20.18.0"}`. Set on an inventory group or host. |
|  `software_catalog_remove_unlisted`  |                   `false`                     | bool        | Group mode only: remove catalog software the host isn't selected for. |
|   `software_catalog_default_state`   |                  `present`                    | str         | State for explicit items that don't set one (`present` / `absent`). |
|       `software_catalog_file`        | `{{ role_path }}/vars/software_catalog.yml`   | str         | Path to the central catalog. |
|   `software_catalog_download_dir`    | `C:\ProgramData\Ansible\SoftwareCatalog`      | str         | Installer cache on the host. |
| `software_catalog_require_checksum`  |                   `false`                     | bool        | Fail if a catalog entry has no SHA256. |
|    `software_catalog_auto_reboot`    |                   `false`                     | bool        | Reboot if an installer requests it. |
|   `software_catalog_reboot_timeout`  |                   `3600`                      | int         | Reboot timeout (seconds). |

## Approved Catalog

| Id                | Application               | Method | Approved versions (**default**) | Side-by-side |
|-------------------|---------------------------|:------:|---------------------------------|:------------:|
| `notepadplusplus` | Notepad++ (x64)           | exe    | **8.7.1**, 8.6.9                | no           |
| `firefox`         | Mozilla Firefox (x64 en-US) | exe  | **160.0.1**, 159.0.1            | no           |
| `nodejs`          | Node.js LTS (x64)         | msi    | **22.11.0**, 20.18.0            | no           |
| `dotnet_runtime`  | .NET Runtime (x64)        | exe    | **8.0.11**, 9.0.0               | yes          |
| `java_runtime`    | Eclipse Temurin JRE (x64) | msi    | **21.0.5+11**, 17.0.13+11       | yes          |

See the header of `vars/software_catalog.yml` for the full entry schema.
Only `side_by_side: true` apps may have more than one version `present` on a host.

### Adding an application or version

Edit `vars/software_catalog.yml` only. Shared settings go on the app; anything
that differs per version goes under `versions` and overrides the shared value:

```yaml
software_catalog_approved:
  7zip:
    display_name: 7-Zip (x64)
    method: msi
    default_version: "24.08"
    side_by_side: false
    versions:
      "24.08":
        url: https://www.7-zip.org/a/7z2408-x64.msi
        checksum: "<sha256>"
```

Then create a matching inventory group (`7zip`). Always quote version keys so
YAML keeps them as strings. For an `exe`, also set `arguments`, `creates_path`
(+ `creates_version` if all versions share a path) and, for removal,
`uninstall_arguments` (plus `product_id` if the installer can't uninstall
itself). To roll out a new version, add it under `versions` and update
`default_version`.

## Inventory Example

Full example: [`docs/software_catalog_inventory.example.yml`](../../docs/software_catalog_inventory.example.yml).
App groups contain **bundle** groups; each host joins one bundle:

```yaml
all:
  children:
    windows:
      children:
        # --- app groups: names must match catalog ids -------------------
        notepadplusplus: { children: { bundle_basic: , bundle_full: , bundle_no_node: } }
        java_runtime:    { children: { bundle_basic: , bundle_full: , bundle_no_node: } }
        dotnet_runtime:  { children: { bundle_full: , bundle_no_node: } }
        nodejs:          { children: { bundle_full: } }

        # --- bundles: hosts go here -------------------------------------
        bundle_basic:    { hosts: { host_a: } }        # Notepad++, JRE
        bundle_full:                                     # everything
          hosts:
            host_b:
              software_catalog_pins: { nodejs: "20.18.0" }
        bundle_no_node:  { hosts: { host_c: } }        # everything but Node.js
```

Result:

| Host     | Installs                                               |
|----------|--------------------------------------------------------|
| `host_a` | notepadplusplus 8.7.1, java_runtime 21.0.5+11          |
| `host_b` | all four; nodejs pinned to 20.18.0                     |
| `host_c` | notepadplusplus, java_runtime, dotnet_runtime          |

Hosts can also be placed directly in app groups; bundles just reduce repetition.

## Ansible Automation Platform

Playbook: [`playbooks/software_catalog.yml`](../../playbooks/software_catalog.yml).

**Desired-state job template (recommended).** No survey. Run against the
Windows inventory (on a schedule, or as a workflow node). Each host gets
exactly what its groups say, installed sequentially in one job. Set
`software_catalog_remove_unlisted: true` in the template's extra vars once
you trust the inventory, so removing a host from a group uninstalls the app.

**Ad-hoc job template (optional).** Same playbook with a survey:

| Survey field                  | Type                        | Notes |
|-------------------------------|-----------------------------|-------|
| `software_catalog_apps`       | Multiple choice (multiple select) | Choices like `notepadplusplus`, `nodejs@20.18.0`, `dotnet_runtime@9.0.0` |
| `software_catalog_default_state` | Multiple choice (single select) | `present` / `absent` |

Tick **Prompt on launch → Limit** to target a host or group. Survey runs never
trigger `remove_unlisted`, so a targeted run can't strip other software.

> Don't put `software_catalog_apps` in a workflow's own survey/extra vars:
> workflow-level variables override every node and would switch all nodes
> to explicit mode.

## Notes

* Absent items are processed before present ones, so a version switch can
  remove the old version in the same run.
* Group names are matched against catalog ids across the whole inventory.
  Avoid reusing a catalog id as a group name for unrelated (e.g. Linux) hosts.
* With `remove_unlisted`, MSI apps are checked by reading the ProductCode
  from their installer, so each unlisted MSI version is downloaded once to
  the cache on every host. EXE apps are checked without downloading.
* `.NET` runtimes and Temurin major versions install side-by-side. Without
  `remove_unlisted`, changing `default_version` adds the new one but leaves
  the old one. Temurin's JAVA_HOME/PATH follow the most recently installed JRE.
* Node.js MSIs refuse to downgrade. With `remove_unlisted` the newer version
  is removed first automatically; otherwise remove it with an explicit
  `nodejs@22.11.0` + `state: absent` run before pinning 20.18.0.
* Notepad++ installs over whatever version is present, in either direction.
* Firefox also installs over the present version in either direction. After a
  downgrade, Firefox's profile downgrade protection may prompt users to create
  a new profile. Firefox's built-in updater stays enabled unless you turn it
  off via enterprise policy (`DisableAppUpdate`). Otherwise it moves hosts past
  the catalog version, and the next run reinstalls the pinned version.

## License

GPL-2.0-or-later

## Author Information

Devin Parrish ([GitHub](https://github.com/pumphouse-p))
