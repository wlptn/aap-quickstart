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

2. Edit the provided **inventory-growth** file or use the following example if installing locally on a single host. 
  
    **NOTES** 
    - SSH keys are only required when installing on remote hosts. If doing a self contained local VM based installation, you can use ansible_connection=local.
    - You do have the option to leverage ansible-vault to secure sensitive information in your inventory file ([docs with examples](https://docs.redhat.com/en/documentation/red_hat_ansible_automation_platform/2.6/html/tested_deployment_models/container-topologies#cont-a-env-a))

    ```shell
    vim inventory-growth
    ```

```ini
# Common variables
# https://docs.redhat.com/en/documentation/red_hat_ansible_automation_platform/2.6/html/rpm_installation/appendix-inventory-files-vars#ref-general-inventory-variables
# -----------------------------------------------------

redis_mode='standalone'

# AAP Gateway
# https://docs.redhat.com/en/documentation/red_hat_ansible_automation_platform/2.6/html/rpm_installation/appendix-inventory-files-vars#ref-gateway-variables
# -----------------------------------------------------
automationgateway_admin_password=<set your own>
automationgateway_pg_host=db.example.org
automationgateway_pg_password=<set your own>

# AAP Controller
# https://docs.redhat.com/en/documentation/red_hat_ansible_automation_platform/2.6/html/rpm_installation/appendix-inventory-files-vars#ref-controller-variables
# -----------------------------------------------------
admin_password=<set your own>
pg_host=db.example.org
pg_password=<set your own>

# AAP Automation Hub
# https://docs.redhat.com/en/documentation/red_hat_ansible_automation_platform/2.6/html/rpm_installation/appendix-inventory-files-vars#ref-hub-variables
# -----------------------------------------------------
automationhub_admin_password=<set your own>
automationhub_pg_host=db.example.org
automationhub_pg_password=<set your own>

# AAP EDA Controller
# https://docs.redhat.com/en/documentation/red_hat_ansible_automation_platform/2.6/html/rpm_installation/appendix-inventory-files-vars#event-driven-ansible-controller
# -----------------------------------------------------
automationedacontroller_admin_password=<set your own>
automationedacontroller_pg_host=db.example.org
automationedacontroller_pg_password=<set your own>
```

3. Run the installer

    ```shell
    export ANSIBLE_COLLECTIONS_PATH="${PWD}"/collections
    ansible-playbook -i <inventory_file_name> ansible.containerized_installer.install -K -vvv
    ```
