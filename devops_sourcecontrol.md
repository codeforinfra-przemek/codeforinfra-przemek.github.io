---
layout: default
title: "DevOps: SourceControl"
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

### Merge: basic commands
```
#Merge another branch into your current branch
git switch main           # or: git checkout main
git merge feature-branch

#Merge with a merge commit (always creates a merge commit)
git merge --no-ff feature-branch

#Fast-forward only (fail if it would create a merge commit)
git merge --ff-only feature-branch

#Abort a merge in progress (e.g., conflicts happened)
git merge --abort

#Continue after resolving conflicts
git add <fixed-files>
git commit                # completes the merge commit
```
### Types of merge (what they mean):

1) Fast-forward merge (FF)

- Happens when your current branch hasn’t diverged and can simply “move forward”.
- No merge commit (by default).
History stays linear.

Command:
```
git merge feature-branch
# or enforce:
git merge --ff-only feature-branch
```
2) 3-way merge (non fast-forward)

When both branches have new commits (diverged).
- Git creates a merge commit (unless you rebase instead).

Combines histories.

Command:
```
git merge feature-branch
# or force a merge commit even if FF is possible:
git merge --no-ff feature-branch
```

3) Squash merge (combine all feature commits into one)

- Doesn’t create a real “merge commit” that ties branch history together.
- Produces one new commit on the target branch.

Command:
```
git merge --squash feature-branch
git commit -m "Add feature X"
```

4) “Ours/Theirs” strategy (special cases)

Useful when you want to resolve conflicts by preferring one side.

Prefer current branch in conflicts:
`git merge -s ours feature-branch`


Prefer their changes for a specific file during conflict:
```
git checkout --theirs path/to/file
git add path/to/file
```

Prefer ours for a specific file:
```
git checkout --ours path/to/file
git add path/to/file
```

### Additional useful command:
```
git log --graph --oneline
git remote -v
git push origin main
git pull origin main
git tag -a v1.0.1 -m 'Payload comparison bug corrected'
git tag

```




