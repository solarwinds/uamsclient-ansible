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

You have the option to set an HTTPS proxy through the use of the `UAMS_HTTPS_PROXY` environment variable. Simply define this variable to point to your desired HTTPS proxy. Remember that the `UAMS_HTTPS_PROXY` environmental variable sets HTTPS proxy only for the connections established by the UAMS Client and its plugins. To use HTTPS proxy during installation set up HTTPS proxy on your machine so that ansible will be able to use it.

```
  environment:
    UAMS_ACCESS_TOKEN: "YOUR_SWO_ACCESS_TOKEN"
    UAMS_METADATA: "ROLE"
    SWO_URL: "https://swo-url"
    UAMS_HTTPS_PROXY: "https://your-proxy" # optional
    UAMS_OVERRIDE_HOSTNAME: "custom_hostname" # optional
    UAMS_MANAGED_LOCALLY: "true" # optional
```

Please find [example playbook that we use in CI testing](ci_test/playbook_galaxy.yaml).

### Override hostname
Optional environment variable `UAMS_OVERRIDE_HOSTNAME` is used to set a custom Agent name. By default, Agent name is set to the hostname.
You can assign value to this variable using variables from inventory file. See example below.


```
# Inventory file

[test_servers]
192.168.0.123 ansible_user=user override_hostname=web_server1
192.168.0.124 ansible_user=user override_hostname=web_server2
```
```
# Playbook file

  environment:
    UAMS_OVERRIDE_HOSTNAME: "DEV_{{ override_hostname }}"
```


### Locally managed Agents
An optional environment variable `UAMS_MANAGED_LOCALLY` is used to set the UAMS Agent as a locally managed by a configuration file. 
Is designed to allow configuration of the UAMS Agent locally, without necessity of adding integrations manually from SWO page.

If the UAMS Agent gets installed as a **locally managed** agent then it will wait for the local configuration file to be accessible. The default local configuration locations are:
- Linux - `/opt/solarwinds/uamsclient/var/local_config.yaml`
- Windows - `C:\ProgramData\SolarWinds\UAMSClient\local_config.yaml`

Ansible will automatically copy the file to the needed location. 
The default template of the local config file is located at `templates/template_local_config.yaml.j2`.
You can specify the **path** to your own template by setting the `local_config_template` variable in the playbook.

```
# Playbook file to install Agent configured by local configuration file
---
- name: Install UAMS client
  # Exisiting hosts group from inventory
  hosts: uams-hosts

  environment:
    UAMS_ACCESS_TOKEN: "{{ uams_access_token  }}"
    UAMS_METADATA: "{{ uams_metadata  }}"
    SWO_URL: "{{ swo_url }}"
    UAMS_MANAGED_LOCALLY: "true"

  roles:
    - role: solarwinds.uamsclient
  vars:
    local_config_template: template.j2
```
In this case the `template.j2` file from the current directory will be used as the local config template.

You can use jinja2 syntax to fill the template with appropriate variables.
To learn more about building the appropriate local config, check out the official documentation


## Uninstallation

Install the [UAMS Client](https://galaxy.ansible.com/solarwinds/uamsclient) role from Ansible Galaxy
```
ansible-galaxy install solarwinds.uamsclient
```

To uninstall UAMS Client on hosts, add `uninstall` tag when running a playbook.
<br>Example:
```
ansible-playbook -i inventory playbook.yml --tags uninstall
```

Please find [example playbook that we use in CI testing](ci_test/playbook_galaxy.yaml).

# Adding DBO Plugin to UAMS Client

## Overview

This Ansible role installs and configures the DBO plugin for the UAMS Client. To install the DBO plugin, you need to run the following command:

```sh
ansible-playbook -i inventory playbook.yml --tags dbo
```

This will execute the tasks associated with the `dbo` tag, installing and configuring the DBO plugin as specified in your inventory or variables files.
Ensure your inventory and/or secrets are properly configured before running the playbook.

## Configuration

To use the DBO plugin, you must define the necessary variables in your Ansible inventory or secrets file. Below is the format for these variables:
Additionally, you must provide a token with full access to have access to the API to install DBO plugins
```yaml
uams_full_access_token: "<full_access_token>"
dbo_plugin:
  - databaseType: "mongo"
    name: "mongodb profiler on dev-amd64-mu listening on 10.0.2.2:27018"
    host: "10.0.2.2"
    port: "27018"
    user: "myUser"
    password: "<password>"
    packetCaptureEnabled: false
    metricsCaptureMethod: "profiler"
```

### Ways to Provide Variables

1. Inventory File: you can define the `dbo_plugin` and `uams_full_access_token` variable directly in your inventory file.
2. Group or Host Variables
3. Ansible Vault

For sensitive information such as passwords, it is recommended to use Ansible Vault to encrypt your variables. You can create an encrypted file for your secrets:


Encrypt this file using the following command:

```sh
ansible-vault encrypt path/to/secrets.yml
```

Then reference it in your playbook or inventory file:

```yaml
vars_files:
  - path/to/secrets.yml
```

## Role variables

| Variable | Description |
| -------------------- | --------------------------------------------------------------- |
| `uams_local_pkg_path` | Override the location where installation package is stored (default: /tmp/uams) |
| `uams_local_pkg_path_windows` | Override the location where installation package is stored on Windows (default: value of TEMP env variable) |
| `uams_remove_installer` | If installation package should be removed (default: true) |

# AWX

UAMS Client role can be also used in the AWX setup. The following should be taken into consideration:

1. AWX must be configured to download roles from Ansible Galaxy. At the the current time (with AWX 0.30.0) it must be enabled in Settings > Jobs (`Enable Role Download`) and also Ansible Galaxy credentials must be configured at the `organization` level. The `roles/requirements.yml` must be present in the project repository and contain required role ([example below](#rolesrequirementsyml)).
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
    version: 1.8.0
```