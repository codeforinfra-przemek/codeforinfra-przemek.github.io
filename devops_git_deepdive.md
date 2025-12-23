---
layout: default
title: "Devops CoreConcepts"
permalink: /ansible-desired-state.html
---

# Git Deep Dive

## Four Areas:

<img width="1209" height="463" alt="image" src="https://github.com/user-attachments/assets/34229513-d2a1-4903-b2bf-b2e7f1fa0855" />

- stash
- working area: Where we work and modify files, git threate it as temporary, 
- index: between working area and repository, also call staging area, `ls .git` show us index file. `git diff` show working area and index. `git diff --cached` comapre index and repository. If there is info `nothing to commit` it mean workingarea&index%repo have the same files. 
- repository: is in .git, if we loook at `ls .git` there is objects, blobs and trees and commits. Can be created&deleted but not change. Commit is pointing on his parents. Each commit i snapshot. And branches are refernce to commit, and commits are link together creating history. Branch is entry point to history of commits. HEAD is current commit. 

Two questions:
- how does this comman move information across the Four Areas?
- How does this command change the Repository?

#### Working Area: Where we work and modify files, 

- `git add` move file to index, and not change repo. And commit move it to repository. Left to right. 
- `switch`/`checkout` - move HEAD and take copy files from repo to working area and index. So head change, so commit change, so we look at different data. 
- `git rm <file>` will remove file from working working area and index. It will ask as to use optio `--force` or `--cache`
- `git reset` - most command create commits and move a branch, reset only move a branch. Reset move where brach is pointing, on what commit exactly. HEAD follow branch. If you use `--hard` reset will copy file from repo to index and working area. `--mixed` copy files from repo to index only, but now working area(mixed is default). `--soft` will not move any files.  If we want to move to old commit as new ones never happen we use reset. Moving branch change history of project, so we need to be carefull like with rebase. Never change history that was share, like on main repo that all team use.
- `git reset HEAD` - will remove file from index, so we will keep it in wokring area but not indes, like nothing was added at all.  `git reset HEAD  --hard` will overwrite also working area.

#### Stash:

- `git stash --include-untracked` - take all data from working area & index, that are not commited and checkout to curent commit. So basically we sae curent work in stash and return to last commit.
- `stash list` & `git stash clear`

#### Working with individual files:

- `git reset HEAD <filename"` we can reset only one file, and ti will copy by default(mixed) this one file from repo to index. As we cannot use --hard for one file we can insted use: `git restore --stage <file> to unstage specific file (move it to working). But in old times people did: `gir checkout HEAD <file>` to copy file from repo to working area and index. Use this with care, as this is destructive command.

#### Commits on part of file:

- `git diff` to see and `git add --patch <file>` - git will look in change, divide them in `hunk` when you choose `[s]` and then you can choose which part/hunk you would like to add. There is more option there so you can choose `?` to see all. Then `git status` will show you 2 the same file to restore/add, more detail will show 'git diff`.

### Explory Past:

```
git log
git log --graph
git log --graph --decorate
git log --graph --decorate --oneline

git show <hash>
git show <HEAD/branch>
git show HEAD^  #show parent commits of head
git show HEAD^^ #show parent parent of head
git show HEAD~2 #show parent parent of head the same as above
git show HEAD~2^2 #show second parent of second commit
git show HEAD@{|1 month ago"} show head oone mont ago

git blam <dir/filr> #show all the line in file and from where they come from(commit)
git diff
git diff HEAD HEAD~2 #comapre head to 2 commits earlier
git diff branch1 branch2

git log --patch
git log --grep apples --oneline
git log --Gapple --online
git log --Gapple --patch #show commit that add or delete those words
git log -3 --oneline # -n show nr of lines/commits
git log HEAD~5..HEAD^ # we use .. to expres range, first use oldest commit so this is little confusing
git log branch1..branch2 --oneline 

```

### Submodules:

- dependecies between projects - base way is to use packet manager like PIP, git give us option to "Nest git project". We can add subproject to out git project using: `git submodule add <gitlink>` then you can go into submodule dir and 'git pull` if there are new commits or change brach.
- `git submodule update --remote--recursive` pull all new staff for all submodels. Keep in mind that submodel are static and git will not pull there automatically or change brach ect. You need to do it. 

### LFS:

- git never forget so if we have big file it become bigger and bigger.
- `git lfs track` - git will downlaod those big file only when we need them. 


