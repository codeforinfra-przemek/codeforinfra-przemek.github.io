---
layout: default
title: "Ansible desired state – controller configuration as code"
permalink: /ansible-desired-state.html
---

[← Back to index](/)

# Ansible desired state – controller configuration as code

Most people first learn Ansible Automation Platform (AAP) through the GUI:  
create a project, define an inventory, add credentials, build a job template, click **Launch**.

This works great for exploration, but it does not scale:

- no code review,
- no clear history of who changed what,
- no easy way to recreate the same automation in another environment.

This article shows how to take **existing GUI-based automation** and turn it into **desired state configuration** for the Ansible Controller itself, using:

- a **Git repository** as the source of truth,
- the `infra.controller_configuration` / `redhat_cop.controller_configuration` collection to manage controller objects as code. :contentReference[oaicite:0]{index=0}  

The result is **Infrastructure-as-Code (IaC)** for your automation platform: projects, job templates, inventories, credentials and schedules all live in Git and are reconciled to the Controller.

Sources:
- https://github.com/redhat-cop/infra.aap_configuration/tree/devel/roles
- https://docs.ansible.com/projects/ansible/latest/os_guide/windows_dsc.html
---

## Desired state in Ansible vs DSC

In the Windows world, *Desired State Configuration (DSC)* is a PowerShell-based framework that lets you describe how a machine should look and then enforces that state. Ansible integrates with DSC through the `ansible.windows.win_dsc` module. :contentReference[oaicite:1]{index=1}  

Ansible itself is also “desired state oriented”:

- each task describes **how a resource should look**,  
- modules are **idempotent** (they do nothing if the desired state already matches reality).

What we are doing here is extending that idea to the **Ansible Automation Platform controller**:

> The controller (AWX / AAP) is just another system whose configuration  
> (projects, job templates, credentials, schedules, inventories) should be managed as code.

---

## From GUI to desired state

I started with a real automation use case:

- a project that runs a backup playbook,
- an inventory with a few hosts (backup server, primary system, repository server),
- multiple credentials,
- a job template,
- two schedules that run the backup daily.

All of this was originally created via the AAP GUI.

The migration path was:

1. **Explore existing objects using the controller’s REST API.**  
2. **Translate those objects into YAML files** that express the desired state.  
3. Use the `infra.controller_configuration` collection to **apply the YAML as code** to the controller. :contentReference[oaicite:2]{index=2}  

---

## Repository structure for controller desired state

The controller configuration lives in a dedicated Git repo.  
A simplified directory layout looks like this:

