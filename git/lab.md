# Task 1: Practice an Enterprise Git and GitHub Workflow

## What you will do
In this lab you will practice the normal Git workflow used when several people work on the same project:

1. Configure your Git identity.
2. Create or clone a GitHub repository.
3. Create a separate feature branch.
4. Make a small change.
5. Check the change with `git status`.
6. Stage and commit the change.
7. Protect secrets using `.gitignore`.
8. Push your feature branch to GitHub.
9. Open a Pull Request for review.

> Beginner idea: Think of the `main` branch as the approved master copy. A feature branch is your private working copy. A Pull Request asks the team to review your working copy before it becomes part of `main`.

---

## Before you start

You need:

- Git installed on your computer.
- A GitHub account and access to the repository you will use.
- A terminal such as Git Bash, PowerShell, Windows Terminal, or the VS Code terminal.
- A text editor such as VS Code.

Run this to confirm Git is installed:

```bash
git --version
```

If a Git version is displayed, continue.

---

# Step 1: Configure your Git username and email

Git records the author of every commit. Configure the name and email that should appear on your commits.

```bash
git config --global user.name "Your Name"
git config --global user.email "your-email@example.com"
```

Check the values:

```bash
git config --global user.name
git config --global user.email
```

### Expected result
The terminal prints the name and email you configured.

> Use the email associated with your GitHub account when that is your organization's required setup.

---

# Step 2: Create or clone the repository

Use **one** of the following options.

## Option A: Clone an existing GitHub repository

On GitHub, open the repository, choose **Code**, and copy the repository URL. Then run:

```bash
git clone <repository-url>
cd <repository-folder>
```

Example shape:

```bash
git clone https://github.com/your-org/sample-app.git
cd sample-app
```

The example URL above is only a placeholder. Use the real URL provided for your lab.

## Option B: Start a new local repository

```bash
mkdir git-workflow-lab
cd git-workflow-lab
git init
```

Create a starter README:

```bash
printf "# Git Workflow Lab\n" > README.md
git add README.md
git commit -m "Initial commit"
```

> If the lab requires GitHub review, the repository must also have a GitHub remote before you can push and create a Pull Request.

Check the repository:

```bash
git status
```

---

# Step 3: Create a feature branch

Do not make the lab change directly on `main`. Create a branch whose name explains the work.

Example from the task:

```bash
git switch -c feature/200-ui-homepage
```

If your Git version does not support `git switch`, use:

```bash
git checkout -b feature/200-ui-homepage
```

Confirm the active branch:

```bash
git branch --show-current
```

### Expected result

```text
feature/200-ui-homepage
```

---

# Step 4: Make a small change

Open a sample file such as `README.md` and add a harmless line, for example:

```markdown
## Homepage update
This change was created while practising the feature-branch workflow.
```

Save the file.

Now ask Git what changed:

```bash
git status
```

### What `git status` means
Git should show the edited file as modified. At this point the file has changed, but it has not yet been selected for the next commit.

Optional: see the actual text change:

```bash
git diff
```

---

# Step 5: Protect secret files with `.gitignore`

Files containing passwords, API keys, tokens, local environment variables, or credentials should not be committed.

Open or create a file named `.gitignore` in the repository root and add entries such as:

```gitignore
.env
.env.*
*.pem
*.key
credentials.json
```

> Important: `.gitignore` prevents matching **untracked** files from being added in normal Git operations. If a secret was already committed, simply adding the filename to `.gitignore` does not remove that secret from Git history. Treat an exposed credential as compromised and follow your organization's credential-rotation process.

Check the repository again:

```bash
git status
```

You should see `.gitignore` as a new or modified file, while ignored secret files should not be offered as normal untracked files.

---

# Step 6: Stage the files

Staging means selecting exactly what should go into the next commit.

First inspect the status:

```bash
git status
```

For this lab, stage the sample change and `.gitignore`. For example:

```bash
git add README.md .gitignore
```

Check again:

```bash
git status
```

### Expected result
The selected files appear under **Changes to be committed**.

> Avoid blindly running `git add .` in a real project until you have checked what files are present. Explicit filenames are easier for a beginner to verify.

---

# Step 7: Commit the change

A commit is a saved checkpoint in Git history.

```bash
git commit -m "Update homepage sample and ignore secret files"
```

Check the result:

```bash
git status
```

You can also view the latest commit:

```bash
git log -1 --oneline
```

### Expected result
Your new commit appears at the top of the log.

