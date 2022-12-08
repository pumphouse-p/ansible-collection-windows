# README for `windows.firewall_profiles`

## Description

An Ansible Role that manages the Windows Firewall Profiles.

## Requirements

* `community.windows`

## Role Variables

Available variables are described in the table below.

|           Variable Name            |           Default           | Required |   Type    |                    Description                     |
|:----------------------------------:|:---------------------------:|:--------:|:---------:|:--------------------------------------------------:|
|      `firewall_profile_state`      |          `enabled`          |    no    |    str    |    Sets state of firewall for the given profile    |
|      `firewall_profile_name`       | `[Domain, Private, Public]` |   yes    | list(str) |         The name of the profile to manage          |
| `firewall_profile_inbound_action`  |            None             |    no    |    str    | Set to `allow` or `block` inbound network traffic  |
| `firewall_profile_outbound_action` |            None             |    no    |    str    | Set to `allow` or `block` outbound network traffic |

## Dependencies

None.

## Example Playbook

```yaml
---
- name: Manage Firewall Profiles
  hosts: win_servers
  become: true

  roles:
  - role: pumphouse_p.windows.firewall_profiles
```

## License

GPL

## Author Information

Devin Parrish ([GitHub](https://github.com/pumphouse-p))