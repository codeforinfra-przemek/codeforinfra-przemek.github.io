---
layout: default
title: "Ansible – quick guide & pilot repo"
permalink: /ansible-quick-guide.html
---

[← Back to index](/)

# Ansible – quick guide & pilot repo

This page is a short overview of how I use Ansible in my homelab and day-to-day automation work.  
All concrete commands and examples live in a dedicated repository:
➡️ **Course:** Whole idea and knowledge was base on course: https://www.youtube.com/watch?v=goclfp6a2IQ&list=PL2_OBreMn7FqZkvMYt6ATmgC0KAGGJNAN prepared by Jeff Geerling.

➡️ **Repo:** [codeforinfra-przemek/Ansible-pilot](https://github.com/codeforinfra-przemek/Ansible-pilot)  
➡️ **Full cheat sheet:** see the [`README.md`](https://github.com/codeforinfra-przemek/Ansible-pilot/blob/main/README.md)

---

## What this repo covers

The `Ansible-pilot` repository is my personal "pilot cockpit" for Ansible:

- testing host connectivity (`ping`, `win_ping`),
- installing and upgrading packages on Debian-like and RedHat-like systems (`apt`, `yum`),
- editing configuration files safely (`lineinfile`, `blockinfile`),
- managing users, files, directories and permissions (`user`, `file`, `get_url`, `unarchive`),
- reboot orchestration (`reboot`),
- Git operations over HTTPS and SSH (`git`),
- rolling updates with `serial`,
- firewall automation on RedHat-like systems (`firewalld`),
- troubleshooting patterns (indentation errors, connection failures, missing parameters),
- using Ansible Galaxy roles via `requirements.yml`,
- understanding the difference between `ansible-core` and the full community package.

All of these are documented as short, copy-paste friendly snippets in the `README.md` of the repo.

### Ansiblepilot
    Based on: https://www.ansiblepilot.com/
    Personal cheat sheet for common Ansible tasks and troubleshooting.

### Test host availability - Ansible module ping:
    ansible-playbook -i inventory 1_ping.yml 

### How to install Ansible in Ubuntu 20.04 - Ansible install
    #!/bin/bash
    sudo apt update
    sudo apt install ansible
    #!/bin/bash
    sudo apt update
    sudo apt install software-properties-common
    sudo add-apt-repository --yes --update ppa:ansible/ansible
    sudo apt remove ansible
    sudo apt install ansible-base

### Edit multi-line text - Ansible module blockinfile
    - ignore/practice

### Ansible troubleshooting - Indentation error
    - problem with tab or lack of it

### Print text or variable during execution - Ansible module debug
    - use -vv to print variable in verbose 
    ansible-playbook -i inventory -vv 5_print_variable.yml

### Target a specific host or group:
    ansible-playbook -i inventory site.yml --limit webservers
### Increase verbosity (useful for troubleshooting):
    ansible-playbook -i inventory site.yml -v    # verbose
    ansible-playbook -i inventory site.yml -vvv  # very verbose

### Test host availability – ping module:
    ansible -i inventory all -m ping
    ansible-playbook -i inventory 1_ping.yml
## Example 1_ping.yml:
 ```
    ---
- name: Test host availability
  hosts: all
  gather_facts: false

  tasks:
    - name: Ping hosts
      ansible.builtin.ping:
```
### Installing Ansible on Ubuntu 20.04
    sudo apt update
    sudo apt install ansible
### Using official Ansible PPA (newer version)
```
    #!/bin/bash
    set -e
    
    sudo apt update
    sudo apt install -y software-properties-common
    sudo add-apt-repository --yes --update ppa:ansible/ansible
```
### remove old Ansible if needed
    sudo apt remove -y ansible

### install ansible-core / community package
    sudo apt install -y ansible

### Editing multi-line text – blockinfile
```
    ---
- name: Configure SSH banner
  hosts: all
  become: true

  tasks:
    - name: Ensure SSH banner is present
      ansible.builtin.blockinfile:
        path: /etc/issue
        block: |
          ###################################################
          #  Authorized access only.                        #
          #  All activity may be monitored and recorded.    #
          ###################################################
```
### Editing single-line text – lineinfile
```
---
- name: Ensure password authentication is disabled
  hosts: all
  become: true

  tasks:
    - name: Disable PasswordAuthentication in sshd_config
      ansible.builtin.lineinfile:
        path: /etc/ssh/sshd_config
        regexp: '^PasswordAuthentication'
        line: 'PasswordAuthentication no'
        backup: yes
```
### Printing variables – debug module
```
---
- name: Debug variables
  hosts: all

  tasks:
    - name: Print variable
      ansible.builtin.debug:
        var: ansible_hostname

    - name: Print custom message
      ansible.builtin.debug:
        msg: "Deploying to host {{ inventory_hostname }}"
```
### Run with extra verbosity to see more details:
    ansible-playbook -i inventory -vv 5_print_variable.yml

### Package management
```
- name: Install nginx on Debian-like systems
  ansible.builtin.apt:
    name: nginx
    state: present
    update_cache: true
```
### Perform a rolling update (serial):
```
---
- name: Rolling update on Debian-like systems
  hosts: webservers
  become: true
  serial: 2

  tasks:
    - name: Upgrade all packages
      ansible.builtin.apt:
        name: '*'
        state: latest
        update_cache: true

```
### RedHat-like systems – yum / dnf
```
- name: Install httpd on RedHat-like systems
  ansible.builtin.yum:
    name: httpd
    state: present
```
### Rolling update:
```
---
- name: Rolling update on RedHat-like systems
  hosts: webservers
  become: true
  serial: 2

  tasks:
    - name: Upgrade all packages
      ansible.builtin.yum:
        name: '*'
        state: latest
```
### Privilege escalation (become, sudo)
```
---
- name: Run tasks with privilege escalation
  hosts: all
  become: true

  tasks:
    - name: Create a directory with sudo
      ansible.builtin.file:
        path: /opt/myapp
        state: directory
        owner: root
        group: root
        mode: '0755'

```
### Prompt for sudo password when running playbook:
    ansible-playbook -i inventory debug_missing_sudo.yml -bK

### Git operations – git module
Checkout via HTTPS:
```
- name: Checkout repository via HTTPS
  ansible.builtin.git:
    repo: 'https://github.com/codeforinfra-przemek/Ansible-pilot.git'
    dest: /opt/ansible-pilot
    version: main
```
### Checkout via SSH:
```
- name: Checkout repository via SSH
  ansible.builtin.git:
    repo: 'git@github.com:codeforinfra-przemek/Ansible-pilot.git'
    dest: /opt/ansible-pilot
    version: main
    accept_hostkey: yes
```
### Copying files – local, remote, and fetch
Copy from local to remote – copy:
```
- name: Copy local file to remote
  ansible.builtin.copy:
    src: files/app.conf
    dest: /etc/myapp/app.conf
    owner: root
    group: root
    mode: '0644'
```
### Fetch from remote to local – fetch:
```
- name: Fetch log files from remote hosts
  ansible.builtin.fetch:
    src: /var/log/myapp/app.log
    dest: ./collected-logs/
    flat: yes
```
### File management – file module:
```
- name: Create empty file
  ansible.builtin.file:
    path: /tmp/empty.txt
    state: touch
```
### Change file permissions:
```
- name: Set file permission
  ansible.builtin.file:
    path: /usr/local/bin/script.sh
    mode: '0755'
```
### Delete file or directory:
```
- name: Delete temporary directory
  ansible.builtin.file:
    path: /tmp/old-data
    state: absent
```
### Downloading files – get_url:
```
- name: Download file from URL
  ansible.builtin.get_url:
    url: https://example.com/app.tar.gz
    dest: /tmp/app.tar.gz
    mode: '0644'
```
### Extracting archives – unarchive:
```
- name: Extract application archive
  ansible.builtin.unarchive:
    src: /tmp/app.tar.gz
    dest: /opt/app
    remote_src: yes
```
### Rolling Update RedHat like systems - Ansible module yum
    - problem with mirror on CentOS:
      solution:  https://techglimpse.com/failed-metadata-repo-appstream-centos-8/
      
### Rebooting hosts – reboot module
```
---
- name: Reboot servers after kernel update
  hosts: all
  become: true

  tasks:
    - name: Reboot and wait for hosts to come back
      ansible.builtin.reboot:
        reboot_timeout: 600
        msg: "Reboot triggered by Ansible"
```

### User management – user module
### Create a user:
```
- name: Create user account
  ansible.builtin.user:
    name: deploy
    comment: "Deployment user"
    shell: /bin/bash
    groups: sudo
    state: present
```
### Remove a user:
```
- name: Remove user account
  ansible.builtin.user:
    name: olduser
    state: absent
    remove: yes    # also remove home directory
```
### Windows host availability – win_ping
    ansible -i inventory windows -m win_ping
Example inventory group:
```
[windows]
winhost1 ansible_host=192.0.2.10
```
### Firewall management – firewalld (RedHat-like)
```
- name: Open HTTP port in firewalld
  ansible.posix.firewalld:
    service: http
    permanent: true
    state: enabled
    immediate: true
```
### Connection failed – troubleshooting
    ansible -i inventory all --list-hosts
    ssh user@remote-host
    python3 --version
    ansible-playbook -i inventory site.yml -vvv

### Ansible troubleshooting – common issues
Indentation error
    Typical message: mapping values are not allowed here or similar.
    Fix: replace tabs with spaces and use 2 spaces per level consistently.

Missing module parameter
    Message example: missing required arguments: name.
Fix: read module docs:
    ansible-doc ansible.builtin.apt

Failure downloading / repository issues
    For example, CentOS 8 appstream metadata problems:
    often solved by updating mirror configuration or switching to a supported repository.

Syntax error 
Run a dry syntax check:
    ansible-playbook -i inventory playbook.yml --syntax-check

"not a valid attribute for a Play" error
Usually means a task-level key is at the wrong indentation level
(e.g. tasks: is missing, or name: is at the play level instead of under tasks:).
Template error while templating string
Typical reasons:
- unmatched {{ / }}
- wrong variable name
- Jinja expression not valid
Use debug and carefully check Jinja synta

### Downloading and using Ansible Galaxy roles
Install a role with requirements.yml

requirements.yml:
```
---
roles:
  - name: lucab85.ansible_role_log4shell
```

Install roles:
    ansible-galaxy install -r requirements.yml

Use role inside playbook
```
---
- name: Apply log4shell mitigation
  hosts: all
  become: true

  roles:
    - role: lucab85.ansible_role_log4shell
```
### Ansible terminology – ansible-core vs community package

ansible-core
- The core Ansible engine and minimal built-in modules/plugins.
- You add Collections as needed.
- Released a few times per year.
- Good when you want a small, controlled base.
Ansible community package
- Meta-package that includes ansible-core + many community Collections.
- Provides hundreds of modules out of the box.
- Great for quick start and lab environments.
In many modern setups you install ansible-core and then explicitly add Collections you need, especially in larger or more controlled environments.



