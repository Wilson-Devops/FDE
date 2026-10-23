# Git and GitHub Beginner Hands-On Lab

## Lab objective

Complete a basic Git and GitHub workflow from start to finish:

```text
Configure Git
    ↓
Create a repository
    ↓
Create a feature branch
    ↓
Change a file
    ↓
Check the change
    ↓
Stage and commit
    ↓
Protect secrets
    ↓
Push to GitHub
    ↓
Create a pull request
```

---

## Git in simple words

Think of Git as **Save Game for your project**.

A **commit** is a named save point. If something breaks later, Git helps you understand what changed and return to an earlier version.

- **Git** is version-control software installed on your computer.
- **GitHub** is an online service used to store Git repositories and collaborate with others.
- **Repository** means the project folder Git tracks.
- **Branch** is a separate line of work where you can make changes without immediately changing the main version.
- **Commit** is a saved snapshot of your work.
- **Push** sends local commits to GitHub.
- **Pull request** asks for a branch to be reviewed and merged into another branch.

---

# Actionable Lab Execution Steps

## Task 1: Check that Git is installed

### Step 1.1: Open a terminal

Use one of the following:

- Windows Terminal
- PowerShell
- Command Prompt
- Git Bash
- Visual Studio Code terminal

### Step 1.2: Check the Git version

```bash
git --version
```

Expected result:

```text
git version 2.x.x
```

If the terminal says that `git` is not recognized, install Git from the official Git website and reopen the terminal.

### Task completion check

- [ ] `git --version` displays a Git version.

---

## Task 2: Configure your Git username and email

Git records an author name and email address with every commit.

### Step 2.1: Configure your name

Replace the example with your own name:

```bash
git config --global user.name "Wilson Manda"
```

### Step 2.2: Configure your email

Use the email associated with your GitHub account:

```bash
git config --global user.email "wilson.devops@gmail.com"
```

### Step 2.3: Verify the configuration

```bash
git config --global user.name
```

```bash
git config --global user.email
```

### Task completion check

- [ ] Git displays the correct username.
- [ ] Git displays the correct email address.

---

## Task 3: Create a local Git repository

### Step 3.1: Create a project folder

```bash
mkdir git-lab
```

### Step 3.2: Enter the project folder

```bash
cd git-lab
```

### Step 3.3: Initialize Git

```bash
git init
```

`git init` changes an ordinary folder into a Git repository.

### Step 3.4: Confirm the repository status

```bash
git status
```

### Task completion check

- [ ] The `git-lab` folder exists.
- [ ] `git init` completes successfully.
- [ ] `git status` works inside the folder.

---

## Task 4: Create the first project file

### Step 4.1: Open the project in Visual Studio Code

```bash
code .
```

If the `code` command is unavailable, open Visual Studio Code manually and select the `git-lab` folder.

### Step 4.2: Create `README.md`

Add the following content:

```markdown
# My Git Lab

Learning Git step by step.
```

Save the file.

### Step 4.3: Check the repository status

```bash
git status
```

Git should show `README.md` as an **untracked file**.

An untracked file is a new file that Git is not tracking yet.

### Task completion check

- [ ] `README.md` exists.
- [ ] `git status` shows `README.md` as untracked.

---

## Task 5: Stage and commit the first file

### Step 5.1: Stage `README.md`

```bash
git add README.md
```

Staging means placing a change into the group of changes for the next commit.

```text
Working file
     ↓
git add
     ↓
Staging area
```

### Step 5.2: Check the status

```bash
git status
```

`README.md` should now appear under changes to be committed.

### Step 5.3: Create the first commit

```bash
git commit -m "Add initial README"
```

The text after `-m` is the commit message. It should briefly describe the change.

### Step 5.4: View the commit history

```bash
git log --oneline
```

Example output:

```text
a12bc34 Add initial README
```

### Task completion check

- [ ] `README.md` is staged.
- [ ] The first commit is created.
- [ ] `git log --oneline` shows the commit.

---

## Task 6: Create a feature branch

A branch provides a separate place to make a change without immediately changing the main branch.

```text
main
  └── feature/200-ui-homepage
```

