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
| -------------------- | ----------------------------------------------------------- |
| `local_pkg_path`        | Override the location where installation package is stored |