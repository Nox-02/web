# update-readme.ps1
# Rewrites README.md as a "how to use GitHub" guide, opens a PR, waits for CI,
# squash-merges it, and syncs your local main. Run from anywhere.

$ErrorActionPreference = 'Stop'

function Check($step) {
    if ($LASTEXITCODE -ne 0) { throw "Failed at: $step" }
}

$repoPath = Join-Path $env:USERPROFILE 'Documents\web'
$branch   = 'docs/github-guide'

Set-Location $repoPath

# --- Make sure tools exist and gh is logged in ---
foreach ($tool in 'git', 'gh') {
    if (-not (Get-Command $tool -ErrorAction SilentlyContinue)) {
        throw "$tool is not installed or not on PATH."
    }
}
gh auth status | Out-Null
Check 'gh auth status (run: gh auth login)'

# --- Start from a clean, up-to-date main ---
$dirty = git status --porcelain
if ($dirty) { throw "You have uncommitted changes. Commit or stash them first." }

git checkout main;  Check 'checkout main'
git pull;           Check 'pull main'

# If the branch already exists locally, remove it so we start fresh
git show-ref --verify --quiet "refs/heads/$branch"
if ($LASTEXITCODE -eq 0) { git branch -D $branch; Check 'delete old local branch' }

git checkout -b $branch; Check 'create branch'

# --- Write the README ---
$readme = @'
# How to Use This Repo on GitHub

A quick guide to the workflow we use here: issues, branches, pull requests, CI, and merging.

## The workflow at a glance

1. Create an issue (user story, bug, or task)
2. Create a branch from `main`
3. Commit your work
4. Push and open a pull request (PR)
5. Wait for CI to pass
6. Squash-merge into `main`
7. Delete the branch and sync locally

## 1. Create an issue

Every piece of work starts as an issue.

- On GitHub: **Issues -> New issue**, then pick a template:
  - **User story**: a feature written from the user's point of view
  - **Bug report**: something is broken
  - **Task**: technical or chore work
- From the terminal: `gh issue create`

Note the issue number (for example `#12`). You will link it in your PR.

## 2. Create a branch

Never work directly on `main`. Start from an up-to-date `main`:

```
git checkout main
git pull
git checkout -b feat/short-description
```

Branch name prefixes:

| Prefix | Use for |
|--------|---------|
| `feat/` | new features |
| `fix/` | bug fixes |
| `chore/` | tooling, config, maintenance |
| `docs/` | documentation only |

## 3. Commit your work

```
git add .
git commit -m "feat: add login form"
```

Commit message format is `type: short summary`, using the same types as the branch prefixes. Small, focused commits are easier to review.

## 4. Push and open a pull request

```
git push -u origin HEAD
gh pr create
```

Or click the **Compare & pull request** button on GitHub after pushing.

The PR template will prefill. Fill it in and link your issue with `Closes #12` so the issue closes automatically on merge.

## 5. Check CI

Every PR runs the CI workflow automatically.

```
gh pr checks --watch
```

- Green check: good to merge.
- Red X: run `gh run view --log-failed` to see what failed, fix it, commit, and push again. CI re-runs on its own.

## 6. Merge

```
gh pr merge --squash --delete-branch
```

Squash merge keeps `main` history to one clean commit per PR. Or use the **Squash and merge** button on GitHub.

## 7. Sync locally

```
git checkout main
git pull
```

## Cheat sheet

| I want to... | Command |
|--------------|---------|
| See which branch I'm on | `git branch` |
| See what I changed | `git status` |
| See my open PRs | `gh pr list` |
| Open the current PR in the browser | `gh pr view --web` |
| List open issues | `gh issue list` |
| Undo local changes to a file | `git restore <file>` |
| Switch back to main | `git checkout main` |

## Rules of thumb

- Never commit straight to `main`. Use a branch and a PR.
- One PR, one purpose. Keep changes small.
- Wait for green CI before merging.
- Pull `main` before starting anything new.
- Delete branches after merging.

See [CONTRIBUTING.md](CONTRIBUTING.md) for more detail on conventions.
'@

[System.IO.File]::WriteAllText(
    (Join-Path $PWD 'README.md'),
    $readme,
    (New-Object System.Text.UTF8Encoding($false))
)
Write-Host 'README.md written' -ForegroundColor Green

# --- Commit and push ---
git add README.md;  Check 'git add'
git commit -m 'docs: rewrite README as a GitHub usage guide'; Check 'git commit'
git push -u origin HEAD; Check 'git push'

# --- Open the PR ---
$body = "Replaces the placeholder README with a step-by-step guide to the GitHub workflow: issues, branches, PRs, CI, and merging."
gh pr create --base main --title 'docs: rewrite README as a GitHub usage guide' --body $body
Check 'gh pr create'

# --- Wait for CI (give GitHub a few seconds to register the run) ---
Write-Host 'Waiting for CI to start...' -ForegroundColor Cyan
Start-Sleep -Seconds 8
gh pr checks --watch
if ($LASTEXITCODE -ne 0) {
    Write-Host 'CI did not pass. Not merging. Inspect with: gh run view --log-failed' -ForegroundColor Red
    exit 1
}

# --- Merge and sync ---
gh pr merge --squash --delete-branch
Check 'gh pr merge'

git checkout main; Check 'checkout main'
git pull;          Check 'pull main'

Write-Host ''
Write-Host 'Done. README is live on main.' -ForegroundColor Green