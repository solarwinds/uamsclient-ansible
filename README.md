# Ansible UAMS Client Role

The Ansible UAMS Client Role installs and configures UAMS Client.

# Setup
## Installation

Clone `uamsclient-ansible` repository to the path of your choice. Possibility to install role via Ansible Galaxy will be provided soon.

To deploy UAMS Client on hosts, add access token and role to your playbook under the `environment` key.

```
  environment:
    UAMS_ACCESS_TOKEN: "YOUR_SWO_ACCESS_TOKEN"
    UAMS_METADATA: "ROLE"
```

Set the path where this role is located

```
  roles:
    - role: '/path/to/the/role'
```

## Role variables

| Variable | Description |
| -------------------- | ----------------------------------------------------------- |
| `local_pkg_path`        | Override the location where installation package is stored |