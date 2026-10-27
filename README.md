# Ansible Configuration Management

Automate Linux server configuration and static website deployment using Ansible on **Red Hat Enterprise Linux (RHEL) 10.1**.

This project uses Ansible playbooks, roles, tasks, handlers, and inventory to configure a remote server consistently and repeatably.

## Roadmap.sh Project

This project is based on the roadmap.sh Configuration Management challenge.

🔗 https://roadmap.sh/projects/configuration-management

## Project Overview

The goal of this project is to automate the setup of a Linux web server using Ansible. Instead of manually installing packages and configuring services, Ansible executes reusable tasks on a remote server through SSH.

### Technologies Used

- **Ansible** — configuration management and automation
- **Red Hat Enterprise Linux 10.1** — managed server operating system
- **AWS EC2** — remote Linux server
- **Nginx** — web server for hosting a static website
- **YAML** — playbooks, role tasks, and handlers
- **SSH** — secure communication between the control node and managed server
- **Git and GitHub** — version control and project hosting

## Project Structure

```text
ansible-setup/
├── ansible.cfg
├── inventory.ini
├── setup.yml
├── README.md
└── roles/
    ├── base/
    │   └── tasks/
    │       └── main.yml
    ├── nginx/
    │   ├── tasks/
    │   │   └── main.yml
    │   └── handlers/
    │       └── main.yml
    ├── app/
    │   ├── tasks/
    │   │   └── main.yml
    │   └── files/
    │       └── website.tar.gz
    └── ssh/
        ├── tasks/
        │   └── main.yml
        └── files/
            └── authorized_key.pub
```

## Roles and Responsibilities

### 1. Base Role

The `base` role prepares the RHEL server by:

- Refreshing DNF package metadata.
- Updating installed packages.
- Configuring the EPEL repository.
- Installing common utilities such as `curl`, `wget`, `git`, `unzip`, `tar`, and `vim-enhanced`.
- Installing Fail2ban.
- Enabling and starting the Fail2ban service.

### 2. Nginx Role

The `nginx` role installs and configures the web server by:

- Installing Nginx using the DNF package manager.
- Enabling Nginx to start automatically at boot.
- Ensuring the Nginx service is running.
- Configuring the document root to `/var/www/html`.
- Using an Ansible handler to restart Nginx when its configuration changes.

### 3. App Role

The `app` role deploys the static website by:

- Ensuring `/var/www/html` exists.
- Extracting `website.tar.gz` from the role's files directory.
- Setting the website files' ownership to `nginx:nginx`.
- Deploying the custom `index.html` page.

The deployed website displays:

> Hello from Ansible!

### 4. SSH Role

The `ssh` role manages SSH public-key authorization for the intended user on the managed server. This allows key-based SSH authentication without relying on password authentication for that key.

The public key is stored in the role's files directory. The target username should match the account used on the RHEL server, such as `ec2-user`.

## Ansible Configuration

### Inventory

The `inventory.ini` file defines the managed server and its connection details.

Example:

```ini
[webservers]
web1 ansible_host=YOUR_SERVER_IP ansible_user=ec2-user
```

Replace `YOUR_SERVER_IP` with the public or private IP address reachable from the Ansible control node.

Ensure your SSH private key and connection settings are configured correctly.

### Ansible Configuration File

The `ansible.cfg` file can define the default inventory and role locations.

```ini
[defaults]
inventory = ./inventory.ini
roles_path = ./roles
```

### Main Playbook

The `setup.yml` playbook runs the roles against the `webservers` inventory group.

```yaml
---
- name: Configure Linux web server
  hosts: webservers
  become: true
  gather_facts: true

  roles:
    - role: base
      tags:
        - base

    - role: nginx
      tags:
        - nginx

    - role: app
      tags:
        - app

    - role: ssh
      tags:
        - ssh
```

The `become: true` setting enables privilege escalation for tasks requiring administrative permissions.

## How to Run the Project

### 1. Verify the inventory

```bash
cd /root/ansible-setup
ansible-inventory --list
```

### 2. Test SSH connectivity

```bash
ansible webservers -m ping
```

A successful response should include `pong`.

### 3. Validate the playbook syntax

```bash
ansible-playbook setup.yml --syntax-check
```

### 4. Run the complete configuration

```bash
ansible-playbook setup.yml
```

### 5. Run individual roles using tags

Run only the base configuration:

```bash
ansible-playbook setup.yml --tags base
```

Install and configure Nginx:

```bash
ansible-playbook setup.yml --tags nginx
```

Deploy the static website:

```bash
ansible-playbook setup.yml --tags app
```

Configure SSH public-key authorization:

```bash
ansible-playbook setup.yml --tags ssh
```

## Verify the Deployment

Check that Nginx is running:

```bash
ansible webservers -b -m command -a "systemctl status nginx"
```

Check the deployed website files:

```bash
ansible webservers -b -m command -a "ls -la /var/www/html"
```

Inspect the deployed HTML:

```bash
ansible webservers -b -m command -a "cat /var/www/html/index.html"
```

Check Fail2ban:

```bash
ansible webservers -b -m command -a "systemctl status fail2ban"
```

Open the server's public IP in a browser:

```text
http://YOUR_SERVER_IP/
```

Ensure the EC2 security group allows inbound HTTP traffic on port `80`.

## Idempotency

Ansible is designed to be idempotent. Running the playbook repeatedly should not make unnecessary changes when the server is already in the desired state.

For example, if Nginx is installed and running and the document root is already configured correctly, Ansible should report those tasks as `ok` rather than `changed`.

Handlers such as `Restart Nginx` run only when a task notifies them because a change has occurred.

## Key Learnings

Through this project, I practised:

- Automating Linux server configuration with Ansible.
- Creating and organizing reusable Ansible roles.
- Managing remote servers through SSH.
- Using inventory files and playbook tags.
- Installing software on RHEL with DNF.
- Configuring and managing Linux services with systemd.
- Deploying static websites with Nginx.
- Managing file ownership and permissions.
- Using handlers to restart services after configuration changes.
- Understanding idempotency and repeatable infrastructure configuration.

## Future Improvements

- Use Ansible variables to make the configuration reusable across multiple servers.
- Use Ansible Vault to protect sensitive information.
- Add handlers and validation checks for configuration changes.
- Add automated testing and deployment verification.
- Integrate the playbook with a CI/CD pipeline.

## Author

**Amal Dev P V**

GitHub: https://github.com/amaldev-devops

Reference: https://roadmap.sh/projects/configuration-management

---

*This project was created for hands-on DevOps learning and to practise infrastructure configuration and automation with Ansible.*

