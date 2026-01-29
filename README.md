# Ansible Automation Platform 2.6 QuickStart - Single Node Containerized Installer

## Overview 
This is an unofficial guide to assist with installing the Ansible Automation Platform on a single node using the containerized installer. 

There are currently three methods to install the Ansible Automation platform components

1. **Containerized Installer (recommended)**
2. RPM Installer 
3. Operator Installer (for OpenShift)

The containerized installer will deploy the following AAP components as containers on one or more hosts. See the [Planning your AAP Installation](https://docs.redhat.com/en/documentation/red_hat_ansible_automation_platform/2.6/html-single/planning_your_installation/index#planning-installation) docs for more details.
- Platform Gateway
- Private Automation Hub
- Ansible Controller
- Event Driven Ansible Controller
- Database

Helpful Links
- [Unofficial (but helpful) AAP Wiki](https://github.com/naps-sled-sas/aap_wiki)
- [Planning your AAP Installation](https://docs.redhat.com/en/documentation/red_hat_ansible_automation_platform/2.6/html-single/planning_your_installation/index#planning-installation)
- [AAP Containerized Installer Docs](https://docs.redhat.com/en/documentation/red_hat_ansible_automation_platform/2.6/html/containerized_installation/index)
- [Video: AAP Containerized Installer Walkthrough](https://www.youtube.com/watch?v=wUcCeyrCvyg&t=24s&ab_channel=RedHatAnsibleAutomation)

## Prerequisites

[**Minimum System Requirements**](https://docs.redhat.com/en/documentation/red_hat_ansible_automation_platform/2.6/html/containerized_installation/preparing-containerized-installation#system-requirements)
- RAM: 16GB 
- CPUs: 4
- Local Disk: 60GB
- Disk IOPS: 3000

**OS Prerequisites**
- A host VM running RHEL 9.4 or later
- A non-root user for the Red Hat Enterprise Linux host, with sudo or other Ansible supported privilege escalation (sudo recommended). This user is responsible for the installation of containerized Ansible Automation Platform.
- The appropriate network ports are open if a firewall is in place. For more information about the ports to open, see [Container topologies](https://docs.redhat.com/en/documentation/red_hat_ansible_automation_platform/2.6/html/tested_deployment_models/container-topologies) in *Tested deployment models*.

## Step 1: Preparing your RHEL Host

1. Set a hostname that is a fully qualified domain name (FQDN) 
    ```shell 
    sudo hostnamectl set-hostname <your_hostname>
    ```

2. Register your RHEL Host
    ```shell
    sudo subscription-manager register
    ```

3. Ensure that BaseOS and AppStream repos are enabled on the host

    ```shell
    sudo dnf repolist
    ```
4. Ensure that the host has DNS configured and can resolve host names and IP addresses by using a fully qualified domain name (FQDN)
   
5. Install the ```ansible-core``` package
   ```shell
   sudo dnf install -y ansible-core
   ```
6. (Optional) Install additional utilities for troubleshooting purposes
   ```shell
   sudo dnf install -y wget git-core rsync vim
   ```

## Step 2: Downloading the AAP Installer

1. Download the latest **Ansible Automation Platform 2.6 Containerized Setup Bundle** .tar file from [Ansible Automation Platform download page](https://access.redhat.com/downloads/content/480/ver=2.6/rhel---9/2.6/x86_64/product-software)

   
3.  Copy the installation program .tar file onto your RHEL host. In this case we are using scp to copy the installer from a workstation to our RHEL host.
    ```shell
    scp ~/Downloads/<aap_setup_bundle_file> <username>:<remote_host>:/target/path
    ```
**NOTE** The installer will require at least 15GB of available space within the filesystem where it exists. If there is insufficient space within your home directory for example, the installer will not pass its pre-requiste checks.

4. Unpack the bundled installer

    ```shell
    tar xfvz ansible-automation-platform-containerized-setup-bundle-<version>-<arch_name>.tar.gz
    ```


## Step 3: Configuring the inventory file

1. On your RHEL host, cd into the unpacked installer directory

    ```shell
    cd ansible-automation-platform-containerized-setup-bundle-<version>-<arch_name>
    ```

2. Create a new inventory file and edit the necessary vars to customize your installation. To start, we can use example inventory file provided via the docs [here](https://docs.redhat.com/en/documentation/red_hat_ansible_automation_platform/2.6/html/tested_deployment_models/container-topologies#cont-a-env-a).
  
    **NOTES** 
    - SSH keys are only required when installing on remote hosts. If doing a self contained local VM based installation, you can use ansible_connection=local.
    - You do have the option to leverage ansible-vault to secure sensitive information in your inventory file ([docs with examples](https://docs.redhat.com/en/documentation/red_hat_ansible_automation_platform/2.6/html/tested_deployment_models/container-topologies#cont-a-env-a))

    ```shell
    vim inventory-custom
    ```
For reference, this an example inventory provided for us via the docs [here](https://docs.redhat.com/en/documentation/red_hat_ansible_automation_platform/2.6/html/tested_deployment_models/container-topologies#cont-a-env-a).
```ini
# This is the Ansible Automation Platform installer inventory file intended for the container growth deployment topology.
# This inventory file expects to be run from the host where Ansible Automation Platform will be installed.
# Consult the Ansible Automation Platform product documentation about this topology's tested hardware configuration.
# https://docs.redhat.com/en/documentation/red_hat_ansible_automation_platform/2.6/html/tested_deployment_models/container-topologies
#
# Consult the docs if you are unsure what to add
# For all optional variables consult the included README.md
# or the Ansible Automation Platform documentation:
# https://docs.redhat.com/en/documentation/red_hat_ansible_automation_platform/2.6/html/containerized_installation

# This section is for your platform gateway hosts
# -----------------------------------------------------
[automationgateway]
aap.example.org

# This section is for your automation controller hosts
# -----------------------------------------------------
[automationcontroller]
aap.example.org

# This section is for your automation hub hosts
# -----------------------------------------------------
[automationhub]
aap.example.org

# This section is for your Event-Driven Ansible controller hosts
# -----------------------------------------------------
[automationeda]
aap.example.org

# This section is for the Ansible Automation Platform database
# -----------------------------------------------------
[database]
aap.example.org

[all:vars]
# Ansible
ansible_connection=local

# Common variables
# https://docs.redhat.com/en/documentation/red_hat_ansible_automation_platform/2.6/html/containerized_installation/appendix-inventory-files-vars#general-variables
# -----------------------------------------------------
postgresql_admin_username=postgres
postgresql_admin_password=<set your own>

registry_username=<your RHN username>
registry_password=<your RHN password>

redis_mode=standalone

# Platform gateway
# https://docs.redhat.com/en/documentation/red_hat_ansible_automation_platform/2.6/html/containerized_installation/appendix-inventory-files-vars#platform-gateway-variables
# -----------------------------------------------------
gateway_admin_password=<set your own>
gateway_pg_host=aap.example.org
gateway_pg_password=<set your own>

# Automation controller
# https://docs.redhat.com/en/documentation/red_hat_ansible_automation_platform/2.6/html/containerized_installation/appendix-inventory-files-vars#controller-variables
# -----------------------------------------------------
controller_admin_password=<set your own>
controller_pg_host=aap.example.org
controller_pg_password=<set your own>
controller_percent_memory_capacity=0.5

# Automation hub
# https://docs.redhat.com/en/documentation/red_hat_ansible_automation_platform/2.6/html/containerized_installation/appendix-inventory-files-vars#hub-variables
# -----------------------------------------------------
hub_admin_password=<set your own>
hub_pg_host=aap.example.org
hub_pg_password=<set your own>
hub_seed_collections=false

# Event-Driven Ansible controller
# https://docs.redhat.com/en/documentation/red_hat_ansible_automation_platform/2.6/html/containerized_installation/appendix-inventory-files-vars#event-driven-ansible-variables
# -----------------------------------------------------
eda_admin_password=<set your own>
eda_pg_host=aap.example.org
eda_pg_password=<set your own>
```

3. Run the installer

    ```shell
    export ANSIBLE_COLLECTIONS_PATH="${PWD}"/collections
    ansible-playbook -i <inventory_file_name> ansible.containerized_installer.install
    ```

###Ansible vault tips

Within the ansible installer directory (the dir you extracted), create two dirs: group_vars/all. This is where we will define our sensitive vars referenced via the installer

Now add a secrets.yml group_vars/all/ file via 

```bash
ansible-vault create group_vars/all/secrets.yml
```
```bash
ansible-vault edit group_vars/all/secrets.yml
```
Example secrets.yml file, following yaml variable conventions
```yaml
registry_username: user@example.com
regsitry_password: password
```
```bash
ansible-playbook -i <inventory_file_name> -e @<vault_file_name> --ask-vault-pass -K -v ansible.containerized_installer.install
```
###Troubleshooting

If you see this
```
ERROR! the playbook: ansible.containerized_installer.install.yml could not be found
```
Run the installer with high verbosity to inspect ansible_collection_location. Its likely that the default setting is not looking at the correct.
```
ansible collection location = /home/user/.ansible/collections:/usr/share/ansible/collections
```
The fix: you need to clear this setting so Ansible falls back to using your ansible.cfg file provided via the ansible installer
```bash
unset ANSIBLE_COLLECTIONS_PATH
unset ANSIBLE_COLLECTIONS_PATHS
```





