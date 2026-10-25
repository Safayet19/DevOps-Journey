<h1 align="center">Git & GitHub - Practical Guide</h1>

<p align="center">A practical reference for Git commands, GitHub repositories, branches, collaboration, troubleshooting, and everyday workflows.</p>

---

This guide uses **complete commands**, clear explanations, and practical examples. Commands are suitable for **Git Bash on Windows**, Linux, and macOS unless stated otherwise. No CI/CD or GitHub Actions configuration is repeated here.

> **Before starting:** Run repository commands from the correct project folder. Names such as `git-practice-demo` and `docs/setup` are examples. Commands that discard work or rewrite history are clearly marked. Never run cleanup commands in a repository without inspecting `git status` first.

## Table of Contents

1. [Git and GitHub Basics](#1-git-and-github-basics)
2. [Installation and Identity](#2-installation-and-identity)
3. [Creating and Cloning Repositories](#3-creating-and-cloning-repositories)
4. [Changes, Staging and Commits](#4-changes-staging-and-commits)
5. [History and Differences](#5-history-and-differences)
6. [Branches and Merging](#6-branches-and-merging)
7. [Remotes and Synchronization](#7-remotes-and-synchronization)
8. [Undo, Recovery and Stash](#8-undo-recovery-and-stash)
9. [GitHub Collaboration](#9-github-collaboration)
10. [Gitignore, Tags and Security](#10-gitignore-tags-and-security)
11. [Multiple GitHub Accounts](#11-multiple-github-accounts)
12. [Common Errors and Fixes](#12-common-errors-and-fixes)
13. [Practical Workflows](#13-practical-workflows)

---

## 1. Git and GitHub Basics

**Git** tracks changes on your computer. **GitHub** hosts Git repositories online and supports collaboration through pull requests, reviews, issues, and access controls. Git works without an internet connection; pushing to GitHub requires access to the remote repository.

| Term | Meaning | Example |
|---|---|---|
| Repository | A folder tracked by Git | `git-practice-demo/` |
| Working tree | Files you are currently editing | `README.md` before staging |
| Staging area (index) | Changes selected for the next commit | Files selected by `git add` |
| Commit | Saved snapshot of staged changes | `git commit -m "Add guide"` |
| Branch | A movable line of commits | `main`, `docs/setup` |
| `HEAD` | Your currently checked-out commit | Usually the tip of the current branch |
| Remote | A name for another Git repository | `origin` points to GitHub |
| Push | Send local commits to a remote | `git push origin main` |
| Fetch | Download remote Git data without merging | `git fetch origin` |
| Pull | Fetch and integrate remote changes | `git pull --ff-only origin main` |
| Pull request (PR) | Request to review and merge branch changes | `docs/setup` -> `main` |

**Everyday flow:** edit -> `git status` -> `git add` -> `git commit` -> `git push`. For teamwork: create branch -> commit -> push branch -> open PR -> review -> merge.

## 2. Installation and Identity

Install Git from [git-scm.com/downloads](https://git-scm.com/downloads). On Windows, **Git Bash** is a convenient terminal; these commands also work in Linux/macOS terminals.

| Task | Complete command | Explanation |
|---|---|---|
| Check installed Git | `git --version` | Show the installed Git version. |
| Set commit author name | `git config --global user.name "Your Name"` | Use this name as author for new commits by default. |
| Set commit author email | `git config --global user.email "you@example.com"` | Set your commit email; use your verified GitHub or GitHub noreply address for attribution/privacy. |
| Set new repository default branch | `git config --global init.defaultBranch main` | Make new `git init` repositories use `main`. |
| See global settings | `git config --global --list` | Show settings in the global Git config. |
| See where settings originate | `git config --list --show-origin` | List settings and the files that provide them. |
| See current repository identity | `git config user.name && git config user.email` | Print the effective author identity. |
| Use a different name for one repository | `git config --local user.name "Your Name"` | Changes only the current repository's author name. |
| Use a different email for one repository | `git config --local user.email "you@example.com"` | Changes only the current repository's author email. |
| Open Git's command help | `git help commit` | Open documentation for the `commit` command. |

**Important:** `user.name` and `user.email` only label commits; they **do not log you in** to GitHub. For authentication, use Git Credential Manager, GitHub CLI, or SSH. Commit email addresses can be public, so consider GitHub's noreply email setting.

### GitHub authentication

**Option A - GitHub CLI (simple):** Install [GitHub CLI](https://cli.github.com/), then run:

```bash
gh auth login
gh auth status
gh auth setup-git
```

`gh auth login` offers interactive authentication; `gh auth status` checks the active account; `gh auth setup-git` configures Git to use GitHub CLI as its credential helper. If Git Credential Manager is already configured, you can continue using it instead. GitHub no longer accepts account passwords for Git operations over HTTPS.

**Option B - SSH (alternative):** Generate a key **only if you do not already have a suitable one**, add its **public** key to **GitHub > Settings > SSH and GPG keys**, then test:

```bash
ssh-keygen -t ed25519 -C "you@example.com"
cat ~/.ssh/id_ed25519.pub
ssh -T git@github.com
```

Verify GitHub's host fingerprint when prompted. A successful SSH test normally says you authenticated but GitHub does not provide shell access. **Never upload the private file** `~/.ssh/id_ed25519`.

## 3. Creating and Cloning Repositories

| Task | Complete command | Explanation |
|---|---|---|
| Initialize a new repository | `git init -b main` | Initialize the **current directory** with branch `main`; run it only in the intended folder. |
| Show repository root | `git rev-parse --show-toplevel` | Print the top-level folder of the current repository. |
| Check if inside a repository | `git status` | Shows status or reports that the folder is not a Git repository. |
| Clone an existing GitHub repository | `git clone https://github.com/Safayet19/Hello-Linux.git` | Download that repository into a new `Hello-Linux` folder. |
| Choose a destination folder | `git clone https://github.com/Safayet19/Hello-Linux.git linux-copy` | Download into `linux-copy` instead of `Hello-Linux`. |
| Clone using GitHub CLI | `gh repo clone Safayet19/Hello-Linux` | Clone the same repository using `gh` (requires GitHub CLI). |
| List remote repositories | `git remote -v` | Show fetch/push addresses for configured remotes. |

**Folder rule:** Clone into a normal projects directory, **not inside another Git repository**, unless you intentionally need a submodule.

```bash
mkdir -p ~/repos
cd ~/repos
git clone https://github.com/Safayet19/Hello-Linux.git
cd Hello-Linux
git status
```

### Make a new local repository

```bash
mkdir git-practice-demo
cd git-practice-demo
git init -b main
printf '# Git Practice\n' > README.md
git add README.md
git commit -m "Initial commit"
```

This is **local only**. Nothing appears on GitHub until you create a remote repository and push it (Section 7 or Practice Workflow 1).

## 4. Changes, Staging and Commits

| Task | Complete command | Explanation |
|---|---|---|
| View working-tree status | `git status` | Show modified, staged, and untracked files. |
| Compact status | `git status -sb` | `-s` uses short status; `-b` shows branch/tracking information. |
| Stage one file | `git add README.md` | Stage the current content of the named file. |
| Stage all changes | `git add -A` | Stage new, modified, and deleted files throughout the repository. |
| Stage changes in current directory | `git add .` | Stage changes under the current directory; from root, generally covers the repository. |
| Interactively stage selected changes | `git add -p README.md` | `-p` asks which change hunks to include. |
| Show what is staged | `git diff --staged` | Review the content selected for the next commit. |
| Commit staged changes | `git commit -m "Update documentation"` | Create a commit with a clear message. |
| Change last commit message (local only) | `git commit --amend -m "Improve documentation"` | Replaces the last commit; avoid amending shared commits. |
| Show most recent commit | `git log -1 --oneline` | Display the latest abbreviated commit. |
| Track file deletion | `git rm old-notes.txt` | Delete a tracked file and stage the deletion; use only if removal is intended. |
| Track a move/rename | `git mv old-name.txt new-name.txt` | Rename a tracked file and stage that change. |

**Staging is not committing.** `git add` prepares the snapshot. `git commit` records it locally. `git push` publishes it to the remote.

## 5. History and Differences

| Task | Complete command | Explanation |
|---|---|---|
| Compact recent history | `git log --oneline -n 10` | Show the latest 10 commits, one per line. |
| Graph of all branches | `git log --graph --oneline --decorate --all -n 20` | Show a compact branch/merge history. |
| Examine latest commit | `git show HEAD` | Show latest commit metadata and changes. |
| Examine one commit | `git show HEAD~1` | Show the commit immediately before `HEAD` (requires at least two commits). |
| Show unstaged changes | `git diff` | Compare working files with staged versions. |
| Show staged changes | `git diff --staged` | Compare the staging area with the latest commit. |
| Compare branches | `git diff main..docs/setup` | Show changes between named branch tips; both branches must exist. |
| Show changed filenames | `git diff --name-only main..docs/setup` | List just files that differ. |
| Show commits for one file | `git log --oneline -- README.md` | Show commits affecting `README.md`. |
| Read a tracked file from last commit | `git show HEAD:README.md` | Print the committed version without editing the current file. |
| Find who last changed each line | `git blame README.md` | Display line-level commit attribution. |

## 6. Branches and Merging

A branch lets you work on a feature without changing `main` immediately. Create branches **after committing or saving existing work**.

| Task | Complete command | Explanation |
|---|---|---|
| Show local branches | `git branch` | List branches; `*` marks the active one. |
| Show local and remote-tracking branches | `git branch -a` | `-a` includes branches known from remotes. |
| Show branch upstreams | `git branch -vv` | List local branches and their tracking information. |
| Create and switch to feature branch | `git switch -c docs/setup` | `-c` creates `docs/setup` and checks it out. |
| Switch to main | `git switch main` | Check out the existing `main` branch. |
| Merge feature into main | `git merge docs/setup` | From `main`, incorporate the feature branch's commits. |
| Force a merge commit when needed | `git merge --no-ff docs/setup` | Creates a merge commit even when fast-forward is possible. |
| Delete a merged local branch | `git branch -d docs/setup` | Safely delete a merged local branch. |
| Show files with merge conflicts | `git status` | Lists unmerged paths. |
| Cancel a conflicting merge | `git merge --abort` | Try to return to the pre-merge state; start merges with a clean tree. |
| Finish a resolved merge | `git merge --continue` | Finish after editing conflict files and staging resolutions. |

### What a merge conflict looks like

```text
<<<<<<< HEAD
Title from main
=======
Title from feature branch
>>>>>>> docs/setup
```

Resolve it by editing the file into the **final content you actually want** and deleting all conflict markers (`<<<<<<<`, `=======`, `>>>>>>>`). Then run `git add README.md` and `git merge --continue`. Do not simply commit the markers.

**Rebase (optional):** `git rebase main` moves your current feature branch's commits onto the latest local `main`. It rewrites commit IDs. Use it on **your own, unshared feature branch**, not as a routine fix for shared `main`.

## 7. Remotes and Synchronization

`origin` is the conventional name for your main GitHub remote. A **remote URL** tells Git where to fetch or push; it is not a branch name.

| Task | Complete command | Explanation |
|---|---|---|
| Show remote URLs | `git remote -v` | Confirm the repository is connected to the right GitHub address. |
| Add a remote to a local project | `git remote add origin https://github.com/Safayet19/git-practice-demo.git` | Connect to a GitHub repository that **already exists**. |
| Correct a remote address | `git remote set-url origin https://github.com/Safayet19/git-practice-demo.git` | Replace the URL without creating a second remote. |
| Fetch new remote commits | `git fetch origin` | Download changes without merging into your working branch. |
| Fetch and discard stale tracking names | `git fetch --prune origin` | `--prune` removes remote-tracking references deleted on the server, not your local branches. |
| Check main's new remote commits | `git log --oneline main..origin/main` | Show fetched remote commits not on local `main`. |
| Pull safely on main | `git pull --ff-only origin main` | Fetch and fast-forward only; refuses to create a merge if histories diverged. |
| First push of main | `git push -u origin main` | Push and set `origin/main` as upstream for local `main`. |
| Subsequent push | `git push` | Push current branch to its configured upstream. |
| Push new feature branch | `git push -u origin docs/setup` | Publish feature branch and set upstream tracking. |
| Delete merged remote branch | `git push origin --delete docs/setup` | Remove that remote branch after merge; does not delete your local branch. |

### Normal push workflow

```bash
git status
git add -A
git diff --staged
git commit -m "Update Git documentation"
git push origin main
```

**Push rejected because remote is ahead?** First inspect the remote, then integrate new commits. If you're on a clean `main` branch and a fast-forward is possible:

```bash
git fetch origin
git status -sb
git pull --ff-only origin main
git push origin main
```

If `--ff-only` refuses, your history diverged. Do **not** automatically force-push: inspect `git log --graph --oneline --all` and decide whether to merge or rebase your own branch.

## 8. Undo, Recovery and Stash

**Understand what you are undoing:** a file edit, a staged change, a local commit, or a commit already pushed. These require different tools.

| Task | Complete command | Explanation |
|---|---|---|
| Unstage a file | `git restore --staged README.md` | Remove it from staging but keep your edited file. |
| Discard unstaged edits to a tracked file | `git restore README.md` | **Destructive:** restore from index; current unstaged edits are lost. |
| Restore one file from last commit | `git restore --source=HEAD -- README.md` | Replace working file with its last committed version. |
| Undo last commit, keep staged changes | `git reset --soft HEAD~1` | Move branch back one commit, preserving staged changes. Local/unshared history only. |
| Undo last commit, keep edited files unstaged | `git reset --mixed HEAD~1` | Move branch back and unstage changes; files remain. Local/unshared history only. |
| Revert a published commit safely | `git revert HEAD` | Creates a new commit reversing the latest commit without rewriting prior history. |
| Save unfinished tracked changes | `git stash push -m "WIP documentation"` | Temporarily save tracked modifications and return to a clean tree. |
| List saved stashes | `git stash list` | Show stash entries. |
| Restore latest stash | `git stash pop` | Reapply latest stash and remove it on successful application; conflicts are possible. |
| Read stash changes | `git stash show -p stash@{0}` | `-p` shows the patch for the latest stash. |
| Find earlier branch positions | `git reflog -n 10` | Show recent movements of `HEAD`, useful for recovering local commits. |
| Preview untracked-file cleanup | `git clean -nd` | List what would be deleted; no files removed. |

**Never use `git reset --hard` as a generic fix.** It can discard tracked working-tree changes. `git push --force` can overwrite other contributors' remote history. If deliberately rewriting **your own feature branch**, understand the impact first; `--force-with-lease` is safer than unconditional force, but still rewrites history.

**Rule:** Use `restore` for files, `reset` for local unpublished history, `revert` for commits already shared. These commands do different jobs.

## 9. GitHub Collaboration

You can use the GitHub website or the optional `gh` CLI. Pull requests and branch protection are **GitHub features**, not basic Git commands.

| Task | Complete command | Explanation |
|---|---|---|
| Check CLI login | `gh auth status` | Show authenticated GitHub accounts. |
| Open repository in browser | `gh repo view --web` | Open the current repository on GitHub. |
| Create a private GitHub repo from current local repo | `gh repo create git-practice-demo --private --source=. --remote=origin --push` | Create a new remote repo, add `origin`, and push current commits; repo name must be available. |
| List pull requests | `gh pr list` | Show PRs for the current GitHub repository. |
| Create a PR from a pushed branch | `gh pr create --base main --head docs/setup --title "Document setup" --body "Adds setup instructions."` | Request review of `docs/setup` into `main`. |
| Open the PR in a browser | `gh pr view --web` | View the PR related to the current branch. |
| Check PR status | `gh pr checks` | Show available CI checks for the current PR. |
| Merge an approved PR | `gh pr merge --merge --delete-branch` | Merge the current branch's PR and delete the remote branch when allowed; review it first. |
| List issues | `gh issue list` | Show repository issues. |
| Open issue creation in browser | `gh issue create --web` | Open GitHub's issue creation UI. |

### GitHub website workflow (without GitHub CLI)

1. In the repository, open **Pull requests > New pull request**.
2. Choose **base: `main`** and **compare: `docs/setup`**.
3. Read the **Files changed** tab, write a title and description, and submit the PR.
4. Wait for requested reviews/checks. Merge only when approved, then delete the feature branch if no longer needed.

**Fork vs clone:** A **fork** is a GitHub repository copy under a different owner, useful when you cannot push directly. A **clone** copies a repository onto your own computer. They are not the same.

**Branch protection:** In **Repository > Settings > Rules**, administrators can require reviews or passing checks before merging. Direct push to `main` may be blocked by repository policy; use a feature branch and PR instead.

## 10. Gitignore, Tags and Security

A `.gitignore` file prevents **untracked matching files** from being added inadvertently. It does **not** erase files already committed.

**Example `.gitignore` (adapt it to the project):**

```gitignore
.env
.env.*
*.log
node_modules/
.idea/
.DS_Store
```

| Task | Complete command | Explanation |
|---|---|---|
| Check why a file is ignored | `git check-ignore -v .env` | Show the matching ignore rule (if any). |
| Stop tracking an already tracked `.env` file | `git rm --cached -- .env` | Remove it from Git's index while keeping the local file; commit the change. Does not remove past history. |
| List tags | `git tag -l` | Display version tags. |
| Create annotated version tag | `git tag -a v1.0.0 -m "First release"` | Attach a version label to the current commit. |
| Inspect a tag | `git show v1.0.0` | Show what the tag points to. |
| Publish a tag | `git push origin v1.0.0` | Send the tag to GitHub. |

**Security rules:** Never commit passwords, API tokens, SSH private keys, `.env` secrets, or service-account credentials. If a secret is committed, **revoke/rotate it first**; a later `.gitignore` or deletion commit does not remove it from old commits. GitHub secrets and GitHub Actions are documented separately.

## 11. Multiple GitHub Accounts

**Author identity**, **GitHub CLI login**, and **Git push authentication** are different things. You can use different commit identities per repository without changing your global defaults:

```bash
# Run inside the specific repository
git config --local user.name "Your Name"
git config --local user.email "account-specific@example.com"
git config user.email
git remote -v
```

For **GitHub CLI**, sign in to each account using `gh auth login`, then switch as needed:

```bash
gh auth status
gh auth switch --hostname github.com --user OTHER_USERNAME
gh auth status
```

Replace `OTHER_USERNAME` with a GitHub account already authenticated in `gh`. Switching GitHub CLI accounts may not automatically change which account a separate HTTPS credential manager uses for Git push.

**For reliable two-account SSH access (optional):** Create separate SSH keys, add each **public** key to the corresponding account, and assign different host aliases in `~/.ssh/config`:

```sshconfig
Host github-personal
    HostName github.com
    User git
    IdentityFile ~/.ssh/id_ed25519_personal
    IdentitiesOnly yes

Host github-other
    HostName github.com
    User git
    IdentityFile ~/.ssh/id_ed25519_other
    IdentitiesOnly yes
```

For a personal repository, set a matching SSH URL:

```bash
git remote set-url origin git@github-personal:Safayet19/git-practice-demo.git
git remote -v
ssh -T git@github-personal
```

Create the named keys and verify their GitHub authorization first. Never paste private keys into the README. This is optional; one account with `gh auth login` is simpler.

## 12. Common Errors and Fixes

| Error or symptom | Likely reason | Specific check or fix |
|---|---|---|
| `fatal: not a git repository` | Current folder isn't within a Git working tree | Run `pwd`; enter the correct repository; confirm with `git rev-parse --show-toplevel`. |
| `Author identity unknown` | Git commit name/email not configured | Set `git config --global user.name "Your Name"` and `git config --global user.email "you@example.com"`. |
| `nothing to commit, working tree clean` | No staged or unstaged changes | Inspect `git status`; edit/create a file before committing. |
| `src refspec main does not match any` | No commit on `main` or branch has a different name | Run `git branch --show-current` and `git log -1`; create first commit if needed. |
| `remote origin already exists` | `origin` was configured already | Run `git remote -v`; correct URL with `git remote set-url origin URL` instead of adding it again. |
| Push rejected (`non-fast-forward`) | GitHub branch has commits missing locally | `git fetch origin`; inspect graph; on clean `main`, try `git pull --ff-only origin main` if histories allow. |
| `Permission denied (publickey)` | Wrong or missing SSH key | Check account key and remote URL; test `ssh -T git@github.com`. |
| GitHub repository `not found` | Wrong URL, wrong authenticated account, or no permission | Check `git remote -v`, `gh auth status`, and repository access. |
| HTTPS authentication fails | Invalid/expired credential or wrong account | Check your approved credential manager or authenticate with `gh auth login`. |
| `adding embedded git repository` | A cloned repo with its own `.git` was put inside another repo | Inspect nested `.git`; see the exact safe remediation below. |
| GitHub README appears inside a folder | Files were committed under an unwanted subdirectory | If you want them at root, move the actual files, remove empty folder, stage, and push. |
| Merge conflict | Both branches changed the same lines | Edit conflict markers, stage resolved files, then `git merge --continue` or `git merge --abort`. |
| `pathspec ... did not match any file` | Wrong path or file not tracked | Confirm name and location using `ls -la` and `git status`. |
| Secret remains visible after adding `.gitignore` | It was already committed | Revoke the secret immediately; `.gitignore` alone cannot rewrite history. |

### A. Embedded Git repository warning (the problem we encountered)

Example: You clone another project into `Hello-Linux/third-party/`. That nested folder has **its own `.git`**, so the parent may record it as a gitlink instead of ordinary files.

If you **intend to copy its ordinary files into the parent repository**, inspect it first:

```bash
git status
ls -la third-party
```

**Only if `third-party` was staged as an embedded Git repository**, unstage its gitlink:

```bash
git rm --cached -r -f third-party
```

**Only if you no longer want it to be an independent Git repository**, remove its nested metadata, then add normal files:

```bash
rm -rf third-party/.git
git add -A
git status
```

This removes nested Git history/metadata from the copied directory. If you need to preserve it as a separate repository, use an intentional **Git submodule** instead. Respect the original project's license and attribution.

### B. Move a single README out of an unnecessary folder

**Only for this exact situation:** `Linux-Commands/README.md` exists and there is **no** `README.md` at the repository root. Do not overwrite an existing root README.

```bash
# Run from the parent repository root
test -f Linux-Commands/README.md && test ! -e README.md && mv Linux-Commands/README.md README.md
rmdir Linux-Commands
git add -A
git diff --staged --stat
git commit -m "Move README to repository root"
git push origin main
```

This moves the file to root and deletes the now-empty folder. `test ! -e README.md` prevents accidental overwrite. If the folder has more files or root `README.md` exists, **inspect first and choose a different move**; do not run these commands blindly.

## 13. Practical Workflows

### Project 1 - Create, commit and publish a practice repository

**Requirements:** Git installed; GitHub CLI installed and authenticated; new GitHub repository name `git-practice-demo` not already used in your account. This lab creates a **private** GitHub repository.

```bash
mkdir -p ~/repos/git-practice-demo
cd ~/repos/git-practice-demo
git init -b main
printf '# Git Practice Demo\n\nLearning Git step by step.\n' > README.md
git add README.md
git commit -m "Initial commit"
git status -sb
gh auth login
gh repo create git-practice-demo --private --source=. --remote=origin --push
git remote -v
```

**Expected:** GitHub has one private repo with `README.md` at its root and your initial commit on `main`. If `git-practice-demo` already exists, choose a unique repository name in the `gh repo create` command.

### Project 2 - Create a feature branch and open a pull request

**Run inside the practice repo from Project 1.**

```bash
git switch -c docs/setup
printf '\n## Setup\n\nRun `git --version` to check Git.\n' >> README.md
git add README.md
git commit -m "Document Git setup"
git push -u origin docs/setup
gh pr create --base main --head docs/setup --title "Add setup instructions" --body "Adds a short Git setup section."
gh pr view --web
```

**Expected:** GitHub shows a PR proposing `docs/setup` into `main`. Review it in the browser and merge only when ready. After merging on GitHub, update your local main:

```bash
git switch main
git pull --ff-only origin main
git branch -d docs/setup
```

### Project 3 - Practice a merge conflict safely (local only)

Use a **separate new practice directory**, not an important repository.

```bash
mkdir -p ~/repos/git-conflict-lab
cd ~/repos/git-conflict-lab
git init -b main
printf 'title=original\n' > config.txt
git add config.txt
git commit -m "Add initial config"

git switch -c feature/title
printf 'title=feature\n' > config.txt
git add config.txt
git commit -m "Change title in feature"

git switch main
printf 'title=main\n' > config.txt
git add config.txt
git commit -m "Change title in main"

git merge feature/title
```

**Expected:** A conflict in `config.txt`, because both branches changed the same line differently. Inspect it using `cat config.txt`. Then resolve and finish:

```bash
printf 'title=resolved\n' > config.txt
git add config.txt
git merge --continue
git log --graph --oneline --all -n 8
```

An editor may open for the merge message. If you want to cancel instead of resolving, run `git merge --abort` **before** completing the merge.

### Project 4 - Check an existing repository before changing it

Use this read-only sequence inside any project:

```bash
pwd
git rev-parse --show-toplevel
git status -sb
git remote -v
git branch -vv
git log --oneline -n 5
```

**Expected:** You can identify the **correct folder, active branch, remote address, upstream, and latest commits** before staging, merging, or pushing anything.

### Practice checklist

- [ ] Configure Git author identity and check your GitHub login separately.
- [ ] Create a local repository, stage files, commit, and inspect history.
- [ ] Clone a repository **outside** other Git repositories.
- [ ] Create a branch, push it, open a PR, and merge with review.
- [ ] Compare `fetch`, `pull`, `push`, and `pull --ff-only`.
- [ ] Resolve a merge conflict in a disposable practice repository.
- [ ] Explain `restore`, `reset`, `revert`, and `stash` without confusing them.
- [ ] Keep secrets out of Git and verify `.gitignore` behavior.
- [ ] Diagnose a wrong remote, embedded repository warning, and rejected push.

---

## Official References

- [Git reference manual](https://git-scm.com/docs)
- [GitHub documentation](https://docs.github.com/)
- [GitHub CLI manual](https://cli.github.com/manual/)
- [GitHub SSH authentication](https://docs.github.com/en/authentication/connecting-to-github-with-ssh)
- [GitHub pull requests](https://docs.github.com/en/pull-requests)

*Use the commands on repositories you own or are authorized to modify. Review paths and staged changes before pushing. This guide is for practical Git and GitHub fundamentals; GitHub Actions is documented separately.*
