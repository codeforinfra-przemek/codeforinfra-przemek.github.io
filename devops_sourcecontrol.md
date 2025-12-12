---
layout: default
title: "Ansible desired state – controller configuration as code"
permalink: /ansible-desired-state.html
---

# DevOps Foundations: DevOps Version Control Concepts and Practices: Introduction to Version Control

### Base commands/setup:
```
git --version
git config --global user.name "Matt Allford"
git config --global user.email "matt@globomantics.com"
git config --global init.defaultBranch main
git config --list --global
git config --list
```
### Config level:

```
# LOCAL (repo) – domyślny poziom, jeśli nie podasz flagi i jesteś w repo
git config --list --local

# GLOBAL (użytkownik)
git config --list --global

# SYSTEM (wszyscy użytkownicy na maszynie)
git config --list --system
```
`git config --show-origin --list` <- check from what file comme specifi value
`git config --show-scope --list` <- show scope on each level
for specific user.name:
```
git config --get --local  user.name
git config --get --global user.name
git config --get --system user.name
```
### Init git project:
```
cd /path/to/your/project
git init
git branch -M main
git add .
git commit -m "Initial commit"
git log
git log --oneline
touch .gitignore #github/gitignore you can fin there template for gitignore
### Create a remote repo (GitHub/GitLab/Bitbucket)
git remote add origin <REPO_URL>
git push -u origin main
```
### Common fixes
```
# If git commit says identity unknown
git config --global user.name "Your Name"
git config --global user.email "you@example.com"
# If remote already exists
git remote set-url origin <REPO_URL>
# If you used master but want main
git branch -M main
git push -u origin main
```













