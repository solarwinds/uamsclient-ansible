# Ansible UAMS Client installer

Ansible is a radically simple IT automation system. 
Use this sample playbook to download and execute the UAMS Client installation script on a remote host.

### Directory structure 
<pre>
.
├── README.md
├── ansible.cfg
├── inventory
├── playbook.yaml
└── vars.yaml
</pre>

- ansible.cfg - defines the inventory file and the private key
- inventory - keeps remote host IP addresses
- playbook.yaml - defines the tasks to execute on a remote machine
- vars.yaml - paramaters for the ansible playbook

### Ansible execution
command: ```ansible-playbook  playbook.yaml```
