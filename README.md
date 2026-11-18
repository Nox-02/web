# Git and GitHub Guide

Everything you need to work in this repo: plain Git first, then the GitHub workflow on top of it. Commands are for PowerShell on Windows but work the same in any terminal.

## Table of contents

1. [Mental model](#1-mental-model)
2. [One-time setup](#2-one-time-setup)
3. [Everyday Git](#3-everyday-git)
4. [Branches](#4-branches)
5. [Working with remotes](#5-working-with-remotes)
6. [Merge conflicts](#6-merge-conflicts)
7. [Undoing things](#7-undoing-things)
8. [Stash, tags, and ignoring files](#8-stash-tags-and-ignoring-files)
9. [The GitHub workflow](#9-the-github-workflow)
10. [GitHub CLI cheat sheet](#10-github-cli-cheat-sheet)
11. [Troubleshooting](#11-troubleshooting)
12. [Glossary](#12-glossary)
13. [Rules of thumb](#13-rules-of-thumb)

---

## 1. Mental model

Git tracks your project in four places:

```
Working directory  ->  Staging area  ->  Local repo  ->  Remote (GitHub)
 (your files)          (git add)         (git commit)     (git push)
```

- **Working directory**: the files you edit.
- **Staging area (index)**: the changes you picked for the next commit.
- **Local repository**: your full history, stored in the `.git` folder.
- **Remote**: a shared copy, here on GitHub, called `origin`.

A **commit** is a snapshot of the project with a message. A **branch** is just a movable label pointing at a commit. `HEAD` is the commit you are currently on.

## 2. One-time setup

Install Git and the GitHub CLI:

```
winget install Git.Git
winget install GitHub.cli
```

Tell Git who you are (this appears on every commit):

```
git config --global user.name "Your Name"
git config --global user.email "you@example.com"
git config --global init.defaultBranch main
git config --global pull.rebase false
```

Log in to GitHub:

```
gh auth login
```

Get the repo onto your machine:

```
git clone https://github.com/Nox-02/web.git
cd web
```

Check your settings any time with `git config --list`.

## 3. Everyday Git

### The core loop

```
git status                 # what changed?
git diff                   # see unstaged changes line by line
git add file.txt           # stage one file
git add .                  # stage everything
git diff --staged          # review what you are about to commit
git commit -m "feat: add login form"
git log --oneline          # see history
```

### Looking around

| I want to... | Command |
|--------------|---------|
| See changed files | `git status` |
| See unstaged changes | `git diff` |
| See staged changes | `git diff --staged` |
| Compact history | `git log --oneline` |
| History with branch graph | `git log --oneline --graph --all` |
| History of one file | `git log --oneline -- path/to/file` |
| Who changed each line | `git blame path/to/file` |
| Show one commit | `git show <commit>` |
| Search commit messages | `git log --grep="login"` |
| Search code in history | `git log -S"functionName"` |

### Commit messages

Use the format `type: short summary` in the present tense, under about 70 characters.

| Type | Use for |
|------|---------|
| `feat` | a new feature |
| `fix` | a bug fix |
| `docs` | documentation only |
| `chore` | tooling, config, maintenance |
| `refactor` | code change with no behavior change |
| `test` | adding or fixing tests |
| `style` | formatting only |

Good: `fix: handle empty email on signup`
Bad: `stuff`, `update`, `fixed it`

Make small, focused commits. One logical change per commit.

## 4. Branches

Branches let you work on something without touching `main`.

```
git branch                       # list local branches
git branch -a                    # include remote branches
git checkout -b feat/search      # create and switch (older style)
git switch -c feat/search        # create and switch (newer style)
git switch main                  # switch to an existing branch
git branch -d feat/search        # delete after merging
git branch -D feat/search        # force delete (unmerged work is lost)
git branch -m new-name           # rename the current branch
```

Branch naming: `type/short-description`, for example `feat/search-bar`, `fix/login-crash`, `docs/readme`, `chore/ci`.

### Merge vs rebase

Both bring changes from one branch into another.

**Merge** keeps history as it happened and adds a merge commit:

```
git switch main
git merge feat/search
```

**Rebase** replays your commits on top of another branch for a straight-line history:

```
git switch feat/search
git rebase main
```

Rule: never rebase commits that other people already pulled. Rebasing rewrites history, so only do it on your own unpushed branches. If unsure, use merge.

## 5. Working with remotes

```
git remote -v                    # show remotes
git fetch                        # download changes, do not apply them
git pull                         # fetch + merge into current branch
git push                         # upload your commits
git push -u origin HEAD          # first push of a new branch, sets tracking
git push origin --delete old     # delete a remote branch
git fetch --prune                # clean up references to deleted remote branches
```

`fetch` is always safe. `pull` changes your files. When in doubt, `git fetch` then `git status` to see how far behind or ahead you are.

If a push is rejected because the remote has new commits, run `git pull`, resolve any conflicts, then push again.

## 6. Merge conflicts

A conflict happens when two branches changed the same lines. Git stops and marks the file:

```
<<<<<<< HEAD
your version
=======
their version
>>>>>>> feat/search
```

To resolve:

1. Run `git status` to see the conflicted files.
2. Open each file, choose the correct code, and delete the three marker lines.
3. Stage the fixed files: `git add file.txt`
4. Finish: `git commit` (for a merge) or `git rebase --continue` (for a rebase).

To back out and start over:

```
git merge --abort
git rebase --abort
```

VS Code shows Accept Current / Accept Incoming buttons right above the markers.

## 7. Undoing things

Pick the situation:

| Situation | Command |
|-----------|---------|
| Discard edits to a file (not staged) | `git restore file.txt` |
| Unstage a file, keep the edits | `git restore --staged file.txt` |
| Fix the last commit message | `git commit --amend -m "new message"` |
| Add a forgotten file to the last commit | `git add file.txt` then `git commit --amend --no-edit` |
| Undo a commit but keep the changes | `git reset --soft HEAD~1` |
| Undo a commit and unstage the changes | `git reset HEAD~1` |
| Undo a commit and delete the changes | `git reset --hard HEAD~1` |
| Undo a pushed commit safely | `git revert <commit>` |
| Delete untracked files (preview first) | `git clean -n` then `git clean -f` |

Important:

- `git reset --hard` and `git clean -f` permanently delete uncommitted work.
- Do not use `reset` or `--amend` on commits you already pushed to a shared branch. Use `git revert`, which adds a new commit that undoes the old one.

### Safety net: reflog

Git remembers where `HEAD` has been for a while, even after a bad reset:

```
git reflog
git reset --hard HEAD@{2}
```

Find the entry from before the mistake and jump back to it.

## 8. Stash, tags, and ignoring files

### Stash

Park unfinished work so you can switch branches:

```
git stash                    # save changes
git stash -u                 # include untracked files
git stash list
git stash pop                # restore the latest and remove it from the list
git stash apply              # restore but keep it in the list
git stash drop               # delete the latest stash
```

### Tags

Mark a release point:

```
git tag v1.0.0
git tag -a v1.0.0 -m "First release"
git push origin v1.0.0
git tag                      # list tags
```

### .gitignore

List files Git should never track, one pattern per line:

```
node_modules/
.env
*.log
dist/
.DS_Store
Thumbs.db
```

If a file is already tracked, adding it to `.gitignore` is not enough. Untrack it first:

```
git rm --cached path/to/file
```

Never commit secrets (API keys, passwords, `.env` files). If you did, rotate the secret right away. Deleting it in a later commit does not remove it from history.

## 9. The GitHub workflow

This is the flow for all changes to `main`:

```
Issue -> Branch -> Commits -> Push -> Pull request -> CI -> Review -> Squash merge -> Sync
```

### Step by step

**1. Create an issue.** On GitHub go to Issues, then New issue, and pick a template: user story, bug report, or task. Or run `gh issue create`. Note the number, for example `#12`.

**2. Branch from an up-to-date main.**

```
git switch main
git pull
git switch -c feat/short-description
```

**3. Work and commit** (see section 3).

**4. Push and open a pull request.**

```
git push -u origin HEAD
gh pr create
```

Fill in the PR template. Link the issue with `Closes #12` in the description so it closes automatically when merged.

**5. Check CI.**

```
gh pr checks --watch
```

- Green check: ready to merge.
- Red X: run `gh run view --log-failed`, fix the problem, commit, and push. CI re-runs automatically.

**6. Get a review** if your team requires one. Respond to comments by pushing more commits to the same branch.

**7. Squash merge.**

```
gh pr merge --squash --delete-branch
```

Squash merge turns the whole PR into one clean commit on `main`.

**8. Sync locally.**

```
git switch main
git pull
```

### Keeping your branch up to date

If `main` moved while you worked:

```
git fetch
git merge origin/main
```

Fix any conflicts (section 6), then push.

### Good pull requests

- One purpose per PR. Keep it small.
- Write a clear title in commit format: `feat: add search bar`.
- Describe what changed and why, and how you tested it.
- Add screenshots for visual changes.
- Do not merge with failing CI.

## 10. GitHub CLI cheat sheet

| I want to... | Command |
|--------------|---------|
| Check I am logged in | `gh auth status` |
| Create an issue | `gh issue create` |
| List open issues | `gh issue list` |
| View an issue | `gh issue view 12` |
| Create a PR | `gh pr create` |
| Create a PR from commits | `gh pr create --fill` |
| List my PRs | `gh pr list` |
| Open current PR in browser | `gh pr view --web` |
| Check out someone's PR | `gh pr checkout 5` |
| Watch CI | `gh pr checks --watch` |
| See failed CI logs | `gh run view --log-failed` |
| Re-run failed CI | `gh run rerun --failed` |
| Merge (squash) | `gh pr merge --squash --delete-branch` |
| Open the repo in browser | `gh repo view --web` |

## 11. Troubleshooting

**"LF will be replaced by CRLF" warnings.** Harmless. Windows and Linux use different line endings. This repo's `.gitattributes` normalizes them.

**"You have divergent branches" on pull.** Your local branch and the remote both have new commits. Run `git config pull.rebase false` and pull again, or use `git pull --rebase` if you prefer a straight history.

**"Your branch is behind origin/main."** Run `git pull`.

**"Please commit your changes or stash them before you switch branches."** Run `git stash`, switch, then `git stash pop` later.

**Push rejected (non-fast-forward).** The remote has commits you do not. Run `git pull`, resolve conflicts, push again. Avoid `git push --force`. If you must, use `git push --force-with-lease`, and never on `main`.

**Committed to main by accident (not pushed yet).**

```
git branch feat/my-work        # save the commits on a new branch
git reset --hard origin/main   # put main back
git switch feat/my-work
```

**Wrong email or name on commits.** Fix the config (section 2). Past commits keep the old values.

**Detached HEAD.** You checked out a commit instead of a branch. Run `git switch -c rescue-branch` to keep your work, or `git switch main` to leave.

**Authentication errors on push.** Run `gh auth login` again, or `gh auth setup-git` to let Git use your GitHub CLI login.

**Pushed a secret.** Rotate it immediately. Then remove it from history with a tool such as `git filter-repo`, and tell your team.

## 12. Glossary

| Term | Meaning |
|------|---------|
| Repository (repo) | A project folder tracked by Git |
| Commit | A saved snapshot with a message |
| Branch | A movable pointer to a commit; a line of work |
| HEAD | The commit or branch you are on now |
| Remote | A copy of the repo on a server, usually named `origin` |
| Clone | Copy a remote repo to your machine |
| Fork | Your own GitHub copy of someone else's repo |
| Stage | Mark changes to include in the next commit |
| Merge | Combine one branch into another |
| Rebase | Replay commits on top of another branch |
| Conflict | Two changes to the same lines that Git cannot combine |
| Pull request (PR) | A request to merge a branch, with review and checks |
| CI | Automated build and tests that run on pushes and PRs |
| Squash merge | Merge a PR as a single commit |
| Issue | A tracked piece of work: story, bug, or task |
| Stash | A shelf for unfinished changes |
| Tag | A named marker on a commit, often a release |

## 13. Rules of thumb

- Never commit straight to `main`. Use a branch and a PR.
- Pull before you start, push when you finish.
- Commit small and often, with clear messages.
- One PR, one purpose.
- Wait for green CI before merging.
- Check `git status` before every commit and `git diff --staged` before committing.
- Never rewrite history that others already have.
- Never commit secrets.
- Delete branches after merging.
- If something goes wrong, stop and run `git status` and `git reflog` before doing anything drastic.

See [CONTRIBUTING.md](CONTRIBUTING.md) for this project's specific conventions.
