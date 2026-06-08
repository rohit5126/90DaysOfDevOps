# Git Commands Reference

## Repository Setup

| Command | Purpose |
|----------|---------|
| `git init` | Create a new Git repository |
| `git clone <url>` | Clone an existing repository |
| `git config --global user.name "Name"` | Set username |
| `git config --global user.email "email@example.com"` | Set email |
| `git config --list` | Show configuration |

---

## Checking Status & History

| Command | Purpose |
|----------|---------|
| `git status` | Show working tree status |
| `git log` | View commit history |
| `git log --oneline` | Compact commit history |
| `git log --graph --all --decorate` | Visual branch history |
| `git show <commit>` | Show commit details |
| `git diff` | Show unstaged changes |
| `git diff --staged` | Show staged changes |
| `git blame <file>` | See who changed each line |

---

## Staging Changes

| Command | Purpose |
|----------|---------|
| `git add <file>` | Stage a file |
| `git add .` | Stage all changes |
| `git add -A` | Stage all changes including deletions |
| `git add -p` | Stage changes interactively |
| `git restore --staged <file>` | Unstage a file |

---

## Committing

| Command | Purpose |
|----------|---------|
| `git commit -m "message"` | Create a commit |
| `git commit -am "message"` | Stage tracked files and commit |
| `git commit --amend` | Modify last commit |
| `git commit --amend --no-edit` | Add changes to last commit without changing message |

---

## Branching

| Command | Purpose |
|----------|---------|
| `git branch` | List branches |
| `git branch <name>` | Create a branch |
| `git switch <branch>` | Switch branches |
| `git switch -c <branch>` | Create and switch to a branch |
| `git checkout <branch>` | Older way to switch branches |
| `git branch -d <branch>` | Delete merged branch |
| `git branch -D <branch>` | Force delete branch |

---

## Merging

| Command | Purpose |
|----------|---------|
| `git merge <branch>` | Merge branch into current branch |
| `git merge --no-ff <branch>` | Force a merge commit |
| `git merge --abort` | Cancel merge in progress |

---

## Rebasing

| Command | Purpose |
|----------|---------|
| `git rebase <branch>` | Rebase current branch |
| `git rebase -i HEAD~N` | Interactive rebase |
| `git rebase --continue` | Continue rebase |
| `git rebase --abort` | Cancel rebase |

---

## Remote Repositories

| Command | Purpose |
|----------|---------|
| `git remote -v` | List remotes |
| `git remote add origin <url>` | Add remote |
| `git fetch` | Download changes |
| `git pull` | Fetch and merge |
| `git pull --rebase` | Fetch and rebase |
| `git push` | Push commits |
| `git push -u origin <branch>` | Set upstream branch |
| `git push --force-with-lease` | Safer force push |

---

## Undoing Changes

| Command | Purpose |
|----------|---------|
| `git restore <file>` | Discard working directory changes |
| `git restore .` | Restore all files |
| `git reset HEAD <file>` | Unstage file |
| `git reset --soft HEAD~1` | Undo commit, keep changes staged |
| `git reset --mixed HEAD~1` | Undo commit, unstage changes |
| `git reset --hard HEAD~1` | Remove commit and changes |
| `git revert <commit>` | Create a commit that undoes another commit |

---

## Stashing

| Command | Purpose |
|----------|---------|
| `git stash` | Save uncommitted changes |
| `git stash push -m "message"` | Create named stash |
| `git stash list` | List stashes |
| `git stash pop` | Apply and remove stash |
| `git stash apply` | Apply stash without removing |
| `git stash drop` | Delete stash |

---

## Tags

| Command | Purpose |
|----------|---------|
| `git tag` | List tags |
| `git tag v1.0` | Create lightweight tag |
| `git tag -a v1.0 -m "release"` | Create annotated tag |
| `git push origin v1.0` | Push tag |
| `git push origin --tags` | Push all tags |

---

## Viewing Information

| Command | Purpose |
|----------|---------|
| `git ls-files` | List tracked files |
| `git reflog` | Show reference history |
| `git shortlog` | Summarized commit history |
| `git describe` | Human-readable commit identifier |

---

## Cleaning

| Command | Purpose |
|----------|---------|
| `git clean -n` | Preview files to delete |
| `git clean -f` | Remove untracked files |
| `git clean -fd` | Remove untracked files and directories |
| `git clean -fdx` | Remove ignored files too |

---

## Cherry-Picking

| Command | Purpose |
|----------|---------|
| `git cherry-pick <commit>` | Apply a commit from another branch |
| `git cherry-pick --continue` | Continue after conflict |
| `git cherry-pick --abort` | Cancel cherry-pick |

---

## Useful Inspection Commands

| Command | Purpose |
|----------|---------|
| `git branch -vv` | Branch tracking information |
| `git remote show origin` | Detailed remote information |
| `git rev-parse HEAD` | Current commit hash |
| `git diff branch1..branch2` | Compare branches |
| `git log branch1..branch2` | Compare branch histories |

---

## Git Worktrees

| Command | Purpose |
|----------|---------|
| `git worktree list` | List worktrees |
| `git worktree add ../folder branch` | Create a new worktree |
| `git worktree remove ../folder` | Remove worktree |

---

# Most Common Daily Workflow

```bash
git status
git pull
git switch -c feature-branch
git add .
git commit -m "Add feature"
git push -u origin feature-branch
```

---

# Top 20 Git Commands to Master

```bash
git init
git clone
git status
git add
git commit
git log
git diff
git branch
git switch
git checkout
git merge
git rebase
git fetch
git pull
git push
git stash
git reset
git restore
git revert
git cherry-pick
```

---

# Advanced Git Commands

```bash
git bisect
git reflog
git cherry-pick
git rebase -i
git stash
git worktree
git blame
git clean
git reset
git revert
```