### Step 6.1: Create and switch to the feature branch

```bash
git switch -c feature/200-ui-homepage
```

If `git switch` is unavailable, use:

```bash
git checkout -b feature/200-ui-homepage
```

### Step 6.2: Confirm the current branch

```bash
git branch
```

The current branch should have an asterisk:

```text
* feature/200-ui-homepage
```

### Task completion check

- [ ] The branch `feature/200-ui-homepage` exists.
- [ ] The feature branch is the current branch.

---

## Task 7: Make and inspect a small change

### Step 7.1: Update `README.md`

Replace its contents with:

```markdown
# My Git Lab

Learning Git step by step.

## Homepage Feature

This feature was created on a separate Git branch.
```

Save the file.

### Step 7.2: Check the status

```bash
git status
```

Git should show that `README.md` was modified.

### Step 7.3: Inspect the exact change

```bash
git diff
```

`git diff` shows the lines that were added, changed, or removed before the change is staged.

### Task completion check

- [ ] `README.md` contains the homepage section.
- [ ] `git status` reports the modified file.
- [ ] `git diff` displays the change.

---

## Task 8: Stage and commit the feature

### Step 8.1: Stage the changed file

```bash
git add README.md
```

### Step 8.2: Confirm the staged change

```bash
git status
```

### Step 8.3: Commit the feature

```bash
git commit -m "Add homepage feature description"
```

### Step 8.4: Confirm that the working tree is clean

```bash
git status
```

Expected message:

```text
nothing to commit, working tree clean
```

### Task completion check

- [ ] The feature change is staged.
- [ ] The feature commit is created.
- [ ] The working tree is clean.

---

## Task 9: Protect secret files with `.gitignore`

Applications may use `.env` files for configuration values. A `.env` file can contain sensitive values, so it should normally not be committed.

Never put real passwords, tokens, API keys, or private keys in this practice lab.

### Step 9.1: Create `.gitignore`

Create a file named `.gitignore` and add:

```gitignore
.env
.env.*
*.key
*.pem
```

This tells Git not to track matching files.

```text
README.md     → track
.gitignore    → track
.env          → ignore
private.key   → ignore
```

### Step 9.2: Create a safe test `.env` file

Create `.env` with fake practice data only:

```text
LAB_VALUE=example
```

### Step 9.3: Test the ignore rule

```bash
git status
```

The `.env` file should not be listed. The `.gitignore` file should be listed.

### Step 9.4: Stage and commit `.gitignore`

```bash
git add .gitignore
```

```bash
git commit -m "Add gitignore for secret files"
```

### Optional verification

```bash
git check-ignore -v .env
```

This command should show which `.gitignore` rule is ignoring `.env`.

### Task completion check

- [ ] `.gitignore` exists.
- [ ] `.env` contains fake data only.
- [ ] `.env` does not appear in `git status`.
- [ ] `.gitignore` is committed.

---

## Task 10: Create a GitHub repository

### Step 10.1: Sign in to GitHub

Open GitHub in a browser and sign in.

### Step 10.2: Create the remote repository

Create a repository named:

```text
git-lab
```

Because the local project already contains a README and commits, do not initialize the GitHub repository with another README, `.gitignore`, or license.

### Step 10.3: Copy the repository URL

It should look similar to:

```text
https://github.com/YOUR-USERNAME/git-lab.git
```

Use the URL displayed in your own GitHub repository.

### Task completion check

- [ ] The GitHub repository `git-lab` exists.
- [ ] The repository URL is copied.

---

## Task 11: Connect the local repository to GitHub

### Step 11.1: Add the GitHub repository as `origin`

Replace the example URL with your actual repository URL:

```bash
git remote add origin https://github.com/YOUR-USERNAME/git-lab.git
```

`origin` is the standard nickname for the remote repository.

### Step 11.2: Verify the remote

```bash
git remote -v
```

The result should show the GitHub URL for fetch and push operations.

### Task completion check

- [ ] The remote named `origin` exists.
- [ ] `git remote -v` shows the correct GitHub URL.

---

## Task 12: Push the main branch

### Step 12.1: Check local branches

