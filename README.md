# Introduction

If you have several Palo Alto firewalls, you may want to back up their configurations regularly. This repository contains an Ansible playbook that automates the backup process for Palo Alto devices.

## Prerequisites

- Ansible installed on your control machine (Linux/MacOS/WSL)
- Access to the Palo Alto devices with credentials
- Basic knowledge of Ansible and YAML syntax

## Installation and Setup

For installation, the following commands will update the repository and install Ansible on your Ansible server.

~~~bash
add-apt-repository --yes --update ppa:ansible/ansible
apt install ansible
~~~

We will create a folder to store the working files.

~~~bash
mkdir ansible
~~~

We will create the necessary config file for Ansible Playbooks.

~~~bash
nano ansible.cfg
~~~

Contents to be written inside the config file:

~~~yaml
[defaults]
inventory = inventory.yaml
private_key_file = ~/.ssh/id_ed25519
host_key_checking = False
callback_whitelist = email_playbook_results
~~~

We will create the necessary Inventory files for Ansible Playbooks.

~~~bash
nano inventory.yaml
~~~

Example inventory.yaml file:

~~~yaml
---
paloalto:
  hosts:
    paloalto-test-01:
      datacenter: DC01
      ansible_host: 10.10.10.1
      ansible_port: 443
      panos_username: "USERNAME"
      panos_password: "PASSWORD"

    paloalto-test-02:
      datacenter: DC02
      ansible_host: 10.10.20.1
      ansible_port: 443
      panos_username: "USERNAME"
      panos_password: "PASSWORD"

~~~

## Inventory content explanation

- `paloalto`: The group name for all Palo Alto devices.
- `hosts`: A list of Palo Alto devices to be managed.
- `paloalto-test-01` and `paloalto-test-02`: Hostnames or identifiers for individual Palo Alto devices.
- `datacenter`: A variable that can be used to specify the data center or location of the device.
- `ansible_host`: The IP address or hostname of the Palo Alto device.
- `ansible_port`: The port number for connecting to the Palo Alto device (default is usually 443 for HTTPS).
- `panos_username`: The username for authenticating to the Palo Alto device.
- `panos_password`: The password for authenticating to the Palo Alto device.

## Running the Playbook

To run the playbook and back up the configurations of all Palo Alto devices listed in the inventory file, use the following command:

~~~bash
ansible-playbook -i inventory.yaml paloalto-backup-playbook.yml
~~~

This command will execute the playbook and create backup files for each Palo Alto device in the specified directory.

### Playbook Variables

You can change below variables in the playbook as per your requirements.

~~~bash
  vars:
    dest_path: "/root/{{ datacenter }}"
    folder: "{{ dest_path }}/{{ inventory_hostname }}/{{ hostvars['localhost']['backup_date'] }}"
    filename: "{{ folder }}/backup_{{ hostvars['localhost']['backup_date'] }}_{{ hostvars['localhost']['backup_time'] }}.yaml"
    latest_file: "{{ dest_path }}/{{ inventory_hostname }}/latest/latest.yaml"
~~~

- dest_path: // Base directory where backups will be stored. You can customize it using the datacenter variable.
- folder: // Directory structure for each backup, organized by device hostname and date.
- filename: // Naming convention for the backup files, including date and time.
- latest_file: // Path to the latest backup file for comparison.

You can customize these variables to fit your directory structure and naming preferences.

## Playbook Explanation

In brief, the playbook first checks for the existence of the backup directories and creates them if they do not exist.

Then, it uses the Palo Alto API to take a backup and saves it as **latest**. It also compares the new backup with the previous one and stores the differences in a **compare** file. This way, you can easily see the changes between configurations.

It backs up all VDOMs on the Palo Alto. If desired, you can filter specific VDOMs or mask passwords in the backup. However, if masking is applied, the backup file cannot be directly uploaded in case of an issue.  

## Callback Plugin for Email Notifications

- The repository includes a custom callback plugin (`email_playbook_results.py`) that sends email notifications with the results of playbook executions.
- Update the email addresses and SMTP server details in the plugin as needed.
- Ensure that the callback plugin is placed in the `callback_plugins` directory and that Ansible is configured to use it.

## Security Considerations

- Ensure that sensitive information such as passwords and API keys are managed securely, using Ansible Vault or environment variables.
- Regularly update Ansible and related dependencies to mitigate security vulnerabilities.
- Use secure methods for storing and transmitting backup files, especially if they contain sensitive configuration data.

## Contributions

Contributions to enhance the playbook or add new features are welcome. Please fork the repository and submit a pull request with your changes.
