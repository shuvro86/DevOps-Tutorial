# Ansible Comprehensive Learning Guide

## Table of Contents

1. [Overview of Ansible](#overview-of-ansible)
2. [Installation Guide](#installation-guide)
3. [Common Commands](#common-commands)
4. [Deep Dive into Ansible](#deep-dive-into-ansible)
5. [Ansible Vault - Security Management](#ansible-vault---security-management)
6. [Roles Deep Dive](#what-are-roles)
7. [Debug and Troubleshooting](#debug-and-troubleshooting)
8. [Ansible Playbook Writing Strategy](#ansible-playbook-writing-strategy)
9. [Common Workflows](#common-workflows)
10. [Ansible with Terraform](#ansible-with-terraform)
11. [Ansible in CI/CD with GitHub Actions](#ansible-in-cicd-with-github-actions)
12. [Best Practices](#best-practices)
13. [Hands-on Examples](#hands-on-examples)
14. [Resources and Further Learning](#resources-and-further-learning)

---

## Overview of Ansible

### What is Ansible?

Ansible is an open source IT automation engine that automates provisioning, configuration management, application deployment, orchestration, and many other IT processes. It is free to use, and the project benefits from the experience and intelligence of its thousands of contributors.

### Key Features

| Feature | Description |
|---------|-------------|
| **Agentless** | No need to install agents on managed nodes |
| **Simple** | Uses YAML syntax for playbooks |
| **Powerful** | Can manage thousands of nodes simultaneously |
| **Secure** | Uses SSH for communication |
| **Flexible** | Works across multiple platforms and clouds |
| **Idempotent** | Safe to run multiple times |

### Architecture Overview

```mermaid
graph TB
    A[Ansible Control Node] --> B[SSH/WinRM]
    B --> C[Managed Node 1]
    B --> D[Managed Node 2]
    B --> E[Managed Node N]
    
    A --> F[Inventory File]
    A --> G[Playbooks]
    A --> H[Modules]
    A --> I[Plugins]
    
    F --> J[Groups]
    F --> K[Variables]
    
    G --> L[Tasks]
    G --> M[Handlers]
    G --> N[Roles]
```

### Core Components

#### 1. Control Node

The machine where Ansible is installed and from which you run commands and playbooks.

#### 2. Managed Nodes

The target systems that Ansible manages. Also called "hosts."

#### 3. Inventory

A list of managed nodes organized into groups.

#### 4. Modules

Units of code that Ansible executes. Each module has a specific use.

#### 5. Tasks

The units of action in Ansible.

#### 6. Playbooks

Ordered lists of tasks, saved so you can run those tasks in that order repeatedly.

---

## Installation Guide

### System Requirements

| Component | Requirement |
|-----------|-------------|
| **Control Node OS** | Linux, macOS, WSL2 |
| **Managed Node OS** | Linux, Windows, macOS, Network devices |
| **Python** | 3.8+ on control node |
| **SSH** | OpenSSH (for Linux/macOS targets) |
| **WinRM** | For Windows targets |

### Installation Methods

#### Method 1: Using pip (Recommended)

```bash
# Install Python pip if not available
sudo apt update
sudo apt install python3-pip

# Install Ansible
pip3 install ansible

# Verify installation
ansible --version
```

#### Method 2: Using Package Managers

**Ubuntu/Debian:**

```bash
sudo apt update
sudo apt install software-properties-common
sudo add-apt-repository --yes --update ppa:ansible/ansible
sudo apt install ansible
```

**CentOS/RHEL/Fedora:**

```bash
# RHEL/CentOS
sudo yum install epel-release
sudo yum install ansible

# Fedora
sudo dnf install ansible
```

**macOS:**

```bash
# Using Homebrew
brew install ansible

# Using MacPorts
sudo port install ansible
```

#### Method 3: Using Docker

```bash
# Pull official Ansible image
docker pull quay.io/ansible/ansible-runner:latest

# Run Ansible in container
docker run --rm -it \
  -v $(pwd):/workspace \
  quay.io/ansible/ansible-runner:latest \
  ansible --version
```

#### Method 4: From Source

```bash
git clone https://github.com/ansible/ansible.git
cd ansible
python3 -m pip install --user -e .
```

### Post-Installation Configuration

#### Create Ansible Configuration File

```bash
# Create global config
sudo mkdir -p /etc/ansible
sudo nano /etc/ansible/ansible.cfg
```

**Basic ansible.cfg:**

```ini
[defaults]
inventory = /etc/ansible/hosts
host_key_checking = False
timeout = 30
forks = 5
log_path = /var/log/ansible.log

[inventory]
enable_plugins = host_list, script, auto, yaml, ini

[ssh_connection]
ssh_args = -o ControlMaster=auto -o ControlPersist=60s
pipelining = True
```

#### Set up SSH Keys

```bash
# Generate SSH key pair
ssh-keygen -t rsa -b 4096 -C "ansible@controlnode"

# Copy public key to managed nodes
ssh-copy-id user@target-host

# Test connectivity
ssh user@target-host
```

### Verification Tests

```bash
# Test Ansible installation
ansible --version

# Test inventory connectivity
ansible all -m ping

# Test ad-hoc command
ansible all -a "uptime"
```

---

## Common Commands

### Essential Ansible Commands Reference

#### Inventory Commands

```bash
# List all hosts
ansible all --list-hosts

# List hosts in specific group
ansible webservers --list-hosts

# Show inventory details
ansible-inventory --list
ansible-inventory --graph

# Verify inventory
ansible-inventory --host webserver1
```

#### Ad-hoc Commands

Ad-hoc commands are one-liners that let you quickly execute tasks without writing playbooks.

```bash
# Basic syntax
ansible [pattern] -m [module] -a "[module options]"

# Ping all hosts
ansible all -m ping

# Check uptime
ansible all -a "uptime"

# Install package
ansible webservers -m apt -a "name=nginx state=present" --become

# Restart service
ansible webservers -m service -a "name=nginx state=restarted" --become

# Copy file
ansible all -m copy -a "src=/tmp/file.txt dest=/tmp/file.txt"

# Execute shell command
ansible all -m shell -a "df -h"

# Gather facts
ansible all -m setup

# Filter facts
ansible all -m setup -a "filter=ansible_os_family"
```

#### Playbook Commands

```bash
# Run playbook
ansible-playbook playbook.yml

# Run with specific inventory
ansible-playbook -i inventory.ini playbook.yml

# Run with extra variables
ansible-playbook playbook.yml -e "variable=value"

# Dry run (check mode)
ansible-playbook playbook.yml --check

# Show differences
ansible-playbook playbook.yml --diff

# Run specific tags
ansible-playbook playbook.yml --tags "configuration,deployment"

# Skip specific tags
ansible-playbook playbook.yml --skip-tags "testing"

# Limit to specific hosts
ansible-playbook playbook.yml --limit "webserver1,webserver2"

# Step through tasks
ansible-playbook playbook.yml --step

# Start at specific task
ansible-playbook playbook.yml --start-at-task "Install packages"
```

#### Ansible Galaxy Commands

```bash
# Install role from Galaxy
ansible-galaxy install username.rolename

# Install from requirements file
ansible-galaxy install -r requirements.yml

# List installed roles
ansible-galaxy list

# Remove role
ansible-galaxy remove username.rolename

# Initialize new role
ansible-galaxy init my-new-role

# Search for roles
ansible-galaxy search nginx

# Install collection
ansible-galaxy collection install community.general
```

#### Ansible Vault Commands

```bash
# Create encrypted file
ansible-vault create secrets.yml

# Edit encrypted file
ansible-vault edit secrets.yml

# Encrypt existing file
ansible-vault encrypt secrets.yml

# Decrypt file
ansible-vault decrypt secrets.yml

# View encrypted file
ansible-vault view secrets.yml

# Change vault password
ansible-vault rekey secrets.yml

# Run playbook with vault
ansible-playbook playbook.yml --ask-vault-pass
ansible-playbook playbook.yml --vault-password-file ~/.vault_pass
```

#### Configuration and Info Commands

```bash
# Show configuration
ansible-config view

# List configuration options
ansible-config list

# Dump current config
ansible-config dump

# Show documentation for module
ansible-doc copy
ansible-doc -s copy  # Show examples

# List all modules
ansible-doc -l

# Show version info
ansible --version

# Show host facts
ansible hostname -m setup
```

#### Debugging Commands

```bash
# Verbose output (1-4 levels)
ansible-playbook playbook.yml -v
ansible-playbook playbook.yml -vv
ansible-playbook playbook.yml -vvv
ansible-playbook playbook.yml -vvvv

# Syntax check
ansible-playbook playbook.yml --syntax-check

# List tasks
ansible-playbook playbook.yml --list-tasks

# List tags
ansible-playbook playbook.yml --list-tags

# List hosts
ansible-playbook playbook.yml --list-hosts
```

### Command Examples by Use Case

#### System Administration

```bash
# Check disk space
ansible all -a "df -h"

# Check memory usage
ansible all -a "free -m"

# Update package cache
ansible all -m apt -a "update_cache=yes" --become

# Install multiple packages
ansible all -m apt -a "name=vim,git,curl state=present" --become

# Create user
ansible all -m user -a "name=devops shell=/bin/bash" --become

# Set file permissions
ansible all -m file -a "path=/tmp/test.txt mode=644 owner=root group=root" --become
```

#### Service Management

```bash
# Start service
ansible webservers -m service -a "name=apache2 state=started" --become

# Stop service
ansible webservers -m service -a "name=apache2 state=stopped" --become

# Enable service on boot
ansible webservers -m service -a "name=apache2 enabled=yes" --become

# Check service status
ansible webservers -a "systemctl status apache2"
```

#### File Operations

```bash
# Create directory
ansible all -m file -a "path=/opt/myapp state=directory mode=755" --become

# Remove file
ansible all -m file -a "path=/tmp/oldfile state=absent"

# Create symlink
ansible all -m file -a "src=/opt/app dest=/usr/local/bin/app state=link" --become

# Download file
ansible all -m get_url -a "url=http://example.com/file.tar.gz dest=/tmp/"
```

---

## Deep Dive into Ansible

### Inventory Management

#### Static Inventory Example

```ini
[webservers]
web1.example.com
web2.example.com

[databases]
db1.example.com
db2.example.com

[production:children]
webservers
databases

[production:vars]
env=production
```

#### Dynamic Inventory

Dynamic inventories help you easily build your inventory and account for changes made to your infrastructure.

### Playbook Structure

```yaml
---
- name: Configure web servers
  hosts: webservers
  become: yes
  vars:
    http_port: 80
    max_clients: 200
  
  tasks:
    - name: Install Apache
      apt:
        name: apache2
        state: present
      notify: restart apache
    
    - name: Copy configuration file
      template:
        src: httpd.conf.j2
        dest: /etc/apache2/apache2.conf
      notify: restart apache
  
  handlers:
    - name: restart apache
      service:
        name: apache2
        state: restarted
```

### Ansible Vault for Security

Use Ansible Vault to encrypt sensitive data.

```bash
# Create encrypted file
ansible-vault create secrets.yml

# Edit encrypted file
ansible-vault edit secrets.yml

# Run playbook with vault
ansible-playbook playbook.yml --ask-vault-pass
```

### Roles Structure

```sh
roles/
├── webserver/
│   ├── tasks/
│   │   └── main.yml
│   ├── handlers/
│   │   └── main.yml
│   ├── templates/
│   │   └── nginx.conf.j2
│   ├── files/
│   ├── vars/
│   │   └── main.yml
│   ├── defaults/
│   │   └── main.yml
│   └── meta/
│       └── main.yml
```

---

## Ansible Vault - Security Management

### What is Ansible Vault?

Ansible Vault is a feature that allows you to encrypt sensitive data such as passwords, keys, and other secrets within Ansible files. It uses AES256 encryption and integrates seamlessly with playbooks and variables.

### Vault Architecture

```mermaid
graph TD
    A[Plain Text Secrets] --> B[Ansible Vault Encrypt]
    B --> C[Encrypted Vault File]
    C --> D[Version Control Safe]
    
    C --> E[Ansible Vault Decrypt]
    E --> F[Runtime Access]
    F --> G[Playbook Execution]
    
    H[Vault Password] --> B
    H --> E
    
    style A fill:#ffcccc
    style C fill:#ccffcc
    style D fill:#ccccff
```

### Basic Vault Operations

#### Creating Encrypted Files

```bash
# Create new encrypted file
ansible-vault create secrets.yml

# This opens your default editor with empty encrypted file
# Add content like:
database_password: "super_secret_password"
api_key: "abc123def456"
ssh_private_key: |
  -----BEGIN OPENSSH PRIVATE KEY-----
  b3BlbnNzaC1rZXktdjEAAAAABG5vbmUAAAAEbm9uZQAAAAAAAAABAAAAlwAAAAdzc2gtcn
  -----END OPENSSH PRIVATE KEY-----
```

#### Encrypting Existing Files

```bash
# Encrypt existing plain text file
echo "password: secretpass123" > secrets.txt
ansible-vault encrypt secrets.txt

# Encrypt multiple files
ansible-vault encrypt file1.yml file2.yml file3.yml

# Encrypt with specific vault ID
ansible-vault encrypt --vault-id prod@prompt secrets.yml
```

#### Viewing and Editing Vault Files

```bash
# View encrypted file content
ansible-vault view secrets.yml

# Edit encrypted file
ansible-vault edit secrets.yml

# Decrypt file permanently
ansible-vault decrypt secrets.yml

# Show encrypted file (cat equivalent)
cat secrets.yml  # Shows encrypted content
```

### Advanced Vault Features

#### Multiple Vault IDs

```bash
# Create vault with specific ID
ansible-vault create --vault-id dev@prompt dev-secrets.yml
ansible-vault create --vault-id prod@prompt prod-secrets.yml

# Use different vault passwords for different environments
ansible-playbook site.yml --vault-id dev@prompt --vault-id prod@prompt
```

#### Vault Password Files

```bash
# Create password file
echo "my_vault_password" > .vault_pass
chmod 600 .vault_pass

# Use password file
ansible-vault create --vault-password-file .vault_pass secrets.yml
ansible-playbook site.yml --vault-password-file .vault_pass

# Using script for dynamic passwords
#!/bin/bash
# vault_password.sh
echo "$(cat ~/.vault_password_$(whoami))"

chmod +x vault_password.sh
ansible-vault create --vault-password-file vault_password.sh secrets.yml
```

### Vault in Playbooks

#### Using Encrypted Variables

```yaml
# group_vars/all/vault.yml (encrypted)
vault_database_password: "encrypted_secret_password"
vault_api_token: "encrypted_api_token"
vault_ssl_cert: |
  -----BEGIN CERTIFICATE-----
  MIIDXTCCAkWgAwIBAgIJAKoK/heBjcOuMA0GCSqGSIb3DQEBBQUAMEUxCzAJBgNV
  -----END CERTIFICATE-----

# group_vars/all/vars.yml (plain text)
database_password: "{{ vault_database_password }}"
api_token: "{{ vault_api_token }}"
ssl_certificate: "{{ vault_ssl_cert }}"
```

#### Mixed Encrypted/Plain Variables

```yaml
# secrets.yml - partially encrypted
user_name: john_doe  # Plain text
user_password: !vault |
  $ANSIBLE_VAULT;1.1;AES256
  66386439653762356661653
  38653761373863393...
  61623731373765663465

database:
  host: db.example.com  # Plain text
  port: 5432           # Plain text
  password: !vault |   # Encrypted
    $ANSIBLE_VAULT;1.1;AES256
    66386439653762356661653
    38653761373863393...
```

#### Playbook with Vault Variables

```yaml
---
- name: Deploy application with secrets
  hosts: webservers
  become: yes
  vars_files:
    - group_vars/all/vault.yml
    - group_vars/all/vars.yml
  
  tasks:
    - name: Create database configuration
      template:
        src: database.conf.j2
        dest: /etc/myapp/database.conf
        mode: '0600'
      vars:
        db_password: "{{ vault_database_password }}"
    
    - name: Setup API configuration
      lineinfile:
        path: /etc/myapp/api.conf
        line: "API_TOKEN={{ vault_api_token }}"
        create: yes
        mode: '0600'
    
    - name: Install SSL certificate
      copy:
        content: "{{ vault_ssl_cert }}"
        dest: /etc/ssl/certs/myapp.crt
        mode: '0644'
      notify: reload nginx
  
  handlers:
    - name: reload nginx
      service:
        name: nginx
        state: reloaded
```

### Vault Best Practices

#### Directory Structure for Secrets

```sh
group_vars/
├── all/
│   ├── vars.yml          # Plain text variables
│   └── vault.yml         # Encrypted secrets
├── production/
│   ├── vars.yml
│   └── vault.yml
└── staging/
    ├── vars.yml
    └── vault.yml

host_vars/
├── web1.example.com/
│   ├── vars.yml
│   └── vault.yml
```

#### Naming Conventions

```yaml
# Good practice - prefix vault variables
vault_mysql_root_password: "secret123"
vault_ssl_private_key: "-----BEGIN PRIVATE KEY-----"
vault_api_credentials:
  username: "api_user"
  password: "api_secret"

# Reference in plain vars
mysql_root_password: "{{ vault_mysql_root_password }}"
ssl_private_key: "{{ vault_ssl_private_key }}"
```

#### Version Control Integration

```bash
# .gitignore
.vault_pass
**/vault_password*
.ansible_vault_password

# Only commit encrypted files
git add group_vars/all/vault.yml
git add host_vars/*/vault.yml
```

### Automation and CI/CD with Vault

#### Environment Variables

```bash
# Set vault password in environment
export ANSIBLE_VAULT_PASSWORD=my_secret_password

# Use in CI/CD
ansible-playbook site.yml  # Automatically uses env var
```

#### GitHub Actions with Vault

```yaml
name: Deploy with Ansible Vault
on: [push]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Install Ansible
        run: pip install ansible
      
      - name: Create vault password file
        run: echo "${{ secrets.ANSIBLE_VAULT_PASSWORD }}" > .vault_pass
      
      - name: Deploy with vault
        run: |
          ansible-playbook deploy.yml \
            --vault-password-file .vault_pass \
            -i inventory/production
      
      - name: Cleanup
        run: rm -f .vault_pass
```

### Troubleshooting Vault Issues

#### Common Problems and Solutions

```bash
# Problem: "Decryption failed"
# Solution: Check vault password
ansible-vault view secrets.yml --ask-vault-pass

# Problem: "Vault format unhashable type"
# Solution: Check YAML formatting in vault file
ansible-vault edit secrets.yml

# Problem: Mixed vault IDs
# Solution: Specify correct vault ID
ansible-playbook site.yml --vault-id prod@prompt

# Problem: Permission denied
# Solution: Check file permissions
chmod 600 .vault_pass
```

#### Vault Recovery

```bash
# If you forget vault password, you cannot recover the data
# Always backup vault passwords securely

# Rekey vault with new password
ansible-vault rekey secrets.yml

# Change vault ID
ansible-vault encrypt --vault-id new_id@prompt decrypted_file.yml
```

### Security Considerations

1. **Password Management**
   - Use strong, unique passwords for each vault
   - Store passwords in secure password managers
   - Rotate vault passwords regularly

2. **Access Control**
   - Limit access to vault files and passwords
   - Use different vault passwords for different environments
   - Implement proper file permissions (600)

3. **Backup Strategy**
   - Backup encrypted vault files
   - Securely store vault passwords separately
   - Test recovery procedures regularly

4. **Audit Trail**
   - Track who has access to vault passwords
   - Monitor vault file access and modifications
   - Use version control for vault files

### What are Roles?

Roles in Ansible are a way to organize tasks, variables, handlers, and other elements into reusable units. They provide a structured approach to break down complex playbooks into smaller, manageable components that can be shared and reused across different projects.

### Role Architecture

```mermaid
graph TD
    A[Ansible Role] --> B[Tasks]
    A --> C[Handlers]
    A --> D[Variables]
    A --> E[Templates]
    A --> F[Files]
    A --> G[Meta]
    A --> H[Defaults]
    A --> I[Tests]
    
    B --> B1[main.yml]
    B --> B2[install.yml]
    B --> B3[configure.yml]
    
    C --> C1[main.yml]
    
    D --> D1[main.yml]
    
    H --> H1[main.yml]
    
    style A fill:#4CAF50
    style B fill:#2196F3
    style C fill:#FF9800
    style D fill:#9C27B0
```

### Standard Role Directory Structure

```sh
roles/
└── webserver/
    ├── README.md
    ├── defaults/
    │   └── main.yml        # Default variables (lowest precedence)
    ├── files/
    │   ├── index.html      # Files to copy to target hosts
    │   └── ssl_cert.pem
    ├── handlers/
    │   └── main.yml        # Handler definitions
    ├── meta/
    │   └── main.yml        # Role metadata and dependencies
    ├── tasks/
    │   ├── main.yml        # Main task file
    │   ├── install.yml     # Installation tasks
    │   ├── configure.yml   # Configuration tasks
    │   └── debian.yml      # OS-specific tasks
    ├── templates/
    │   ├── nginx.conf.j2   # Jinja2 templates
    │   └── index.html.j2
    ├── tests/
    │   ├── inventory       # Test inventory
    │   └── test.yml        # Test playbook
    └── vars/
        └── main.yml        # Role variables (higher precedence)
```

### Creating Roles

#### Using ansible-galaxy

```bash
# Create new role structure
ansible-galaxy init webserver

# Create role in specific directory
ansible-galaxy init roles/database

# Create role with custom template
ansible-galaxy init --role-skeleton=~/custom-skeleton webserver
```

#### Manual Role Creation

```bash
mkdir -p roles/webserver/{tasks,handlers,templates,files,vars,defaults,meta,tests}
touch roles/webserver/{tasks,handlers,vars,defaults,meta}/main.yml
```

### Role Components Deep Dive

#### 1. Tasks (tasks/main.yml)

```yaml
---
# roles/webserver/tasks/main.yml
- name: Include OS-specific variables
  include_vars: "{{ ansible_os_family }}.yml"
  tags: always

- name: Include OS-specific tasks
  include_tasks: "{{ ansible_os_family }}.yml"
  tags: install

- name: Ensure web server is configured
  include_tasks: configure.yml
  tags: configure

- name: Ensure web server is started
  include_tasks: service.yml
  tags: service
```

```yaml
# roles/webserver/tasks/install.yml
---
- name: Install web server packages
  package:
    name: "{{ webserver_packages }}"
    state: present
  become: yes
  notify: restart webserver

- name: Create web server user
  user:
    name: "{{ webserver_user }}"
    system: yes
    shell: /bin/false
    home: "{{ webserver_home }}"
    create_home: no
  become: yes
```

#### 2. Handlers (handlers/main.yml)

```yaml
---
- name: restart webserver
  service:
    name: "{{ webserver_service_name }}"
    state: restarted
  become: yes

- name: reload webserver
  service:
    name: "{{ webserver_service_name }}"
    state: reloaded
  become: yes

- name: restart firewall
  service:
    name: "{{ firewall_service }}"
    state: restarted
  become: yes
  listen: "restart network services"
```

#### 3. Variables and Defaults

```yaml
# roles/webserver/defaults/main.yml
---
webserver_port: 80
webserver_ssl_port: 443
webserver_user: www-data
webserver_group: www-data
webserver_home: /var/www
webserver_document_root: "{{ webserver_home }}/html"
webserver_log_level: warn
webserver_modules:
  - rewrite
  - ssl
  - headers

# Security settings
webserver_hide_version: true
webserver_server_tokens: false
```

```yaml
# roles/webserver/vars/main.yml
---
# OS-specific packages
webserver_packages:
  Debian:
    - apache2
    - apache2-utils
  RedHat:
    - httpd
    - httpd-tools

webserver_service_name:
  Debian: apache2
  RedHat: httpd

webserver_config_file:
  Debian: /etc/apache2/apache2.conf
  RedHat: /etc/httpd/conf/httpd.conf
```

#### 4. Templates (templates/)

```jinja2
<!-- roles/webserver/templates/index.html.j2 -->
<!DOCTYPE html>
<html>
<head>
    <title>{{ webserver_site_title | default('Welcome') }}</title>
</head>
<body>
    <h1>{{ webserver_site_title | default('Welcome to ' + inventory_hostname) }}</h1>
    <p>Server: {{ inventory_hostname }}</p>
    <p>Environment: {{ environment | default('development') }}</p>
    <p>Last Updated: {{ ansible_date_time.iso8601 }}</p>
    
    {% if webserver_features is defined %}
    <h2>Features:</h2>
    <ul>
    {% for feature in webserver_features %}
        <li>{{ feature }}</li>
    {% endfor %}
    </ul>
    {% endif %}
</body>
</html>
```

```apache
# roles/webserver/templates/vhost.conf.j2
<VirtualHost *:{{ webserver_port }}>
    ServerName {{ webserver_server_name }}
    {% if webserver_server_alias is defined %}
    ServerAlias {{ webserver_server_alias | join(' ') }}
    {% endif %}
    
    DocumentRoot {{ webserver_document_root }}
    
    <Directory {{ webserver_document_root }}>
        Options -Indexes +FollowSymLinks
        AllowOverride All
        Require all granted
    </Directory>
    
    LogLevel {{ webserver_log_level }}
    ErrorLog ${APACHE_LOG_DIR}/{{ webserver_server_name }}_error.log
    CustomLog ${APACHE_LOG_DIR}/{{ webserver_server_name }}_access.log combined
    
    {% if webserver_ssl_enabled | default(false) %}
    # SSL Configuration
    SSLEngine on
    SSLCertificateFile {{ webserver_ssl_cert_file }}
    SSLCertificateKeyFile {{ webserver_ssl_key_file }}
    {% endif %}
</VirtualHost>
```

#### 5. Meta Information (meta/main.yml)

```yaml
---
galaxy_info:
  author: DevOps Team
  description: Web server configuration role
  company: Example Corp
  license: MIT
  min_ansible_version: 2.9
  
  platforms:
    - name: Ubuntu
      versions:
        - 18.04
        - 20.04
        - 22.04
    - name: CentOS
      versions:
        - 7
        - 8
    - name: Debian
      versions:
        - 10
        - 11
  
  galaxy_tags:
    - web
    - apache
    - nginx
    - server

dependencies:
  - role: common
    tags: always
  - role: firewall
    vars:
      firewall_allowed_ports:
        - "{{ webserver_port }}"
        - "{{ webserver_ssl_port }}"
    when: firewall_enabled | default(true)
```

### Using Roles in Playbooks

#### Basic Role Usage

```yaml
---
- name: Configure web servers
  hosts: webservers
  become: yes
  roles:
    - common
    - webserver
    - monitoring
```

#### Role with Variables

```yaml
---
- name: Configure web servers with custom settings
  hosts: webservers
  become: yes
  roles:
    - role: webserver
      vars:
        webserver_port: 8080
        webserver_ssl_enabled: true
        webserver_modules:
          - rewrite
          - ssl
          - headers
          - security
```

#### Conditional Role Execution

```yaml
---
- name: Configure servers based on role
  hosts: all
  become: yes
  roles:
    - role: webserver
      when: "'webservers' in group_names"
    
    - role: database
      when: "'databases' in group_names"
    
    - role: loadbalancer
      when: inventory_hostname in groups['loadbalancers']
```

#### Role with Tags

```yaml
---
- name: Deploy application stack
  hosts: all
  become: yes
  roles:
    - role: common
      tags: [always, common]
    
    - role: webserver
      tags: [web, apache]
    
    - role: database
      tags: [db, mysql]
      when: database_enabled | default(true)
```

### Advanced Role Patterns

#### Role Parameterization

```yaml
# group_vars/webservers.yml
webserver_sites:
  - name: example.com
    document_root: /var/www/example.com
    ssl_enabled: true
    custom_config: |
      Header always set Strict-Transport-Security "max-age=63072000"
  
  - name: api.example.com
    document_root: /var/www/api
    ssl_enabled: true
    proxy_pass: http://backend:3000
```

```yaml
# Role task handling multiple sites
- name: Create virtual host configurations
  template:
    src: vhost.conf.j2
    dest: "/etc/apache2/sites-available/{{ item.name }}.conf"
  loop: "{{ webserver_sites }}"
  notify: reload webserver

- name: Enable virtual hosts
  file:
    src: "/etc/apache2/sites-available/{{ item.name }}.conf"
    dest: "/etc/apache2/sites-enabled/{{ item.name }}.conf"
    state: link
  loop: "{{ webserver_sites }}"
  notify: reload webserver
```

#### Role Composition

```yaml
# Complex playbook with multiple roles
---
- name: Full stack deployment
  hosts: all
  become: yes
  
  pre_tasks:
    - name: Update package cache
      apt:
        update_cache: yes
      when: ansible_os_family == "Debian"
  
  roles:
    # Infrastructure roles
    - role: common
      tags: [always, infrastructure]
    
    - role: security
      tags: [security, infrastructure]
    
    - role: monitoring
      tags: [monitoring, infrastructure]
    
    # Application roles
    - role: webserver
      tags: [web, application]
      when: "'webservers' in group_names"
    
    - role: database
      tags: [db, application]
      when: "'databases' in group_names"
    
    - role: cache
      tags: [cache, application]
      when: "'cache' in group_names"
  
  post_tasks:
    - name: Verify services are running
      service:
        name: "{{ item }}"
        state: started
      loop:
        - "{{ webserver_service_name }}"
        - "{{ database_service_name }}"
      when: item is defined
```

### Role Testing with Molecule

#### Molecule Configuration

```yaml
# roles/webserver/molecule/default/molecule.yml
---
dependency:
  name: galaxy
driver:
  name: docker
platforms:
  - name: instance-ubuntu
    image: ubuntu:20.04
    pre_build_image: true
    command: /lib/systemd/systemd
    privileged: true
    volumes:
      - /sys/fs/cgroup:/sys/fs/cgroup:ro
  - name: instance-centos
    image: centos:8
    pre_build_image: true
    command: /usr/lib/systemd/systemd
    privileged: true
    volumes:
      - /sys/fs/cgroup:/sys/fs/cgroup:ro
provisioner:
  name: ansible
  playbooks:
    converge: converge.yml
verifier:
  name: ansible
```

```yaml
# roles/webserver/molecule/default/converge.yml
---
- name: Converge
  hosts: all
  become: yes
  roles:
    - role: webserver
      vars:
        webserver_port: 8080
        webserver_features:
          - SSL Support
          - Custom Error Pages
          - Security Headers
```

#### Running Tests

```bash
# Install molecule
pip install molecule[docker]

# Create test scenario
cd roles/webserver
molecule init scenario

# Run tests
molecule test

# Test specific platform
molecule test --platform-name instance-ubuntu

# Debug mode
molecule converge
molecule login
molecule verify
```

### Role Best Practices

#### 1. Naming Conventions

- Use descriptive role names: `webserver`, `database`, `monitoring`
- Prefix role variables: `webserver_port`, `database_name`
- Use consistent naming across roles

#### 2. Documentation

```markdown
# roles/webserver/README.md

# Webserver Role

This role installs and configures Apache/Nginx web server.

## Requirements

- Target hosts should have internet connectivity
- Sudo privileges required

## Role Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `webserver_port` | 80 | HTTP port |
| `webserver_ssl_port` | 443 | HTTPS port |
| `webserver_user` | www-data | Web server user |

## Dependencies

- common
- firewall (optional)

## Example Playbook

```yaml
- hosts: webservers
  roles:
    - role: webserver
      vars:
        webserver_port: 8080
```

```

#### 3. Variable Management
- Use `defaults/main.yml` for sensible defaults
- Use `vars/main.yml` for role-specific constants
- Allow override through playbook variables
- Use proper variable precedence

#### 4. Idempotency
```yaml
# Good - Idempotent task
- name: Ensure configuration directory exists
  file:
    path: /etc/myapp
    state: directory
    mode: '0755'

# Avoid - Non-idempotent
- name: Create directory
  shell: mkdir -p /etc/myapp
```

---

## Debug and Troubleshooting

### Debugging Strategies

Debugging is a critical skill when working with Ansible. Here are comprehensive strategies and tools to help identify and resolve issues.

### Verbosity Levels

```bash
# Basic verbosity levels
ansible-playbook playbook.yml -v     # Shows task results
ansible-playbook playbook.yml -vv    # Shows task results and configuration
ansible-playbook playbook.yml -vvv   # Includes connection debugging
ansible-playbook playbook.yml -vvvv  # Adds SSH debugging and more

# Verbose ad-hoc commands
ansible all -m ping -v
ansible all -m setup -vv
```

### Debug Module

The debug module is your primary tool for troubleshooting variables and task execution.

#### Basic Debug Usage

```yaml
---
- name: Debug demonstration
  hosts: localhost
  vars:
    my_variable: "Hello World"
    my_list:
      - item1
      - item2
      - item3
    my_dict:
      key1: value1
      key2: value2
  
  tasks:
    - name: Debug simple variable
      debug:
        var: my_variable
    
    - name: Debug with custom message
      debug:
        msg: "The value of my_variable is {{ my_variable }}"
    
    - name: Debug complex variables
      debug:
        var: my_dict
    
    - name: Debug with verbosity control
      debug:
        msg: "This only shows with -vv or higher"
        verbosity: 2
    
    - name: Conditional debug
      debug:
        msg: "This host is {{ inventory_hostname }}"
      when: inventory_hostname == "localhost"
```

#### Advanced Debug Techniques

```yaml
---
- name: Advanced debugging techniques
  hosts: all
  gather_facts: yes
  
  tasks:
    - name: Debug all variables for host
      debug:
        var: hostvars[inventory_hostname]
      when: debug_mode | default(false)
    
    - name: Debug filtered facts
      debug:
        var: ansible_facts['networking']['interfaces']
    
    - name: Debug loop items
      debug:
        msg: "Processing item: {{ item }}"
      loop:
        - web1
        - web2
        - db1
    
    - name: Debug conditional execution
      debug:
        msg: "This is a {{ ansible_os_family }} system"
      when: ansible_os_family is defined
    
    - name: Debug variable types
      debug:
        msg: |
          Variable type: {{ my_var | type_debug }}
          Variable value: {{ my_var }}
      vars:
        my_var: "{{ some_variable | default('undefined') }}"
```

### Common Debugging Scenarios

#### 1. Variable Debugging

```yaml
---
- name: Variable debugging scenarios
  hosts: localhost
  vars:
    undefined_var: "{{ some_undefined_var | default('fallback_value') }}"
  
  tasks:
    - name: Check if variable is defined
      debug:
        msg: "Variable is defined: {{ my_var is defined }}"
      vars:
        my_var: "test"
    
    - name: Debug variable precedence
      debug:
        msg: |
          Group vars: {{ group_var | default('not set') }}
          Host vars: {{ host_var | default('not set') }}
          Play vars: {{ play_var | default('not set') }}
    
    - name: Debug environment variables
      debug:
        var: ansible_env
    
    - name: Check variable content
      debug:
        msg: |
          Is empty: {{ my_string == '' }}
          Is none: {{ my_string is none }}
          Length: {{ my_string | length }}
      vars:
        my_string: "{{ some_var | default('') }}"
```

#### 2. Task Execution Debugging

```yaml
---
- name: Task execution debugging
  hosts: all
  
  tasks:
    - name: Debug task results
      shell: uptime
      register: uptime_result
    
    - name: Show command output
      debug:
        var: uptime_result
    
    - name: Debug return codes
      debug:
        msg: |
          Return code: {{ uptime_result.rc }}
          stdout: {{ uptime_result.stdout }}
          stderr: {{ uptime_result.stderr }}
    
    - name: Debug failed tasks
      fail:
        msg: "This task failed intentionally"
      when: debug_failures | default(false)
      ignore_errors: yes
      register: failed_task
    
    - name: Debug failure information
      debug:
        var: failed_task
      when: failed_task is failed
```

#### 3. Template and File Debugging

```yaml
---
- name: Template debugging
  hosts: localhost
  vars:
    config_items:
      - name: item1
        value: value1
      - name: item2
        value: value2
  
  tasks:
    - name: Debug template variables
      debug:
        msg: |
          Template will use:
          {% for item in config_items %}
          - {{ item.name }}: {{ item.value }}
          {% endfor %}
    
    - name: Create template and debug
      template:
        src: config.j2
        dest: /tmp/debug_config
      register: template_result
    
    - name: Show template result
      debug:
        var: template_result
    
    - name: Show generated file content
      debug:
        var: lookup('file', '/tmp/debug_config')
```

### Error Analysis and Resolution

#### Common Error Patterns

```yaml
# 1. Undefined Variable Error
- name: Handle undefined variables
  debug:
    msg: "{{ undefined_var | default('Variable not defined') }}"

# 2. Type Errors
- name: Ensure correct variable types
  debug:
    msg: "Port: {{ webserver_port | int }}"
  vars:
    webserver_port: "80"  # String that should be int

# 3. SSH Connection Issues
- name: Test connectivity
  ping:
  register: ping_result
  ignore_errors: yes

- name: Debug connection issues
  debug:
    msg: |
      Connection successful: {{ ping_result is succeeded }}
      Error: {{ ping_result.msg | default('No error') }}
```

### Advanced Troubleshooting Tools

#### 1. Ansible Playbook Debugger

```bash
# Enable debugger on task failure
ansible-playbook playbook.yml --start-at-task="problematic task" --step

# Interactive debugging
# When task fails, you'll enter debugger mode:
# (Pdb) p task_vars['variable_name']
# (Pdb) p result
# (Pdb) c  # continue
# (Pdb) q  # quit
```

#### 2. Custom Debug Callback Plugin

```python
# callback_plugins/debug_callback.py
from ansible.plugins.callback import CallbackBase

class CallbackModule(CallbackBase):
    CALLBACK_VERSION = 2.0
    CALLBACK_TYPE = 'stdout'
    CALLBACK_NAME = 'debug_callback'

    def v2_playbook_on_task_start(self, task, is_conditional):
        self._display.display(f"Starting task: {task.name}")

    def v2_runner_on_ok(self, result):
        self._display.display(f"Task succeeded on {result._host.name}")

    def v2_runner_on_failed(self, result, ignore_errors=False):
        self._display.display(f"Task failed on {result._host.name}: {result._result}")
```

#### 3. Custom Debug Role

```yaml
# roles/debug/tasks/main.yml
---
- name: System Information Debug
  debug:
    msg: |
      Hostname: {{ inventory_hostname }}
      OS Family: {{ ansible_os_family }}
      Distribution: {{ ansible_distribution }} {{ ansible_distribution_version }}
      Kernel: {{ ansible_kernel }}
      Architecture: {{ ansible_architecture }}
      Python Version: {{ ansible_python_version }}
      
- name: Network Information Debug
  debug:
    msg: |
      Default IPv4: {{ ansible_default_ipv4.address | default('N/A') }}
      All IPs: {{ ansible_all_ipv4_addresses | join(', ') }}
      
- name: Memory and CPU Debug
  debug:
    msg: |
      Total Memory: {{ (ansible_memtotal_mb / 1024) | round(2) }} GB
      CPU Cores: {{ ansible_processor_vcpus }}
      CPU Model: {{ ansible_processor[2] | default('Unknown') }}
```

### Performance Debugging

#### 1. Task Timing

```yaml
---
- name: Performance debugging
  hosts: all
  tasks:
    - name: Time consuming task
      shell: sleep 5
      register: slow_task
      
    - name: Debug task duration
      debug:
        msg: |
          Task started: {{ slow_task.start }}
          Task ended: {{ slow_task.end }}
          Duration: {{ slow_task.delta }}
```

#### 2. Profiling Callback

```bash
# Enable profiling
export ANSIBLE_CALLBACK_PLUGINS=/path/to/callback_plugins
export ANSIBLE_STDOUT_CALLBACK=profile_tasks

ansible-playbook playbook.yml
```

### Debugging Configuration Issues

#### ansible.cfg Debugging

```ini
[defaults]
# Enable debugging
log_path = /var/log/ansible.log
display_skipped_hosts = true
display_ok_hosts = true
host_key_checking = false

# Connection debugging
[ssh_connection]
ssh_args = -o ControlMaster=auto -o ControlPersist=60s -o UserKnownHostsFile=/dev/null
pipelining = true
```

#### Inventory Debugging

```bash
# Test inventory parsing
ansible-inventory --list
ansible-inventory --graph
ansible-inventory --host hostname

# Debug specific group
ansible webservers --list-hosts
ansible-inventory --list | jq '.webservers'
```

### Troubleshooting Checklist

#### Pre-execution Checks

```bash
# 1. Syntax validation
ansible-playbook playbook.yml --syntax-check

# 2. Dry run
ansible-playbook playbook.yml --check

# 3. List tasks
ansible-playbook playbook.yml --list-tasks

# 4. List hosts
ansible-playbook playbook.yml --list-hosts

# 5. Test connectivity
ansible all -m ping
```

#### Runtime Debugging

```yaml
# Add to playbook for comprehensive debugging
- name: Debug environment
  debug:
    msg: |
      Current user: {{ ansible_user_id }}
      Working directory: {{ ansible_env.PWD }}
      Python executable: {{ ansible_python_interpreter }}
      Ansible version: {{ ansible_version.full }}
```

#### Post-execution Analysis

```bash
# Check logs
tail -f /var/log/ansible.log

# Analyze failed hosts
ansible-playbook playbook.yml --limit @/tmp/failed_hosts

# Retry failed hosts
ansible-playbook playbook.yml --limit @retry_hosts.retry
```

---

### Variable Precedence

```mermaid
graph TD
    A[Extra Vars] --> B[Task Vars]
    B --> C[Block Vars]
    C --> D[Role Vars]
    D --> E[Play Vars]
    E --> F[Host Vars]
    F --> G[Group Vars]
    G --> H[Role Defaults]
    
    style A fill:#ff9999
    style B fill:#ffcc99
    style C fill:#ffff99
    style D fill:#ccff99
    style E fill:#99ffcc
    style F fill:#99ccff
    style G fill:#cc99ff
    style H fill:#ff99cc
```

---

## Ansible Playbook Writing Strategy

### Strategic Approach to Playbook Development

Writing effective Ansible playbooks requires a systematic approach that combines planning, best practices, and iterative development. This section provides a comprehensive strategy for creating maintainable, scalable, and reliable automation.

### Development Lifecycle

```mermaid
graph TD
    A[Requirements Analysis] --> B[Design & Planning]
    B --> C[Environment Setup]
    C --> D[Prototype Development]
    D --> E[Testing & Validation]
    E --> F[Documentation]
    F --> G[Production Deployment]
    G --> H[Monitoring & Maintenance]
    
    E --> D
    H --> B
    
    style A fill:#e1f5fe
    style G fill:#c8e6c9
    style H fill:#fff3e0
```

### Phase 1: Requirements Analysis and Planning

#### 1. Define Objectives

```markdown
## Playbook Requirements Checklist

### Primary Objectives
- [ ] What specific task needs automation?
- [ ] Which systems/environments are targeted?
- [ ] What are the success criteria?
- [ ] What are the rollback requirements?

### Stakeholder Requirements
- [ ] Who are the end users?
- [ ] What permissions are required?
- [ ] What are the security constraints?
- [ ] What are the performance expectations?

### Technical Requirements
- [ ] Operating system compatibility
- [ ] Software dependencies
- [ ] Network requirements
- [ ] Storage requirements
```

#### 2. Infrastructure Mapping

```yaml
# Document your infrastructure
environments:
  development:
    hosts: 3
    os: Ubuntu 20.04
    services: [web, db]
  
  staging:
    hosts: 6
    os: Ubuntu 20.04
    services: [web, db, cache]
  
  production:
    hosts: 20
    os: Ubuntu 22.04
    services: [web, db, cache, monitoring]

dependencies:
  external_services:
    - DNS
    - Load Balancer
    - CDN
  
  internal_dependencies:
    - Database cluster
    - Message queue
    - File storage
```

### Phase 2: Design and Architecture

#### 1. Playbook Structure Design

```mermaid
graph LR
    A[Site.yml] --> B[Environment Playbooks]
    B --> C[Service Playbooks]
    C --> D[Role Composition]
    D --> E[Task Modules]
    
    B --> B1[dev.yml]
    B --> B2[staging.yml]
    B --> B3[prod.yml]
    
    C --> C1[web.yml]
    C --> C2[database.yml]
    C --> C3[monitoring.yml]
    
    D --> D1[Common Role]
    D --> D2[Application Roles]
    D --> D3[Infrastructure Roles]
```

#### 2. Strategy Pattern Selection

| Pattern | Use Case | Example |
|---------|----------|---------|
| **Monolithic** | Simple, single-purpose automation | Basic server setup |
| **Modular** | Complex, multi-component systems | Full application stack |
| **Layered** | Infrastructure + Application separation | Cloud + Config management |
| **Environment-specific** | Different configs per environment | Dev/Staging/Prod variations |

#### 3. Role-Based Architecture

```yaml
# Strategic role organization
roles/
├── infrastructure/          # Infrastructure roles
│   ├── common/             # Base system configuration
│   ├── security/           # Security hardening
│   ├── monitoring/         # System monitoring
│   └── backup/             # Backup configuration
├── middleware/             # Middleware roles
│   ├── webserver/          # Web server (nginx/apache)
│   ├── database/           # Database systems
│   ├── cache/              # Caching systems
│   └── message_queue/      # Message queuing
└── application/            # Application-specific roles
    ├── frontend/           # Frontend applications
    ├── api/                # API services
    └── workers/            # Background workers
```

### Phase 3: Development Strategy

#### 1. Start Small - Build Incrementally

```yaml
# Development progression strategy

# Step 1: Basic connectivity and facts
- name: Phase 1 - Connectivity Test
  hosts: all
  tasks:
    - name: Test connectivity
      ping:
    
    - name: Gather system facts
      setup:

# Step 2: Add basic system configuration
- name: Phase 2 - Basic System Setup
  hosts: all
  become: yes
  tasks:
    - name: Update package cache
      apt:
        update_cache: yes
    
    - name: Install essential packages
      apt:
        name: [curl, wget, vim, git]
        state: present

# Step 3: Add service-specific configuration
# Step 4: Add application deployment
# Step 5: Add monitoring and logging
```

#### 2. Variable Strategy

```yaml
# Variable organization strategy
group_vars/
├── all/
│   ├── common.yml          # Global settings
│   ├── vault.yml           # Encrypted secrets
│   └── versions.yml        # Software versions
├── environment/
│   ├── development.yml     # Dev-specific settings
│   ├── staging.yml         # Staging settings
│   └── production.yml      # Production settings
└── service/
    ├── webservers.yml      # Web server settings
    ├── databases.yml       # Database settings
    └── monitoring.yml      # Monitoring settings
```

#### 3. Task Organization Patterns

##### Pattern 1: Functional Organization

```yaml
# Organize by function
tasks/
├── main.yml                # Main entry point
├── install.yml             # Installation tasks
├── configure.yml           # Configuration tasks
├── deploy.yml              # Deployment tasks
├── backup.yml              # Backup tasks
└── cleanup.yml             # Cleanup tasks
```

##### Pattern 2: Lifecycle Organization

```yaml
# Organize by lifecycle
tasks/
├── main.yml                # Main orchestration
├── pre_install.yml         # Pre-installation
├── install.yml             # Installation
├── post_install.yml        # Post-installation
├── configure.yml           # Configuration
├── start_services.yml      # Service startup
└── validate.yml            # Validation
```

### Phase 4: Implementation Best Practices

#### 1. Idempotency Strategy

```yaml
# Ensure idempotent operations
- name: Good - Idempotent file creation
  copy:
    content: "{{ config_content }}"
    dest: /etc/myapp/config.conf
    backup: yes
  notify: restart service

# Avoid - Non-idempotent operations
- name: Avoid - Command that runs every time
  shell: echo "{{ config_content }}" >> /etc/myapp/config.conf
```

#### 2. Error Handling Strategy

```yaml
# Comprehensive error handling
- name: Critical task with proper error handling
  block:
    - name: Attempt main operation
      service:
        name: "{{ service_name }}"
        state: started
      register: service_result
  
  rescue:
    - name: Log error details
      debug:
        msg: "Service start failed: {{ service_result.msg | default('Unknown error') }}"
    
    - name: Attempt recovery
      include_tasks: service_recovery.yml
      when: auto_recovery_enabled | default(true)
    
    - name: Fail with informative message
      fail:
        msg: "Failed to start {{ service_name }}. Check logs and service configuration."
  
  always:
    - name: Cleanup temporary files
      file:
        path: /tmp/service_install
        state: absent
```

#### 3. Testing Strategy Integration

```yaml
# Built-in testing approach
- name: Deploy application with validation
  block:
    - name: Deploy application
      include_tasks: deploy_app.yml
    
    - name: Wait for service to be ready
      wait_for:
        port: "{{ app_port }}"
        host: "{{ app_host }}"
        timeout: 60
    
    - name: Validate deployment
      uri:
        url: "http://{{ app_host }}:{{ app_port }}/health"
        method: GET
        status_code: 200
      register: health_check
      retries: 5
      delay: 10
    
    - name: Run smoke tests
      include_tasks: smoke_tests.yml
      when: run_smoke_tests | default(true)
```

### Phase 5: Advanced Strategies

#### 1. Multi-Environment Strategy

```yaml
# Environment-aware playbook structure
- name: Multi-environment deployment
  hosts: "{{ target_environment | default('development') }}"
  vars:
    environment_config: "{{ environments[target_environment] }}"
  
  pre_tasks:
    - name: Validate environment configuration
      assert:
        that:
          - target_environment is defined
          - target_environment in environments
        fail_msg: "Invalid or undefined target environment"
    
    - name: Load environment-specific variables
      include_vars: "environments/{{ target_environment }}.yml"
  
  roles:
    - role: common
      vars:
        common_config: "{{ environment_config.common }}"
    
    - role: application
      vars:
        app_config: "{{ environment_config.application }}"
      when: "'application' in environment_config"
```

#### 2. Rollback Strategy

```yaml
# Implement rollback capabilities
- name: Application deployment with rollback
  hosts: webservers
  serial: "{{ deployment_serial | default('30%') }}"
  
  vars:
    deployment_id: "{{ ansible_date_time.epoch }}"
    rollback_enabled: true
  
  tasks:
    - name: Create deployment backup
      include_tasks: create_backup.yml
      when: rollback_enabled
    
    - name: Deploy new version
      block:
        - name: Stop application
          service:
            name: "{{ app_service }}"
            state: stopped
        
        - name: Deploy application
          include_tasks: deploy_application.yml
        
        - name: Start application
          service:
            name: "{{ app_service }}"
            state: started
        
        - name: Validate deployment
          include_tasks: validate_deployment.yml
      
      rescue:
        - name: Rollback on failure
          include_tasks: rollback_deployment.yml
          when: rollback_enabled
        
        - name: Fail deployment
          fail:
            msg: "Deployment failed and rollback completed"
```

#### 3. Performance Optimization Strategy

```yaml
# Performance considerations
- name: Optimized playbook execution
  hosts: webservers
  strategy: free          # Allow parallel execution
  serial: 5               # Process 5 hosts at a time
  gather_facts: no        # Skip facts if not needed
  
  vars:
    ansible_pipelining: true
    ansible_ssh_multiplexing: true
  
  tasks:
    - name: Gather minimal facts
      setup:
        gather_subset:
          - '!all'
          - '!any'
          - network
          - virtual
      when: facts_required | default(false)
    
    - name: Use efficient modules
      package:
        name: "{{ packages }}"
        state: present
      # Instead of looping with individual package tasks
```

### Phase 6: Documentation Strategy

#### 1. Inline Documentation

```yaml
---
# playbook: web_server_deployment.yml
# purpose: Deploy and configure web servers
# author: DevOps Team
# version: 2.1.0
# last_modified: 2024-01-15

- name: Web Server Deployment Playbook
  hosts: webservers
  become: yes
  
  # Variables explanation:
  # webserver_port: HTTP port for web server (default: 80)
  # ssl_enabled: Enable SSL/TLS (default: false)
  # app_version: Application version to deploy
  
  vars:
    webserver_port: 80
    ssl_enabled: false
    deployment_user: deploy
  
  tasks:
    - name: Install web server packages
      # This task installs the required web server packages
      # Supports both Apache and Nginx based on webserver_type variable
      package:
        name: "{{ webserver_packages[webserver_type] }}"
        state: present
      tags: [installation, packages]
```

#### 2. README Template

```markdown
# Playbook Name

## Purpose
Brief description of what this playbook accomplishes.

## Requirements
- Ansible version: 2.9+
- Target OS: Ubuntu 18.04+
- Required privileges: sudo
- Network access: Internet connectivity required

## Variables

### Required Variables
| Variable | Description | Example |
|----------|-------------|---------|
| `app_name` | Application name | `myapp` |
| `app_version` | Version to deploy | `1.2.3` |

### Optional Variables
| Variable | Default | Description |
|----------|---------|-------------|
| `webserver_port` | `80` | HTTP port |
| `ssl_enabled` | `false` | Enable SSL |

## Usage

### Basic Deployment
```bash
ansible-playbook deploy.yml -i inventory/production
```

### With Custom Variables

```bash
ansible-playbook deploy.yml -e "app_version=1.3.0 ssl_enabled=true"
```

## Tags

- `installation`: Package installation tasks
- `configuration`: Configuration tasks  
- `deployment`: Application deployment
- `validation`: Post-deployment validation

## Troubleshooting

Common issues and solutions...

```

### Development Workflow

#### 1. Version Control Strategy

```bash
# Git workflow for playbooks
git/
├── main                    # Stable, production-ready
├── develop                 # Integration branch
├── feature/new-service     # Feature branches
├── hotfix/security-patch   # Emergency fixes
└── release/v2.1.0         # Release preparation

# Commit message convention
git commit -m "feat(webserver): add SSL configuration support

- Add SSL certificate management
- Update template for HTTPS redirect
- Add validation for SSL certificates

Closes #123"
```

#### 2. Testing Workflow

```yaml
# .github/workflows/ansible-test.yml
name: Ansible Testing Pipeline

on: [push, pull_request]

jobs:
  syntax-check:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Setup Ansible
        run: pip install ansible ansible-lint
      - name: Syntax check
        run: ansible-playbook --syntax-check site.yml
      - name: Lint check
        run: ansible-lint .

  molecule-test:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        scenario: [default, ubuntu, centos]
    steps:
      - uses: actions/checkout@v4
      - name: Run Molecule tests
        run: molecule test --scenario-name ${{ matrix.scenario }}
```

### Key Takeaways

1. **Plan Before Code**: Always start with requirements analysis and design
2. **Start Simple**: Build incrementally from basic connectivity to complex automation
3. **Test Early**: Integrate testing from the beginning of development
4. **Document Everything**: Maintain comprehensive documentation for maintainability
5. **Version Control**: Use proper Git workflows for collaboration
6. **Error Handling**: Implement robust error handling and rollback mechanisms
7. **Performance**: Consider execution efficiency and resource usage
8. **Security**: Integrate security practices throughout development
9. **Monitoring**: Include validation and monitoring in your automation
10. **Iterate**: Continuously improve based on feedback and lessons learned

Remember: Ansible is a desired state engine by design. Focus on defining what the end state should look like rather than the specific steps to get there.

---

## Common Workflows

### Infrastructure Provisioning Workflow

```mermaid
flowchart TD
    A[Start] --> B[Plan Infrastructure]
    B --> C[Create Inventory]
    C --> D[Write Provisioning Playbook]
    D --> E[Execute Playbook]
    E --> F[Verify Deployment]
    F --> G[Configure Services]
    G --> H[Deploy Applications]
    H --> I[End]
    
    F --> J[Rollback on Failure]
    J --> C
```

### Application Deployment Workflow

#### Blue-Green Deployment

```yaml
---
- name: Blue-Green Deployment
  hosts: loadbalancer
  tasks:
    - name: Deploy to Green Environment
      include_tasks: deploy_green.yml
    
    - name: Health Check Green
      uri:
        url: "http://{{ green_servers }}/health"
        method: GET
      register: health_check
    
    - name: Switch Traffic to Green
      when: health_check.status == 200
      template:
        src: lb_config.j2
        dest: /etc/nginx/conf.d/upstream.conf
      notify: reload nginx
    
    - name: Decommission Blue
      include_tasks: decommission_blue.yml
```

### Configuration Management Workflow

Ansible automates the testing of disaster recovery procedures, enabling organizations to regularly validate the effectiveness of their recovery plans without extensive manual effort.

#### Common Configuration Tasks

- User management
- Package installation
- Service configuration
- Security hardening
- Monitoring setup

### Multi-Environment Workflow

```mermaid
graph LR
    A[Development] --> B[Staging]
    B --> C[Production]
    
    D[Feature Branch] --> A
    E[Release Branch] --> B
    F[Main Branch] --> C
    
    style A fill:#90EE90
    style B fill:#FFD700
    style C fill:#FF6B6B
```

---

## Ansible with Terraform

### Integration Benefits

Terraform is used to manage your infrastructure, but Ansible is a better option if you want to install and configure software on your compute instances.

#### Comparison Table

| Aspect | Terraform | Ansible |
|--------|-----------|---------|
| **Primary Purpose** | Infrastructure Provisioning | Configuration Management |
| **Language** | HCL (HashiCorp Configuration Language) | YAML |
| **State Management** | State file required | Stateless |
| **Idempotency** | Built-in | Built-in |
| **Agent** | Agentless | Agentless |
| **Best For** | Infrastructure as Code | Application deployment & configuration |

### Integration Patterns

#### Pattern 1: Sequential Execution

```mermaid
sequenceDiagram
    participant Developer
    participant Terraform
    participant Cloud Provider
    participant Ansible
    participant Servers
    
    Developer->>Terraform: terraform apply
    Terraform->>Cloud Provider: Create infrastructure
    Cloud Provider-->>Terraform: Return server details
    Terraform->>Developer: Infrastructure ready
    Developer->>Ansible: ansible-playbook
    Ansible->>Servers: Configure & deploy apps
    Servers-->>Ansible: Configuration complete
```

#### Pattern 2: Terraform Provisioner

```hcl
resource "null_resource" "ansible_provisioner" {
  provisioner "local-exec" {
    command = "ansible-playbook -i '${aws_instance.web.public_ip},' playbook.yml"
    working_dir = "${path.module}/ansible"
  }
  
  depends_on = [aws_instance.web]
}
```

#### Pattern 3: Dynamic Inventory Integration

```yaml
---
- name: Read Terraform State
  hosts: localhost
  tasks:
    - name: Get EC2 instances from Terraform
      terraform:
        project_path: "../terraform"
        state: present
      register: tf_output
    
    - name: Add hosts to inventory
      add_host:
        name: "{{ item.public_ip }}"
        groups: web_servers
      loop: "{{ tf_output.outputs.ec2_instances.value }}"

- name: Configure EC2 instances
  hosts: web_servers
  tasks:
    - name: Install packages
      apt:
        name: nginx
        state: present
```

### Best Integration Practices

1. **Separation of Concerns**
   - Use Terraform for infrastructure
   - Use Ansible for configuration

2. **State Management**
   - Store Terraform state remotely
   - Use Ansible inventory plugins

3. **CI/CD Integration**
   - Run Terraform first
   - Pass outputs to Ansible

---

## Ansible in CI/CD with GitHub Actions

### Overview

GitHub Actions allows users to automate the software development and deployment tasks defined in a GitHub repository. Combining them will enable you to automate and streamline your Ansible deployments, leveraging continuous integration and continuous delivery (CI/CD) principles.

### CI/CD Pipeline Flow

```mermaid
graph LR
    A[Code Push] --> B[GitHub Actions Trigger]
    B --> C[Checkout Code]
    C --> D[Install Dependencies]
    D --> E[Lint Ansible Code]
    E --> F[Test Playbooks]
    F --> G[Deploy to Staging]
    G --> H[Run Tests]
    H --> I[Deploy to Production]
    
    style A fill:#87CEEB
    style I fill:#98FB98
```

### GitHub Actions Workflow Example

```yaml
name: Ansible CI/CD Pipeline

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  lint:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Set up Python
        uses: actions/setup-python@v4
        with:
          python-version: '3.9'
      
      - name: Install Ansible and dependencies
        run: |
          pip install ansible ansible-lint
      
      - name: Lint Ansible playbooks
        run: |
          ansible-lint playbooks/

  test:
    needs: lint
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Install Ansible
        run: |
          pip install ansible
      
      - name: Test playbook syntax
        run: |
          ansible-playbook --syntax-check playbooks/site.yml

  deploy-staging:
    needs: test
    if: github.ref == 'refs/heads/main'
    runs-on: ubuntu-latest
    environment: staging
    steps:
      - uses: actions/checkout@v4
      
      - name: Install Ansible
        run: |
          pip install ansible
      
      - name: Configure SSH
        run: |
          mkdir -p ~/.ssh
          echo "${{ secrets.SSH_PRIVATE_KEY }}" > ~/.ssh/id_rsa
          chmod 600 ~/.ssh/id_rsa
          ssh-keyscan -H ${{ secrets.STAGING_HOST }} >> ~/.ssh/known_hosts
      
      - name: Deploy to staging
        run: |
          ansible-playbook -i inventories/staging playbooks/site.yml
        env:
          ANSIBLE_HOST_KEY_CHECKING: False

  deploy-production:
    needs: deploy-staging
    if: github.ref == 'refs/heads/main'
    runs-on: ubuntu-latest
    environment: production
    steps:
      - uses: actions/checkout@v4
      
      - name: Install Ansible
        run: |
          pip install ansible
      
      - name: Deploy to production
        run: |
          ansible-playbook -i inventories/production playbooks/site.yml
        env:
          ANSIBLE_HOST_KEY_CHECKING: False
```

### Advanced CI/CD Patterns

#### Matrix Strategy for Multiple Environments

```yaml
strategy:
  matrix:
    environment: [dev, staging, prod]
    include:
      - environment: dev
        inventory: inventories/dev
      - environment: staging
        inventory: inventories/staging
      - environment: prod
        inventory: inventories/prod
```

#### Conditional Deployment

```yaml
- name: Deploy based on branch
  if: |
    (github.ref == 'refs/heads/develop' && matrix.environment == 'dev') ||
    (github.ref == 'refs/heads/staging' && matrix.environment == 'staging') ||
    (github.ref == 'refs/heads/main' && matrix.environment == 'prod')
  run: |
    ansible-playbook -i ${{ matrix.inventory }} playbooks/site.yml
```

### Self-Hosted Runners

The main reason for this is the runner has connectivity to the infrastructure you're attempting to automate. You also have the ability to pre-stage any necessary dependencies on the runner.

```yaml
jobs:
  deploy:
    runs-on: self-hosted
    steps:
      - uses: actions/checkout@v4
      - name: Run Ansible playbook
        run: |
          ansible-playbook -i inventory.ini playbook.yml
```

---

## Best Practices

### Project Structure

A well-organized directory structure is essential for maintaining a scalable Ansible project.

```
ansible-project/
├── ansible.cfg
├── inventories/
│   ├── production/
│   │   ├── hosts.yml
│   │   └── group_vars/
│   └── staging/
│       ├── hosts.yml
│       └── group_vars/
├── group_vars/
│   ├── all.yml
│   └── webservers.yml
├── host_vars/
├── roles/
│   ├── common/
│   ├── webserver/
│   └── database/
├── playbooks/
│   ├── site.yml
│   └── deploy.yml
├── files/
├── templates/
└── vars/
```

### Coding Standards

#### Task Naming

```yaml
# Good
- name: Install and start Apache web server
  apt:
    name: apache2
    state: present

# Bad
- apt:
    name: apache2
    state: present
```

#### Variable Naming Conventions

All defaults and all arguments to a role should have a name that begins with the role name to help avoid collision with other names.

```yaml
# Good - Role variables with prefix
webserver_port: 80
webserver_document_root: /var/www/html

# Bad - Generic names
port: 80
document_root: /var/www/html
```

#### Use of Tags

Tags allow you to run specific parts of a playbook.

```yaml
- name: Install web server
  apt:
    name: nginx
    state: present
  tags:
    - webserver
    - installation

- name: Start web server
  service:
    name: nginx
    state: started
  tags:
    - webserver
    - service
```

### Security Best Practices

#### Privilege Escalation

Adhere to security best practices by executing tasks with the least possible privilege. Implement this by specifying 'become: true' at the task level rather than for the entire playbook.

```yaml
# Good - Task-level privilege escalation
- name: Create user account
  user:
    name: appuser
    shell: /bin/bash
  become: yes

- name: Display message
  debug:
    msg: "User created successfully"
  # No privilege escalation needed

# Avoid - Playbook-level privilege escalation for all tasks
```

#### Secrets Management

```yaml
# Use Ansible Vault for sensitive data
database_password: !vault |
  $ANSIBLE_VAULT;1.1;AES256
  66386439653762356661653...
```

### Testing and Quality Assurance

#### Molecule Testing

```yaml
# molecule/default/molecule.yml
dependency:
  name: galaxy
driver:
  name: docker
platforms:
  - name: instance
    image: ubuntu:20.04
    pre_build_image: true
provisioner:
  name: ansible
verifier:
  name: ansible
```

#### Ansible Lint Configuration

```yaml
# .ansible-lint
exclude_paths:
  - .cache/
  - .github/
  - molecule/
  - .venv/

rules:
  - yaml
  - no-tabs
  - indentation
  - trailing-whitespace
```

### Performance Optimization

#### Parallel Execution

```yaml
# ansible.cfg
[defaults]
forks = 50
host_key_checking = False
pipelining = True
```

#### Fact Caching

```yaml
# ansible.cfg
[defaults]
fact_caching = jsonfile
fact_caching_connection = /tmp/facts_cache
fact_caching_timeout = 86400
```

---

## Hands-on Examples

### Example 1: Complete Web Server Setup

```yaml
---
- name: Setup LAMP Stack
  hosts: webservers
  become: yes
  vars:
    mysql_root_password: "{{ vault_mysql_root_password }}"
    
  tasks:
    - name: Update package cache
      apt:
        update_cache: yes
      tags: packages
    
    - name: Install LAMP packages
      apt:
        name:
          - apache2
          - mysql-server
          - php
          - php-mysql
          - libapache2-mod-php
        state: present
      tags: packages
    
    - name: Start and enable services
      service:
        name: "{{ item }}"
        state: started
        enabled: yes
      loop:
        - apache2
        - mysql
      tags: services
    
    - name: Configure MySQL root password
      mysql_user:
        name: root
        password: "{{ mysql_root_password }}"
        login_user: root
        login_password: "{{ mysql_root_password }}"
        check_implicit_admin: yes
      tags: database
    
    - name: Create web application directory
      file:
        path: /var/www/myapp
        state: directory
        owner: www-data
        group: www-data
        mode: '0755'
      tags: filesystem
    
    - name: Deploy application code
      template:
        src: index.php.j2
        dest: /var/www/myapp/index.php
        owner: www-data
        group: www-data
        mode: '0644'
      notify: restart apache
      tags: deployment
  
  handlers:
    - name: restart apache
      service:
        name: apache2
        state: restarted
```

### Example 2: Docker Container Management

```yaml
---
- name: Manage Docker Containers
  hosts: docker_hosts
  become: yes
  tasks:
    - name: Install Docker
      apt:
        name:
          - docker.io
          - docker-compose
        state: present
    
    - name: Start Docker service
      service:
        name: docker
        state: started
        enabled: yes
    
    - name: Pull application image
      docker_image:
        name: "{{ app_image }}"
        source: pull
        force_source: yes
    
    - name: Run application container
      docker_container:
        name: "{{ app_name }}"
        image: "{{ app_image }}"
        state: started
        restart_policy: always
        ports:
          - "80:8080"
        env:
          DATABASE_URL: "{{ database_url }}"
```

### Example 3: Kubernetes Deployment

```yaml
---
- name: Deploy to Kubernetes
  hosts: localhost
  connection: local
  tasks:
    - name: Create namespace
      kubernetes.core.k8s:
        name: myapp
        api_version: v1
        kind: Namespace
        state: present
    
    - name: Deploy application
      kubernetes.core.k8s:
        state: present
        definition:
          apiVersion: apps/v1
          kind: Deployment
          metadata:
            name: myapp-deployment
            namespace: myapp
          spec:
            replicas: 3
            selector:
              matchLabels:
                app: myapp
            template:
              metadata:
                labels:
                  app: myapp
              spec:
                containers:
                - name: myapp
                  image: "{{ app_image }}"
                  ports:
                  - containerPort: 8080
    
    - name: Create service
      kubernetes.core.k8s:
        state: present
        definition:
          apiVersion: v1
          kind: Service
          metadata:
            name: myapp-service
            namespace: myapp
          spec:
            selector:
              app: myapp
            ports:
            - port: 80
              targetPort: 8080
            type: LoadBalancer
```

---

## Resources and Further Learning

### Official Documentation

- [Ansible Documentation](https://docs.ansible.com/)
- [Ansible Galaxy](https://galaxy.ansible.com/)
- [Red Hat Ansible Automation Platform](https://www.redhat.com/en/technologies/management/ansible)

### Community Resources

- [Ansible GitHub Repository](https://github.com/ansible/ansible)
- [Ansible Community Collections](https://github.com/ansible-collections)
- [Ansible Best Practices Guide](https://redhat-cop.github.io/automation-good-practices/)

### Learning Paths

#### Beginner

1. Complete Ansible basics tutorial
2. Write first playbook
3. Learn about inventory and variables
4. Practice with simple automation tasks

#### Intermediate

1. Master roles and collections
2. Learn Ansible Vault for secrets
3. Implement CI/CD with Ansible
4. Practice with cloud providers

#### Advanced

1. Custom module development
2. Complex workflow orchestration
3. Performance optimization
4. Enterprise-scale deployments

### Certification

- **Red Hat Certified Engineer (RHCE)** - Focuses on Ansible automation
- **Red Hat Certified Specialist in Ansible Automation** - Dedicated Ansible certification

### Books and Courses

- "Ansible for DevOps" by Jeff Geerling
- "Learning Ansible 2.7" by Fabio Alessandro Locati
- Various online courses on platforms like Udemy, Coursera, and Red Hat Training

### Key Takeaways

1. **Start Simple**: Begin with basic playbooks and gradually add complexity
2. **Follow Best Practices**: Use proper project structure and naming conventions
3. **Test Everything**: Implement testing with tools like Molecule and ansible-lint
4. **Version Control**: Always keep your automation code in Git
5. **Security First**: Use Ansible Vault and follow security best practices
6. **Community**: Leverage the vast Ansible community and Galaxy collections
7. **Continuous Learning**: Keep up with new features and best practices

Ansible is a desired state engine by design. If you're trying to "write code" in your plays and roles, you're setting yourself up for failure. Our YAML-based playbooks were never meant to be for programming.

Remember: Automation is a journey, not a destination. Start with what you know, automate incrementally, and always keep learning!

---

*This guide provides a comprehensive foundation for learning and implementing Ansible in your DevOps journey. Continue practicing with real-world scenarios and stay engaged with the Ansible community for the latest updates and best practices.*
