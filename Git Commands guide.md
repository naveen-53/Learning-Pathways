# Git Commands Guide (Team Project Workflow)

This guide is a practical **team-focused** Git reference: from setup → daily work → branching → PRs → conflict resolution → undoing mistakes.

---

## Table of Contents
- [1. Initial Setup](#1-initial-setup)
- [2. Create or Clone a Repo](#2-create-or-clone-a-repo)
- [3. Daily Workflow (Most Used)](#3-daily-workflow-most-used)
- [4. Branching Workflow](#4-branching-workflow)
- [5. Pull Requests Workflow](#5-pull-requests-workflow)
- [6. Undo & Fix Mistakes](#6-undo--fix-mistakes)
- [7. Merge Conflicts](#7-merge-conflicts)
- [8. Stash (Save Work Temporarily)](#8-stash-save-work-temporarily)
- [9. History & Logs](#9-history--logs)
- [10. Tags & Releases](#10-tags--releases)
- [11. Cleaning & Maintenance](#11-cleaning--maintenance)
---

## 1. Initial Setup

### Download git
download git use this link - 
[https://git-scm.com/install/windows](https://git-scm.com/install/windows)

### Install & check Git
```bash
git --version
```
### Set your identity
- set your github username
- set your github email
```
git config --global user.name "Your Name"
git config --global user.email "you@email.com"
```

### Set default branch name
- you can set default branch using this command
```
git config --global init.defaultBranch main
```

### Improve terminal output
```
git config --global color.ui auto
```

### View config
- using list you can view all configuration
- using user.name you can view username
```
git config --list
git config user.name
```

---

## 2. Create or Clone a Repo

### Start a new repo
- you can create a new repo using this command
```
git init
```

### Clone an existing repo
```
git clone https://github.com/org/repo.git
```

### Clone into a custom folder name
```
git clone https://github.com/org/repo.git myfolder
```

---
## 3. Daily Workflow

### Check repo status
```
git status
```

### Pull latest changes from remote
```
git pull
```

### See changes in files
```
git diff
```

### Stage files
```
git add file-name
git add .          # all files
```

### Commit changes
```
git commit -m "commit message"
```

### Push your branch
```
git push
git push -u origin your-branch
```

---
## 4. Branching Workflow

### View branches
```
git branch
git branch -a
```

### Create new branch

```
git branch branch-name
```
### Create and switch
```
git checkout -b branch-name
git switch main
git switch -c branch-name
```

### Rename branch
```
git branch -m old-name new-name
```

### Delete local branch
```
git branch -d branch-name
git branch -D branch-name   # force delete
```

### Delete remote branch
```
git push origin --delete branch-name
```
--- 

## 5. Pull Requests Workflow

### Typical team workflow
1. Checkout main
2. Pull latest main
3. switch branch or create a branch
4. Work & commit
5. Push branch
6. Open PR (GitHub/GitLab)
7. Fix PR review requests
8. Merge PR

```
git switch main
git pull --rebase
git switch -c branch-name

# work...
git add .
git commit -m "commit message"

git push -u origin branch-name
```

---

## 6. Undo & Fix Mistakes
### Undo changes in working directory (not staged yet)
```
git restore file-name
git restore .
```

### Unstage a file (keeps changes)
```
git restore --staged file-name
```

### Amend last commit (edit message or add missing files)
```
git add .
git commit --amend -m "commit message"
```

### Revert a commit (safe for shared branch)
```
git revert <commit_hash>
```

### Reset (dangerous if already pushed)
  #### Soft reset (keep changes staged)
  ```
  git reset --soft HEAD~1
  ```

  #### Mixed reset (keep changes unstaged)
  ```
  git reset HEAD~1
  ```

  #### Hard reset (DESTROYS changes)
  ```
  git reset --hard HEAD~1
  ```


### Undo a hard reset using reflog
- reflog is history of where your HEAD and branches have been.
```
git reflog
git reset --hard <hash_from_reflog>
```

---

## 7. Merge Conflicts
- **git rebase** move(replay) your commits on top of another branch. 
### When pulling causes conflicts
```
git pull --rebase
# conflict appears
```

### See conflict files
```
git status
```

### After resolving conflicts manually
  #### For rebase:
  ```
  git add .
  git rebase --continue
  ```

  #### To cancel rebase:
  ```
  git rebase --abort
  ```

  #### For merge:
  ```
  git add .
  git commit
  ```

---

## 8. Stash (Save Work Temporarily)
- **Git stash** is like a temporary drawer where you can quickly save your unfinished wirk without committing it
### Save uncommitted work
```
git stash
```

### Save with message
```
git stash push -m "stash nessage"
```

### View stashes
```
git stash list
```

### Apply stash (keeps stash in list)
```
git stash apply
```

### Pop stash (removes stash from list)
```
git stash pop
```

### Drop a stash
```
git stash drop stash@{0}
```

### Clear all stashes
```
git stash clear
```

## 9. History & Logs
### Show commit history
```
git log
git log --oneline
git log --oneline --graph --decorate --all
```

### View changes in a commit
```
git show <commit_hash>
```

### Blame (who changed a line)
```
git blame file1
```

---

## 10. Tags & Releases

### Create tag
```
git tag v1.0.0
```

### Annotated tag
```
git tag -a v1.0.0 -m "Release version 1.0.0"
```

### Push tags
```
git push origin v1.0.0
git push origin --tags
```

### List tags
```
git tag
```
--- 

## 11. Cleaning & Maintenance
### Remove untracked files (preview first!)
```
git clean -n
```

### Remove untracked files
```
git clean -f
```

### Remove untracked directories too
```
git clean -fd
```

---

## 12. Advanced / Power Commands
### Fetch (download remote updates without merging)
```
git fetch
git fetch --all
```

### Compare local branch with remote
```
git diff origin/main..main
```

### Set upstream
```
git branch --set-upstream-to=origin/main main
```

### List remotes
```
git remote -v
```

### Change remote URL
```
git remote set-url origin https://github.com/org/repo.git
```
### add upstream
```
git remote add upstream url-link
```