---

# Step 8: Push the feature branch to GitHub

For a cloned repository that already has an `origin` remote, run:

```bash
git push -u origin feature/200-ui-homepage
```

The `-u` links your local branch with the remote branch. After that, future pushes from the same branch can normally use:

```bash
git push
```

Verify the remote configuration if needed:

```bash
git remote -v
```

> If `git remote -v` shows no GitHub remote, add the remote URL supplied for your lab before pushing. Do not invent a repository URL.

---

# Step 9: Create a Pull Request

After the branch is pushed:

1. Open the repository on GitHub.
2. Locate the pushed branch `feature/200-ui-homepage`.
3. Choose the option to create a **Pull Request** for that branch.
4. Set the base branch required by the lab, commonly `main`.
5. Give the Pull Request a clear title, for example `Update homepage sample`.
6. In the description, briefly explain what changed and how you checked it.
7. Submit the Pull Request for review.

Example description:

```text
What changed:
- Added a small README/homepage practice change.
- Added common secret-file patterns to .gitignore.

Validation:
- Checked git status before staging.
- Confirmed the commit on the feature branch.
```

> Do not merge the Pull Request unless the lab or reviewer specifically asks you to do so.

---

# Step 10: Final verification

Run these commands locally:

```bash
git branch --show-current
git status
git log -1 --oneline
git remote -v
```

You should be able to verify:

- You are working on the feature branch.
- Your intended changes were committed.
- No credential file was intentionally committed.
- The repository has the expected GitHub remote.
- The feature branch exists on GitHub.
- A Pull Request has been created for review.

---

# Easy mental model

```text
Repository = project folder tracked by Git
Branch     = safe workspace for one change
Status     = what changed?
Stage      = what do I want in the checkpoint?
Commit     = save the checkpoint locally
Push       = send the branch to GitHub
Pull Request = ask others to review the branch
```

The workflow is:

```text
main
  |
  +--> create feature branch
          |
          +--> edit file
          +--> git status
          +--> git add
          +--> git commit
          +--> git push
                  |
                  +--> GitHub Pull Request
                           |
                           +--> review
```

---

# Common beginner problems

## `git: command not found`
Git is not installed or is not available in the terminal's PATH. Install/configure Git using the method approved for your machine, then reopen the terminal.

## `fatal: not a git repository`
You are probably outside the repository folder.

Check the current directory and files:

```bash
pwd
ls
```

Then `cd` into the cloned/created repository.

## Push asks for authentication or fails
Do not put a password or token into a source file. Use the GitHub authentication method approved for the lab or your organization.

## A secret file appears in `git status`
Do not stage or commit it. Add the correct pattern to `.gitignore`, then run `git status` again.

## I accidentally worked on `main`
Do not push immediately. Preserve the change on a feature branch first when it has not been shared:

```bash
git switch -c feature/200-ui-homepage
```

Then check:

```bash
git status
git log -1 --oneline
```

If changes have already been pushed or credentials were committed, follow the lab/organization recovery procedure rather than rewriting shared history without guidance.

---

# Lab checklist

- [ ] Git username configured.
- [ ] Git email configured.
- [ ] Repository created or cloned.
- [ ] Feature branch created with a clear name.
- [ ] Small sample change made.
- [ ] `git status` checked before staging.
- [ ] `.env` and other secret-file patterns added to `.gitignore`.
- [ ] Intended files staged.
- [ ] Change committed with a clear message.
- [ ] Feature branch pushed to GitHub.
- [ ] Pull Request created for review.
- [ ] Required proof/submission link prepared.

---

# Submission note

The task screen asks for a shareable verification link. Use the exact deliverable required by your course/instructor. For this Git/GitHub task, a reviewer may logically need the GitHub Pull Request or repository evidence, but the screenshot's submission box specifically mentions a Google Drive/architecture-diagram link. Because those two instructions do not fully match, follow the course's authoritative submission guidance rather than guessing.

---

## Source note

The visible task screenshot defines the five lab actions: configure Git and create/clone a repository; create a feature branch; make, inspect, stage and commit a small change; ignore `.env`/secret files; and push the feature branch and create a Pull Request.

The attached FDE podcast preparation pack is broader project context and discusses Git/CI/CD and credential hygiene, but it is not the `Enterprise_Git_Playbook.pdf` named by this lab task. The detailed beginner explanations and commands above are supplemental guidance so the task can be followed step by step.
