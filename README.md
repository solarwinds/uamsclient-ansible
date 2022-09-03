# Ansible UAMS Client Role

The Ansible UAMS Client Role installs and configures UAMS Client.

# Setup
## Installation

Install the [UAMS Client](https://galaxy.ansible.com/solarwinds/uamsclient) role from Ansible Galaxy
```
ansible-galaxy install solarwinds.uamsclient
```

To deploy UAMS Client on hosts, add access token and role to your playbook under the `environment` key.

```
  environment:
    UAMS_ACCESS_TOKEN: "YOUR_SWO_ACCESS_TOKEN"
    UAMS_METADATA: "ROLE"
```

## Role variables

| Variable | Description |
| -------------------- | --------------------------------------------------------------- |
| `uams_local_pkg_path` | Override the location where installation package is stored (default: /tmp/uams) |
| `uams_local_pkg_path_windows` | Override the location where installation package is stored on Windows (default: value of TEMP env variable) |
| `uams_remove_installer` | If installation package should be removed (default: true) |