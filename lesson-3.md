# Git Workflow Exercises

## Remote Git Repository Usage

### Creating an Account

1. Go to one of the following:
    - Bitbucket: [bitbucket.org](https://bitbucket.org/)
    - Azure DevOps: [dev.azure.com](https://dev.azure.com/)
2. Sign up for a free account and log in.

---

### Creating a Project and Repository

1. Create a project named `automation`.

2. Create a repository named `egitim_demo` under the `automation` project.

3. Create the repository with a `README.md` file from the start:
    - While creating the repository, enable **Initialize repository with a README**
    - This creates the repository on the default branch (`main`) with an initial `README.md`

4. Update `README.md` and create your first manual commit:
    - Go to **Source** (or **Files**) and open `README.md`
    - Click **Edit** and update the content:
        ```md
        # egitim_demo

        Initial README update
        ```
    - Commit message: `update README`
    - Commit directly to the default branch (`main`)

5. Explore the repository UI:
    - Navigate to **Branches** via the left menu and notice that the default branch now exists.
    - Navigate to **Commits** via the left menu and confirm that both the automatic initial commit and your README update commit are visible.

6. Create a new branch named `dev` via the UI:
    - Click **Branches** → **Create branch**
    - Branch name: `dev`
    - Branch from: the default branch (`main` or `master`)

7. Add a file named `deployment.yaml` to the `dev` branch via the UI:
    - Go to **Source** → select the `dev` branch → click **Add file**
    - File name: `deployment.yaml`
    - Content:
        ```yaml
        apiVersion: apps/v1
        kind: Deployment
        metadata:
          name: demo-deployment
          labels:
            app: demo
        spec:
          replicas: 1
          selector:
            matchLabels:
              app: demo
          template:
            metadata:
              labels:
                app: demo
            spec:
              containers:
              - name: demo
                image: nginx:latest
                ports:
                - containerPort: 80
        ```
    - Commit message: `add initial deployment.yaml`
    - Commit directly to `dev` branch.

8. Verify the commit appeared under **Commits**.

9. Compare branches:
    - In `egitim_demo`, go to **Branches** (left menu) → click the three dots (`...`) next to a branch → **More options** → **Compare branches**
    - Select `dev` vs the default branch to see the diff

10. Manage notifications:
    - **Bitbucket:** Go to **Source** → three dots menu → **Manage notifications**
    - **Azure DevOps:** Go to **Project Settings** → **Notifications** → **New subscription**
        - Create a subscription to get notified only when the default branch is updated:
            | Field | Value |
            |---|---|
            | Description | `A commit is pushed` |
            | Subscriber | `automation Team` |
            | Deliver to | `Custom email address` |
            | Address | `your@email.com` |
            | Filter | `A specific team project` → `automation` |
            | Filter criteria — Field | `Branches updated` |
            | Filter criteria — Operator | `Contains` |
            | Filter criteria — Value | your default branch name (for example `main`) |
        - Click **Finish** to save.
        - Make a commit to the `dev` branch and verify that no email arrives.
        - Make a commit to the default branch (for example, edit `README.md` via the UI) and verify that the notification email arrives.

11. Explore the `demo_app` repository (pre-created for demo purposes):
    - Show commit history under **Commits**.
    - Add a new file named `deployment.yaml` via the UI with the same content as step 7.
    - Show all commits across all branches via **Commits** → **All branches**.

---

### Pull Request Workflow

> Goal: clone the repo, make a change on a feature branch, push it, and open a pull request.

1. Copy the clone URL from the repository UI (`Clone` button).

2. Clone the repository:
    ```bash
    git clone <url>
    cd egitim_demo
    ```

3. List branches and switch to `dev`:
    ```bash
    git branch          # show local
    git branch -r       # show remote
    git branch -a       # show all local and remote branches
    git checkout dev
    ```

4. Create a feature branch from `dev`:
    ```bash
    git checkout -b feature/deployment-task
    ```

5. Edit `deployment.yaml` — change `replicas` from `1` to `5`:
    ```yaml
    spec:
      replicas: 5   # was 1
    ```

6. Stage and commit:
    ```bash
    git add deployment.yaml
    git commit -m "increase replica count to 5"
    ```

7. Try pushing without setting upstream (to show the error):
    ```bash
    git push
    # Error: no upstream branch set
    git remote -v   # show where origin points
    ```

8. Push with upstream:
    ```bash
    git push --set-upstream origin feature/deployment-task
    ```

9. In the repository UI:
    - Go to **Branches** — confirm `feature/deployment-task` is visible.
    - Click **Create pull request** (the banner that appears automatically, or go to **Pull requests** → **New**).
    - Source branch: `feature/deployment-task` → Target branch: `dev`
    - Title: `Increase replica count to 5`
    - Click **Create**.

10. Review the diff in the PR, then click **Merge**.

---

## Team Workflow and Conflict Resolution

> Scenario: Ali and Veli both branch from the same `dev` commit. Ali merges first. Veli continues on the older base, so his pull request later conflicts with `Dockerfile`.

### Setup (before Ali and Veli start)

1. In the remote repository UI, create a `Dockerfile` in the `dev` branch:
    - Go to **Source** → select `dev` branch → **Add file**
    - File name: `Dockerfile`
    - Content:
        ```dockerfile
        FROM alpine:latest
        COPY . /app
        WORKDIR /app
        CMD ["echo", "Hello, World!"]
        ```
    - Commit message: `add initial Dockerfile`

2. If you are doing this alone, use two separate folders or terminals before merging anything:
    - one folder for Ali
    - one folder for Veli
    - create Veli's branch before merging Ali's pull request

### Ali's Steps

3. Clone the repository:
    ```bash
    git clone <repo-url>
    cd egitim_demo
    ```

4. Switch to `dev` and create Ali's feature branch:
    ```bash
    git checkout dev
    git pull
    git checkout -b feature/ali
    ```

5. Create `deployment-ali.yaml`:
    ```yaml
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: ali-deployment
      labels:
        app: ali
    spec:
      replicas: 3
      selector:
        matchLabels:
          app: ali
      template:
        metadata:
          labels:
            app: ali
        spec:
          containers:
          - name: ali-app
            image: nginx:latest
            ports:
            - containerPort: 80
    ```

6. Edit `Dockerfile` — change the last line:
    ```dockerfile
    FROM alpine:latest
    COPY . /app
    WORKDIR /app
    CMD ["echo", "Hello, Ali!"]
    ```

7. Commit and push:
    ```bash
    git add deployment-ali.yaml Dockerfile
    git commit -m "add ali deployment and update Dockerfile"
    git push --set-upstream origin feature/ali
    ```

- Do not merge Ali's pull request yet. First create Veli's branch from the old `dev` state.

---

### Veli's Steps

> Veli starts from the same original `dev` commit as Ali. He does **not** pull again after Ali merges, so he is unaware of Ali's `Dockerfile` change.

8. Before Ali's pull request is merged, clone the repository (or use a separate folder):
    ```bash
    git clone <repo-url> egitim_demo_veli
    cd egitim_demo_veli
    git checkout dev
    git checkout -b feature/veli
    ```

9. Create `deployment-veli.yaml`:
    ```yaml
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: veli-deployment
      labels:
        app: veli
    spec:
      replicas: 3
      selector:
        matchLabels:
          app: veli
      template:
        metadata:
          labels:
            app: veli
        spec:
          containers:
          - name: veli-app
            image: nginx:latest
            ports:
            - containerPort: 80
    ```

10. Edit `Dockerfile` — change the last line:
    ```dockerfile
    FROM alpine:latest
    COPY . /app
    WORKDIR /app
    CMD ["echo", "Hello, Veli!"]
    ```

11. Commit and push:
    ```bash
    git add deployment-veli.yaml Dockerfile
    git commit -m "add veli deployment and update Dockerfile"
    git push --set-upstream origin feature/veli
    ```

12. Now merge Ali's pull request first:
    - In the UI: create a pull request from `feature/ali` → `dev`, then merge it.

13. Create or refresh Veli's pull request from `feature/veli` → `dev`.
    - You will see: **`This pull request can't be merged`** — because `Dockerfile` was already changed by Ali.

### Resolve the Conflict (Veli)

14. Pull the latest `dev` into Veli's branch and merge:
    ```bash
    git checkout dev
    git pull
    git checkout feature/veli
    git merge dev
    ```
    - Git will report a conflict in `Dockerfile`.

15. Open `Dockerfile` — you will see conflict markers:
    ```
    <<<<<<< HEAD
    CMD ["echo", "Hello, Veli!"]
    =======
    CMD ["echo", "Hello, Ali!"]
    >>>>>>> dev
    ```
    - Decide which line to keep (or combine), then remove the conflict markers.
    - Final version (keeping Veli's line):
    ```dockerfile
    FROM alpine:latest
    COPY . /app
    WORKDIR /app
    CMD ["echo", "Hello, Veli!"]
    ```

16. Stage the resolved file and commit:
    ```bash
    git add Dockerfile
    git commit -m "resolve merge conflict in Dockerfile"
    git push
    ```

17. In the UI: the PR is now mergeable — click **Merge**.

---

## Pushing a Local Repository to Remote

> Goal: initialize a repo locally and push it to Bitbucket/Azure DevOps for the first time (no remote repo was cloned).

### Prerequisites
- You have a Bitbucket or Azure DevOps account.
- You have created an empty repository on Bitbucket/Azure DevOps named `from_local_demo` (no README, no `.gitignore` — keep it empty).

### Step 1: Configure Your Identity
```bash
git config --global user.name "Your Name"
git config --global user.email "your_email@example.com"
```

### Step 2: Create a Local Repository
```bash
mkdir from_local_demo && cd from_local_demo
git init
touch README.md
git add README.md
git commit -m "Added README.md"
git checkout -b dev
git branch
```

#### README.md — Markdown Basics

Open `README.md` and try the following examples one by one.

The full content to paste into `README.md`:

````
# Project Title

## Description
This is **bold**, this is *italic*, this is ***bold and italic***.

## Features
- Feature one
- Feature two
  - Sub-feature

## Numbered List
1. First step
2. Second step

## Inline Code
Inline: `print('hello world')`

## Link and Image
[Google](https://www.google.com)
![Alt text](image.png)

## Table
| Name  | Role |
|-------|------|
| Ali   | Dev  |
| Veli  | QA   |

## Blockquote
> This is a note or a warning.

## Horizontal Rule
---
````

Commit the updated README:
```bash
git add README.md
git commit -m "add markdown examples to README"
```

### Step 3: Add a File and Commit
Create a file named `test.py`:
```python
print('hello world')
```

```bash
git add test.py
git commit -m "initial commit: add test.py"
git log
```

### Step 3b: Make Additional Commits
Add a second line to `test.py`:
```python
print('hello world')
print('line-2')
```

```bash
git add test.py
git commit -m "added line-2"
git log
```

Add a third line to `test.py`:
```python
print('hello world')
print('line-2')
print('line-3')
```

```bash
git add test.py
git commit -m "added line-3"
git log
```

### Step 4: Connect to the Remote Repository
```bash
git remote add origin https://<username>@bitbucket.org/<username>/from_local_demo.git
# Azure DevOps example:
# git remote add origin https://dev.azure.com/<org>/<project>/_git/from_local_demo
git branch
git branch -r
git remote -v    # verify the remote URL is correct
```

### Step 5: Push to Remote
```bash
git push -u origin dev
```

- Go to the Bitbucket/Azure DevOps repository UI.
- Confirm the `dev` branch exists and the full commit history is visible, including the README and `test.py` commits.

---

## git fetch vs git pull

> **Key concept from the diagram:**
> - `git fetch` — downloads changes from the Remote Git repository into your **local Git repository** only. Your working directory is untouched. You decide when (and whether) to merge.
> - `git pull` — does `git fetch` **+** `git merge` in one step. It downloads and immediately applies changes to your working directory.

### Setup: Simulate a Remote Change

Before starting, make a change directly in the remote repository UI (Bitbucket/Azure DevOps) so there is something to fetch:

1. Open the `from_local_demo` repository in the UI.
2. Find `test.py` on the `dev` branch → click **Edit**.
3. Add a new line at the bottom:
    ```python
    print('added from remote UI')
    ```
4. Commit with message: `add line from remote UI`

Now the remote is **1 commit ahead** of your local repo.

---

### Part 1: git fetch (download only, do NOT merge)

```bash
# Check current state — local is behind
git log --oneline

# Fetch: download the remote commit into your local repo
git fetch origin

# The remote commit is now stored locally under origin/dev
# BUT your working directory has NOT changed yet
git log --oneline          # still shows old commits
git log --oneline origin/dev  # shows the new commit from the UI
```

Compare what changed:
```bash
git diff HEAD origin/dev   # diff between local and remote
```

Manually merge when you are ready:
```bash
git merge origin/dev
git log --oneline          # now includes the remote commit
```

---

### Part 2: git pull (fetch + merge in one step)

First, simulate another remote change:
1. Edit `test.py` in the UI again → add: `print('second remote line')`
2. Commit with message: `add second line from remote UI`

Now pull:
```bash
git pull
git log --oneline   # remote commit is immediately visible in working directory
```

---

### Summary

| Command | Downloads to local repo | Merges into working directory |
|---|---|---|
| `git fetch` | Yes | No — you merge manually |
| `git pull`  | Yes | Yes — automatic |

**When to use `git fetch`:** You want to inspect what changed before merging (safer in shared branches).

**When to use `git pull`:** You trust the remote changes and want to update immediately (common on personal branches).

---

## Repository Settings for Bitbucket

1. Go to **Repository settings** (left sidebar, bottom).
2. Under **User and group access** → add a user and assign `read`, `write`, or `admin` permission.
3. Under **Access keys** → add a deploy key (SSH public key) for read-only CI access.
4. Under **Branch restrictions** → set rules for the `main` branch (or a branch pattern):
    - Restrict direct pushes.
    - Control who can merge.
    - Add merge checks such as minimum approvals, resolved tasks, or passing builds.
5. Under **Pull request and merge settings** → choose the repository-level merge strategies users can select in pull requests:

- **Merge commit** — preserves all commits, creates a merge commit:
    ```
    A---B---C---D---E (main)
        \           /
         F---G---H (feature)
    ```
- **Squash** — all PR commits are squashed into one commit on `main`:
    ```
    A---B---C---D---E---S (main)
                        |
                        S = squashed from F+G+H
    ```
- **Fast Forward** — only allowed when there are no new commits on `main`; no merge commit created:
    ```
    A---B---C---F---G---H (main, after fast-forward)
    ```

6. **Project settings** → configure project-level permissions and defaults.
7. **Webhooks** → add a webhook URL to trigger external services (e.g., CI/CD) on push or PR events.
8. **Downloads** (left menu) → attach release artifacts or binaries to the repository.

---

## Repository Settings for Azure DevOps

1. Go to **Project Settings** → **Repositories**.
2. Under **Repositories** → select a repository → configure permissions such as `Read`, `Contribute`, `Create branch`, and `Create tag`.
3. Protect the default branch with **Policies** / **Branch policies**:
    - Require a minimum number of reviewers.
    - Check for linked work items.
    - Check for comment resolution.
    - Limit merge types.
    - Add build validation or status checks when needed.
4. Review repository-level settings such as the default branch and inherited policies.
5. Under **Project Settings** → **Service hooks** → **Web Hooks**, send push or pull request events to external systems.

---

## .gitignore

### Introduction
`.gitignore` tells Git which files to never track. These are typically build artifacts, secrets, logs, and OS-generated files.

### Step 1: Create and commit `.gitignore`
```bash
touch .gitignore
git add .gitignore
git commit -m "add .gitignore"
```

### Step 2: Basic rules — pattern types

Open `.gitignore` and add the following rules one by one to see each in action:

#### Pattern 1: `*.log` — ignore by extension
Ignores any file ending with `.log`, regardless of name.
```
*.log
```
```bash
touch app.log
touch error.log
git status --ignored   # app.log and error.log → ignored
```

---

#### Pattern 2: `logs/` — ignore a directory
Ignores the entire `logs/` folder and everything inside it.
```
logs/
```
```bash
mkdir logs && touch logs/server.log
git status --ignored   # logs/ → ignored
```

---

#### Pattern 3: `**/debug.log` — ignore across all subdirectories
`**` matches any number of nested folders. Useful when the same filename can appear anywhere in the project.
```
**/debug.log
```
```bash
mkdir -p src/utils
touch debug.log
touch src/debug.log
touch src/utils/debug.log
git status --ignored   # all three → ignored
```

---

#### Pattern 4: `!` — exception (un-ignore a specific file)
If you want to keep one file inside a folder trackable, ignore the folder contents, not the folder itself:
```
logs/*
!logs/important.log
```
```bash
mkdir -p logs
touch logs/server.log
touch logs/important.log
git status --ignored --short
# ?? logs/
# !! logs/server.log
git add logs/important.log
git status --ignored --short
# A  logs/important.log
# !! logs/server.log
```

---

### Step 3: Confirm non-ignored files still show up
```bash
touch main.py
git status   # main.py appears as untracked — it is NOT in .gitignore
```

---

## SSH Key Setup

> Use SSH instead of HTTPS to avoid entering your password on every push/pull.

### Why SSH? — See the error first

Try cloning with SSH before setting up the key. You will get this error:

```bash
git clone git@ssh.dev.azure.com:v3/<org>/<project>/<repo>
# fatal: Could not read from remote repository.
# remote: Public key authentication failed.
```

This means the remote does not recognize your machine yet. The fix: generate an SSH key, add the public key to your account, then clone again.

---

### Step 1: Generate a new SSH key pair
If you already have an `id_rsa` key, give this one a different name to avoid overwriting it:
```bash
ssh-keygen -t rsa -b 2048
# When prompted for a file name, type a custom path:
# /Users/<username>/.ssh/id_rsa_azure
# Optionally set a passphrase
```

### Step 2: Copy the public key
```bash
cat ~/.ssh/id_rsa_azure.pub
```
Copy the output — this is the key you will paste into Bitbucket or Azure DevOps.

### Step 3: Add the public key to your account

**Bitbucket:** `Avatar` → `Personal settings` → `SSH keys` → `Add key`

**Azure DevOps:** `User settings` (top right) → `SSH public keys` → `New Key`

A dialog will appear with two fields:

| Field | Value |
|---|---|
| Name | `id_rsa_azure.pub` |
| Public Key Data | paste the full output of `cat ~/.ssh/id_rsa_azure.pub` |

Click **Add** to save.

### Step 4: Add the key to your SSH agent
```bash
ssh-add ~/.ssh/id_rsa_azure
```

### Step 5: Configure `~/.ssh/config`

**For Bitbucket:**
```
Host bitbucket.org
    AddKeysToAgent yes
    IdentityFile ~/.ssh/id_rsa_azure
```

**For Azure DevOps:**
```
Host ssh.dev.azure.com
    IdentityFile ~/.ssh/id_rsa_azure
    IdentitiesOnly yes
```

### Step 6: Test the connection
```bash
ssh -T git@bitbucket.org        # should greet you by username
ssh -T git@ssh.dev.azure.com    # should confirm authentication
```

---

## Forking a Repository

> A fork is a copy of someone else's repository under your own account. It allows you to make changes and propose them back to the original project via a pull request — without needing direct access to the original repo.

1. Go to [github.com/rancher/rancher](https://github.com/rancher/rancher)
2. Click **Fork** (top right) → choose your account → click **Create fork**.
3. In your forked repo, open `README.md` via the UI → click the pencil (edit) icon.
4. Add a line at the top:
    ```
    # My Fork Notes
    ```
5. Commit message: `update README with fork notes` → commit directly to `main`.
6. Go to **Pull requests** → **New pull request**.
    - Base repository: `rancher/rancher` (the original)
    - Head repository: `<your-account>/rancher` (your fork)
    - Observe the diff — this is how open-source contributions work.