```bash
git branch
```

### Step 12.2: Switch to the initial branch

If the initial branch is `main`:

```bash
git switch main
```

If the initial branch is `master`, switch to it and rename it:

```bash
git switch master
git branch -M main
```

### Step 12.3: Push `main` to GitHub

```bash
git push -u origin main
```

The `-u` option connects the local `main` branch to the remote `main` branch. Later, you can normally use `git push` without writing the remote and branch names.

GitHub may request authentication depending on your Git setup.

### Step 12.4: Verify on GitHub

Refresh the repository page. The initial `README.md` should appear on the `main` branch.

### Task completion check

- [ ] The local initial branch is named `main`.
- [ ] `main` is pushed to GitHub.
- [ ] GitHub displays the committed project file.

---

## Task 13: Push the feature branch

### Step 13.1: Switch to the feature branch

```bash
git switch feature/200-ui-homepage
```

### Step 13.2: Push the branch to GitHub

```bash
git push -u origin feature/200-ui-homepage
```

GitHub should now contain both branches:

```text
main
feature/200-ui-homepage
```

### Task completion check

- [ ] The feature branch is pushed to GitHub.
- [ ] GitHub shows `feature/200-ui-homepage`.

---

## Task 14: Create a pull request

A pull request asks for the feature branch to be reviewed and merged into `main`.

```text
feature/200-ui-homepage
           ↓
      Pull Request
           ↓
          main
```

### Step 14.1: Start the pull request

Open the GitHub repository and compare the recently pushed feature branch.

Use:

```text
Base branch: main
Compare branch: feature/200-ui-homepage
```

### Step 14.2: Add a title

```text
Add homepage feature
```

### Step 14.3: Add a description

```markdown
## What changed

- Updated the README
- Added homepage feature information
- Added `.gitignore` rules for secret files
```

### Step 14.4: Create the pull request

Review the displayed file changes and create the pull request.

### Task completion check

- [ ] The base branch is `main`.
- [ ] The compare branch is `feature/200-ui-homepage`.
- [ ] The pull request has a clear title and description.
- [ ] The pull request is created.

---

# Final validation

Run these commands locally:

```bash
git status
git branch
git log --oneline --all
git remote -v
```

Confirm the following:

- [ ] `git status` does not show unexpected changes.
- [ ] `main` and `feature/200-ui-homepage` exist.
- [ ] The commit history includes the README, feature, and `.gitignore` commits.
- [ ] The `origin` remote points to the correct GitHub repository.
- [ ] The GitHub repository contains both branches.
- [ ] A pull request exists from the feature branch into `main`.

---

# Lab completion checklist

- [ ] Git is installed.
- [ ] Git username is configured.
- [ ] Git email is configured.
- [ ] The local `git-lab` repository is created.
- [ ] `README.md` is created.
- [ ] The first commit is created.
- [ ] `feature/200-ui-homepage` is created.
- [ ] A feature change is made.
- [ ] `git status` is checked.
- [ ] `git diff` is checked.
- [ ] The feature is staged and committed.
- [ ] `.gitignore` is created.
- [ ] `.env` is ignored.
- [ ] The GitHub repository is created.
- [ ] The local repository is connected to GitHub.
- [ ] `main` is pushed.
- [ ] The feature branch is pushed.
- [ ] A pull request is created.

## Lab result

When every item is checked, the lab is complete.

Remember the basic workflow:

```text
Edit → Status → Diff → Add → Commit → Push → Pull Request
```

---

# Quick command reference

## Check what is happening

```bash
git status
```

## See unstaged changes

```bash
git diff
```

## Stage a specific file

```bash
git add README.md
```

## Stage all current changes

```bash
git add .
```

Review `git status` before committing when using `git add .`.

## Create a commit

```bash
git commit -m "Describe the change"
```

## List branches

```bash
git branch
```

## Create and switch to a branch

```bash
git switch -c branch-name
```

## Switch branches

```bash
git switch branch-name
```

## View commit history

```bash
git log --oneline
```

## Download and integrate remote changes

```bash
git pull
```

## Upload local commits

```bash
git push
```
