# README for `pumphouse_p.windows.firewall_profiles`

## Description

An Ansible Role that manages Windows Firewall profiles (Domain, Private, and Public). This role allows you to enable/disable the firewall and configure default inbound and outbound actions for each network profile.

## Requirements

* `community.windows`
* Windows Server 2012 or later, or Windows 8/10/11
* Administrator privileges on the target system

## Role Variables

Available variables are described in the table below.

|           Variable Name            |           Default           | Required |   Type    |                    Description                     |
|:----------------------------------:|:---------------------------:|:--------:|:---------:|:--------------------------------------------------:|
|      `firewall_profile_state`      |          `enabled`          |    no    |    str    | Sets state of firewall for the given profile (`enabled` or `disabled`) |
|      `firewall_profile_name`       | `[Domain, Private, Public]` |    no    | list(str) | The name of the profile(s) to manage |
| `firewall_profile_inbound_action`  |            `None`           |    no    |    str    | Set to `allow` or `block` for inbound network traffic |
| `firewall_profile_outbound_action` |            `None`           |    no    |    str    | Set to `allow` or `block` for outbound network traffic |

## Windows Firewall Profiles

Windows Firewall uses three network profiles:
* **Domain** - Applied when connected to a domain controller
* **Private** - Applied when connected to a private network (home/work)
* **Public** - Applied when connected to a public network (untrusted)

## Dependencies

None.

## Example Playbook

### Enable Firewall on All Profiles

```yaml
---
- name: Enable Windows Firewall on all profiles
  hosts: win_servers
  become: true

  roles:
  - role: pumphouse_p.windows.firewall_profiles
```

### Configure Public Profile Only

```yaml
---
- name: Configure Public firewall profile
  hosts: win_servers
  become: true

  roles:
  - role: pumphouse_p.windows.firewall_profiles
    vars:
      firewall_profile_name:
        - Public
      firewall_profile_state: enabled
      firewall_profile_inbound_action: block
      firewall_profile_outbound_action: allow
```

### Disable Firewall on Private Network

```yaml
---
- name: Disable firewall for private network
  hosts: win_servers
  become: true

  roles:
  - role: pumphouse_p.windows.firewall_profiles
    vars:
      firewall_profile_name:
        - Private
      firewall_profile_state: disabled
```

### Configure Multiple Profiles

```yaml
---
- name: Configure domain and private profiles
  hosts: win_servers
  become: true

  roles:
  - role: pumphouse_p.windows.firewall_profiles
    vars:
      firewall_profile_name:
        - Domain
        - Private
      firewall_profile_state: enabled
      firewall_profile_inbound_action: block
      firewall_profile_outbound_action: allow
```

## License

GPL

## Author Information

Devin Parrish ([GitHub](https://github.com/pumphouse-p))
