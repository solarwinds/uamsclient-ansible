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

To deploy UAMS Client on hosts, add `access token`, `role` and `swo_url` to your playbook under the `environment` key.

```
  environment:
    UAMS_ACCESS_TOKEN: "YOUR_SWO_ACCESS_TOKEN"
    UAMS_METADATA: "ROLE"
    UAMS_SWO_URL: "https://swo-url"
```

## Role variables

| Variable | Description |
| -------------------- | --------------------------------------------------------------- |
| `uams_local_pkg_path` | Override the location where installation package is stored (default: /tmp/uams) |
| `uams_local_pkg_path_windows` | Override the location where installation package is stored on Windows (default: value of TEMP env variable) |
| `uams_remove_installer` | If installation package should be removed (default: true) |