# Ansible UAMS Client Role

The Ansible UAMS Client Role installs and configures UAMS Client.

# Setup
## Requirements
- When using with Windows hosts the ansible.windows collection is required. Please install it on the controller host with the following command:
```
ansible-galaxy collection install ansible.windows
```
## Installation

Install the [UAMS Client](https://galaxy.ansible.com/solarwinds/uamsclient) role from Ansible Galaxy
```
ansible-galaxy install solarwinds.uamsclient
```

To deploy UAMS Client on hosts, add `access token`, `role` and `swo url` to your playbook under the `environment` key. Values can be hardcoded but at least for `access token` it's recommended to use variable and not expose token in plain text. 

```
  environment:
    UAMS_ACCESS_TOKEN: "YOUR_SWO_ACCESS_TOKEN"
    UAMS_METADATA: "ROLE"
    SWO_URL: "https://swo-url"
```

Please find [example playbook that we use in CI testing](ci_test/playbook_galaxy.yaml).

## Role variables

| Variable | Description |
| -------------------- | --------------------------------------------------------------- |
| `uams_local_pkg_path` | Override the location where installation package is stored (default: /tmp/uams) |
| `uams_local_pkg_path_windows` | Override the location where installation package is stored on Windows (default: value of TEMP env variable) |
| `uams_remove_installer` | If installation package should be removed (default: true) |

# AWX

UAMS Client role can be also used in the AWX setup. The following should be taken into consideration:

1. AWX must be configured to download roles from Ansible Galaxy. At the the current time (with AWX 0.30.0) it must be enabled in Settings > Jobs (`Enable Role Download`) and also Ansible Galaxy credentials must be configured at the `organization` level. The `roles/requirements.yml` must be present in the project repository and contain required role ([example below](#roles/requirements.yml)).
2. Values for variables representing `access token`, `role` and `swo url` should be provided from AWX interface.
3. Playbook must contain `hosts:` value matching hosts group (or individual host) defined in the AWX inventory. Playbook is not available in the AWX for `job template` creation if the proper value was not provided.
4. Setting `Privilege escalation` in `job template` might cause failure for tasks delegated to localhost due to missing `sudo` command in the execution container.

## Examples for AWX setup
### Playbook
```
---
- name: Install UAMS client
  # Exisiting hosts group from inventory
  hosts: uams-hosts

  environment:
    UAMS_ACCESS_TOKEN: "{{ uams_access_token  }}"
    UAMS_METADATA: "{{ uams_metadata  }}"
    SWO_URL: "{{ swo_url }}"

  roles:
    - role: solarwinds.uamsclient
```
### roles/requirements.yml

If `version` is omitted, the latest version shall be installed.

```
roles:
  - src: solarwinds.uamsclient
    version: 1.4.0
```