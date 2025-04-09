# Ansible UAMS Client Role

The Ansible UAMS Client Role installs and configures the UAMS Client.

# Setup
## Requirements
- When using Windows hosts, the `ansible.windows` collection is required. Install it on the controller host with the following command:
```
ansible-galaxy collection install ansible.windows
```
## Installation

Install the [UAMS Client](https://galaxy.ansible.com/solarwinds/uamsclient) role from Ansible Galaxy:
```
ansible-galaxy install solarwinds.uamsclient
```

To deploy the UAMS Client on hosts, add the `UAMS_ACCESS_TOKEN`, `UAMS_METADATA`, and `SWO_URL` to your playbook under the `environment` key. Values can be hardcoded, but for `UAMS_ACCESS_TOKEN`, it is recommended to use a variable to avoid exposing the token in plain text.

You can set an HTTPS proxy using the `UAMS_HTTPS_PROXY` environment variable. This variable configures the HTTPS proxy for connections established by the UAMS Client and its plugins. To use an HTTPS proxy during installation, configure the HTTPS proxy on your machine so that Ansible can use it.
If you set `UAMS_METADATA` to "role:host-monitoring", the UAMS Client will be installed with host monitoring. To skip installing the `uams-otel-collector-plugin`, leave this environment variable empty.
```yaml
  environment:
    UAMS_ACCESS_TOKEN: "YOUR_SWO_ACCESS_TOKEN"
    UAMS_METADATA: "role:host-monitoring"
    SWO_URL: "xx-yy.cloud.solarwinds.com"
    UAMS_HTTPS_PROXY: "https://your-proxy" # optional
    UAMS_OVERRIDE_HOSTNAME: "custom_hostname" # optional
    UAMS_MANAGED_LOCALLY: "true" # optional
```
### Examples

Below is an example playbook to install the UAMS Client with host monitoring. This playbook uses `localhost` as the target. Replace `YOUR_SWO_ACCESS_TOKEN` with your actual SolarWinds Observability access token.

```yaml
---
- name: Install UAMS Client on localhost
  hosts: localhost
  remote_user: root
  environment:
    SWO_URL: "na-01.cloud.solarwinds.com"
    UAMS_ACCESS_TOKEN: "YOUR_SWO_ACCESS_TOKEN"
    UAMS_METADATA: "role:host-monitoring"
  roles:
    - role: solarwinds.uamsclient
```

Please find [another example playbook that we use in CI testing](https://github.com/solarwinds/uamsclient-ansible/blob/master/ci_test/playbook_galaxy.yaml).

### Override Hostname
The optional environment variable `UAMS_OVERRIDE_HOSTNAME` allows you to set a custom Agent name. By default, the Agent name is set to the hostname. You can assign a value to this variable using inventory file variables. See the example below:

```
# Inventory file

[test_servers]
192.168.0.123 ansible_user=user override_hostname=web_server1
192.168.0.124 ansible_user=user override_hostname=web_server2
```
```yaml
# Playbook file

  environment:
    UAMS_OVERRIDE_HOSTNAME: "DEV_{{ override_hostname }}"
```

### Locally Managed Agents
The optional environment variable `UAMS_MANAGED_LOCALLY` configures the UAMS Agent as locally managed using a configuration file. This allows the UAMS Agent to be configured locally without requiring manual integration setup from the SWO page.

When installed as a **locally managed** agent, the UAMS Agent waits for the local configuration file to become accessible. The default local configuration file locations are:
- Linux: `/opt/solarwinds/uamsclient/var/local_config.yaml`
- Windows: `C:\ProgramData\SolarWinds\UAMSClient\local_config.yaml`

You can optionally use a `credentials_config.yaml` file to define credential providers. This file complements `local_config.yaml` by retrieving credentials from the specified providers. Both files are stored in the same directory.

Ansible automatically copies both files (`credentials_config.yaml` and `local_config.yaml`) to the required locations.

The default templates used by Ansible for these files are located in the **templates directory of the solarwinds.uamsclient role** at:
- `local_config.yaml`: `templates/template_local_config.yaml.j2`
- `credentials_config.yaml`: `templates/template_credentials_config.yaml.j2`

You can specify custom template paths by setting the `local_config_template` and/or `credentials_config_template` variables in the playbook.

```yaml
# Playbook file to install Agent configured by a local configuration file
---
- name: Install UAMS client
  # Existing hosts group from inventory
  hosts: uams-hosts

  environment:
    UAMS_ACCESS_TOKEN: "{{ uams_access_token }}"
    UAMS_METADATA: "{{ uams_metadata }}"
    SWO_URL: "{{ swo_url }}"
    UAMS_MANAGED_LOCALLY: "true"

  roles:
    - role: solarwinds.uamsclient
  vars:
    local_config_template: local_config.yaml.j2
    credentials_config_template: credentials_config.yaml.j2
```
In this case, the `local_config.j2` file from the current directory is used as the local configuration template, and the `credentials_template.j2` file is used as the credentials configuration template. 

When specifying template files, ensure that the paths are correctly defined and consider the directory precedence used by Ansible when searching for files.

You can use Jinja2 syntax to populate the template with appropriate variables. For more information on building the local config and credentials config, refer to the **[SolarWinds Observability documentation](https://documentation.solarwinds.com/en/success_center/observability/default.htm#cshid=app-agent-local-config)**. This documentation includes details on deploying plugins, integrations, and supported credential providers.

## Uninstallation

Install the [UAMS Client](https://galaxy.ansible.com/solarwinds/uamsclient) role from Ansible Galaxy
```
ansible-galaxy install solarwinds.uamsclient
```

To uninstall the UAMS Client on hosts, add the `uninstall` tag when running a playbook.
<br>Example:
```
ansible-playbook -i inventory playbook.yml --tags uninstall
```

Refer to [an example playbook that we use in CI testing](https://github.com/solarwinds/uamsclient-ansible/blob/master/ci_test/playbook_galaxy.yaml).

# Adding the DBO Plugin to the UAMS Client

The Ansible role simplifies the process of installing and configuring the DBO plugin for the UAMS Client. 
Before proceeding, ensure that the UAMS Client is already installed on the target hosts. 

Use the provided playbook to define the necessary variables and execute the installation and configuration of the DBO plugin.

## Preparing a Configuration File

To install and configure the DBO plugin, define the necessary variables in your Ansible inventory or secrets file. Below is the format for these variables.
Additionally, you must provide an API access token to install DBO plugins.
```yaml
api_access_token: "<api_access_token>"
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
Please refer to the [official documentation](https://documentation.solarwinds.com/en/success_center/observability/content/settings/api-tokens.htm?cshid=app-add-token-tag#Create) for instructions on obtaining an API access token.

## Installing the DBO Plugin

To install the DBO plugin, execute the following command:

```sh
ansible-playbook -i inventory playbook.yml --tags dbo
```

This command runs tasks associated with the `dbo` tag, installing and configuring the DBO plugin as specified in your inventory or variables files. Ensure your inventory and/or secrets are properly configured before running the playbook. 
This option uses API calls to SWO and is only available for **remote-managed agents (not locally managed)**.

## Ways to Provide Variables

1. **Inventory File**: Define the `dbo_plugin` and `api_access_token` variables directly in your inventory file.
2. **Group or Host Variables**
3. **Ansible Vault**

For sensitive information such as passwords, it is recommended to use Ansible Vault to encrypt your variables. You can create an encrypted file for your secrets.

Encrypt the file using the following command:

```sh
ansible-vault encrypt path/to/secrets.yml
```

Then reference it in your playbook or inventory file:

```yaml
vars_files:
  - path/to/secrets.yml
```

# Role Variables

| Variable                  | Description                                                                 |
|---------------------------|-----------------------------------------------------------------------------|
| `uams_local_pkg_path`     | Override the location where the installation package is stored (default: /tmp/uams) |
| `uams_local_pkg_path_windows` | Override the location where the installation package is stored on Windows (default: value of TEMP env variable) |
| `uams_remove_installer`   | Whether the installation package should be removed (default: true) |

# AWX

The UAMS Client role can also be used in an AWX setup. Consider the following:

1. AWX must be configured to download roles from Ansible Galaxy. At the current time (with AWX 0.30.0), it must be enabled in Settings > Jobs (`Enable Role Download`), and also Ansible Galaxy credentials must be configured at the `organization` level. The `roles/requirements.yml` must be present in the project repository and contain the required role ([example below](#rolesrequirementsyml)).
2. Values for variables representing `access token`, `role`, and `swo url` should be provided from the AWX interface.
3. The playbook must contain a `hosts:` value matching the hosts group (or individual host) defined in the AWX inventory. The playbook is not available in the AWX for `job template` creation if the proper value was not provided.
4. Setting `Privilege escalation` in the `job template` might cause failure for tasks delegated to localhost due to the missing `sudo` command in the execution container.

## Examples for AWX Setup

Below are examples to help you configure and use the UAMS Client role in an AWX setup. These examples demonstrate how to define variables, create job templates, and manage inventories effectively.

### Playbook
```yaml
---
- name: Install UAMS client
  # Existing hosts group from inventory
  hosts: uams-hosts

  environment:
    UAMS_ACCESS_TOKEN: "{{ uams_access_token }}"
    UAMS_METADATA: "{{ uams_metadata }}"
    SWO_URL: "{{ swo_url }}"

  roles:
    - role: solarwinds.uamsclient
```
### roles/requirements.yml

If `version` is omitted, the latest version will be installed.

```yaml
roles:
  - src: solarwinds.uamsclient
    version: 1.8.0