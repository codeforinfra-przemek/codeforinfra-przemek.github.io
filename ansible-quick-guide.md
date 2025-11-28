```markdown
---
layout: default
title: "Ansible – quick guide & pilot repo"
permalink: /ansible-quick-guide.html
---

[← Back to index](/)

# Ansible – quick guide & pilot repo

This page is a short overview of how I use Ansible in my homelab and day-to-day automation work.  
All concrete commands and examples live in a dedicated repository:

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

---

## Example: simple connectivity check

For example, to validate that all hosts in your inventory are reachable and ready:

```bash
ansible -i inventory all -m ping