```text
.
├── collections
│   └── requirements.yml
├── config
│   ├── credentials.yml
│   ├── inventory.yml
│   ├── job_templates.yml
│   ├── projects.yml
│   └── schedules.yml
├── backup_job.yml
├── backup_config_sync.yml
└── sync_controller.yml
```
collections/requirements.yml – collections for controller configuration (e.g. ansible.controller, infra.controller_configuration). 
config/*.yml – desired state for controller objects (credentials, inventory, projects, job templates, schedules).
backup_job.yml – playbook that actually performs the backup (runs on managed hosts).
backup_config_sync.yml / sync_controller.yml – playbooks that apply the controller config-as-code.

### Collections requirements
collections/requirements.yml:
```
---
collections:
  - name: ansible.controller         # or awx.awx, depending on platform
  - name: infra.controller_configuration
    # or: redhat_cop.controller_configuration
```
Install them with:
  ansible-galaxy collection install -r collections/requirements.yml

### Describing credentials as desired state
Instead of creating credentials manually in the UI, we define them in config/credentials.yml:
```
---
controller_credentials:
  - name: "ACME | Backup Server | svc_backup"
    description: "Service account used for backup job"
    credential_type: Machine
    organization: "ACME"
    inputs:
      username: "svc_backup"
    state: present

controller_credential_input_sources:
  - description: "ACME | Backup Server | svc_backup password"
    input_field_name: "password"
    target_credential: "ACME | Backup Server | svc_backup"
    source_credential: "ACME | Vault | backup-password"
    metadata:
      object_query: "safe=backup-password;object=svc_backup"
      object_query_format: "Exact"
    state: present
```
Key points:
- controller_credentials defines the credential object in the controller.
- controller_credential_input_sources shows how to populate the password from an external secrets source (for example a vault integration exposed as a credential in the controller).
- Everything is versioned in Git, reviewed via pull requests, and can be reproduced in another controller.

### Inventory and hosts as code
config/inventory.yml contains both the inventory and hosts:
```
---
controller_inventories:
  - name: "ACME | backup_targets"
    description: "Targets involved in backup workflow"
    organization: "ACME"
    state: present

controller_hosts:
  - name: "backup_server"
    description: "Backup server"
    inventory: "ACME | backup_targets"
    enabled: true
    state: present
    variables:
      ansible_host: 198.51.100.10
      ansible_user: "svc_backup"
      ansible_ssh_transfer_method: "sftp"
      ansible_remote_tmp: "/backups/tmp/.ansible"

  - name: "primary_system"
    description: "Primary system that needs to be backed up"
    inventory: "ACME | backup_targets"
    enabled: true
    state: present
    variables:
      ansible_host: 203.0.113.20
      ansible_user: "svc_backup"
      ansible_ssh_transfer_method: "sftp"
      ansible_remote_tmp: "/tmp/.ansible"

  - name: "repo_server"
    description: "Git/artefact repository server"
    inventory: "ACME | backup_targets"
    enabled: true
    state: present
    variables:
      ansible_connection: "local"
      ansible_user: "automation"
```
Now:
- adding a new host is a Git change, not a click in the GUI,
- differences between environments (lab / staging / prod) can be handled via branches or overlays.

### Projects as code
config/projects.yml:
```
---
controller_projects:
  - name: "ACME | backup_automation"
    description: "Backup automation project"
    organization: "ACME"
    scm_type: "git"
    scm_url: "https://git.example.com/acme/backup-automation.git"
    scm_branch: "main"
    scm_clean: true
    scm_update_on_launch: true
    allow_override: true
    credential: "ACME | Git | svc_git"
    default_environment: "ACME | Default EE"
    update: true
    wait: true
    scm_delete_on_update: false
    state: present
```

### Job templates as desired state
The core of the automation is a job template that runs backup_job.yml.
config/job_templates.yml:
```
---
controller_templates:
  - name: "ACME | Backup | copy_data_to_backup"
    description: "Copy data from primary system to backup server"
    organization: "ACME"
    job_type: "run"
    project: "ACME | backup_automation"
    inventory: "ACME | backup_targets"
    execution_environment: "ACME | Default EE"
    playbook: "backup_job.yml"
    credentials:
      - "ACME | Backup Server | svc_backup"
    verbosity: 1
    state: present
```
This replaces manual GUI configuration with a precise, repeatable definition of:
- which project and playbook to use,
- which inventory and credentials to attach,
- which execution environment to run in.

### Schedules as desired state
Finally, we define controller schedules in config/schedules.yml:
```
---
controller_schedules:
  - name: "Backup job – daily"
    description: "Run backup job once per day"
    unified_job_template: "ACME | Backup | copy_data_to_backup"
    rrule: "DTSTART:20250101T010000Z RRULE:FREQ=DAILY;INTERVAL=1"
    verbosity: 0
    state: present
```

The rrule field uses standard iCalendar RRULE syntax, which makes it easy to reason about frequency and start times.
If the schedule needs to change (time, interval, frequency), the modification is a Git diff, not an ad-hoc click in the UI.

### The controller configuration playbook

All of these YAML files are consumed by a single playbook that applies the desired state to the controller.
sync_controller.yml
```
---
- name: Synchronise Ansible Controller configuration with desired state
  hosts: localhost
  connection: local
  gather_facts: false

  collections:
    - ansible.controller
    - infra.controller_configuration    # or redhat_cop.controller_configuration

  vars_files:
    - config/credentials.yml
    - config/inventory.yml
    - config/projects.yml
    - config/job_templates.yml
    - config/schedules.yml

  roles:
    - role: infra.controller_configuration.dispatch
      # or: redhat_cop.controller_configuration.dispatch
```
Running the playbook:
  `ansible-playbook -i localhost, sync_controller.yml  # personaly i runned it from portal using predefinde in comapny job`

- Re-using the same desired state repo
- Because everything is in code, re-using the same repo across environments is straightforward:
- Same repo, different inventories – use different branches, or different config/inventory.yml files per environment.
- Same structure, different secrets – credentials are defined structurally in Git, but sensitive values are sourced from a vault or separate secret store.
- Promotion flow – changes are developed and tested in a non-production controller, then promoted to production via Git branches and the same sync_controller.yml playbook.

### Exploring existing GUI automation via the API

If you already have automation created in the GUI, the easiest way to convert it to desired state is:
Open the GUI and navigate to the object (credential, project, job template, etc.).
Look at the URL to find its numeric ID, for example:
https://aap.example.com/#/credentials/42/details
Use the controller API to view the JSON representation:
https://aap.example.com/api/v2/credentials/42/
Copy the relevant fields (name, organization, type, SCM URL, playbook name, etc.) into the corresponding YAML file in the config repo.
The API endpoints you will use most often are:
  /api/v2/inventories/
  /api/v2/credentials/
  /api/v2/projects/
  /api/v2/job_templates/
  /api/v2/workflow_job_templates/

This is often the most practical way to reverse-engineer GUI-created automation into a clean desired state configuration.

### Why this matters

Moving controller configuration to desired state (configuration as code) unlocks:

Git history – every change to automation is documented.

Code review – changes go through pull requests instead of “click ops”.

Reproducibility – you can rebuild a controller, or create a new one, purely from code.

Multi-environment consistency – dev / test / prod controllers stay aligned.

Disaster recovery – the controller config becomes part of your standard backup and recovery processes. 
redhat.com
+1

This approach keeps the fast feedback of the GUI for experimentation, while promoting stable automation into a proper Infrastructure-as-Code workflow once you are happy with it.




