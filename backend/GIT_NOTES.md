# Complete Git & GitHub Master Guide (A to Z)

This guide covers the entire lifecycle of version control with **Git** (local) and **GitHub** (remote cloud), pre-push setup, security rules, troubleshooting, and daily workflows.

---

## 1. The Core Mental Model: The 4 Areas of Git

To understand Git, you must understand the 4 places your code lives:

```
+-----------------------------------------------------------------------------------+
|  1. Working Directory  ==> (git add .) ==>  2. Staging Area (Index)              |
|     (Your actual files)                        (Files prepared for snapshot)      |
|                                                          |                        |
|                                                    (git commit)                   |
|                                                          v                        |
|  4. Remote (GitHub)   <== (git push) <==   3. Local Repository (.git database)   |
|     (Cloud backup/sharing)                     (Permanent committed history)      |
+-----------------------------------------------------------------------------------+
```

1. **Working Directory:** The folder on your computer where you write code. Files can be *untracked*, *modified*, or *unmodified*.
2. **Staging Area (Index):** The waiting room. When you run `git add`, you tell Git: *"These are the specific changes I want to include in my next save point."*
3. **Local Repository:** When you run `git commit`, Git takes a permanent snapshot of everything in the Staging Area and saves it inside the hidden `.git` folder on your machine.
4. **Remote Repository (GitHub):** The online copy on github.com. When you run `git push`, your local commits are uploaded to GitHub.

---

## 2. CRITICAL PRE-PUSH STEP: Protecting Secrets with `.gitignore`

### Why did `.env` end up on GitHub?
Git tracks files based on two simple rules:
1. If a file is not in `.gitignore`, Git considers it eligible to be tracked.
2. **If Git has already started tracking a file, adding it to `.gitignore` later will NOT stop Git from tracking it!**

If you ran `git add .` before you created or updated your `.gitignore` file, your `.env` was committed and pushed to GitHub.

---

### How to Create `.gitignore` (Always Do This FIRST)
In the root directory of your project, create a file named exactly:
```text
.gitignore
```

Add the following standard entries:

```gitignore
# Environment & Secrets (NEVER PUSH THESE)
.env
.env.local
.env.*.local
*.env

# Dependencies / Packages (Massive folders; easily re-installed)
node_modules/

# Production Build outputs
dist/
build/
.next/
out/

# System and Editor files
.DS_Store
Thumbs.db
.vscode/
.idea/

# Logs
npm-debug.log*
yarn-debug.log*
yarn-error.log*
```

---

### EMERGENCY FIX: How to Remove `.env` from GitHub Without Deleting it Locally

If you already pushed `.env` to GitHub and need it gone from GitHub while keeping your local `.env` intact on your computer, run these exact steps in your terminal:

```bash
# Step 1: Tell Git to stop tracking .env (removes it from Git index, keeps the local file)
git rm --cached .env

# (If you have multiple env files like .env.local, run):
git rm --cached .env*

# Step 2: Make sure .env is written inside your .gitignore file!
# Open .gitignore and confirm ".env" is listed.

# Step 3: Commit the removal
git commit -m "fix: stop tracking .env and keep it ignored"

# Step 4: Push the change to GitHub
git push
```

> [!WARNING]
> **Security Note:** If your GitHub repository was **public** when you pushed `.env`, anyone could have seen your database passwords, API keys, or JWT secrets. You must **rotate / regenerate** those passwords and API keys immediately, because Git commit history retains past commits unless history is completely rewritten.

---

## 3. One-Time Computer Setup (Identity & Authentication)

Before using Git on any computer, your identity must be configured:

### A. Set Global Name and Email
Git attaches your name and email to every commit you make:
```bash
git config --global user.name "Your GitHub Username or Real Name"
git config --global user.email "your-github-registered-email@example.com"
```

To verify your configuration:
```bash
git config --global --list
```

### B. Authentication with GitHub
When you push to GitHub for the first time on Windows:
* Git uses the **Git Credential Manager (GCM)**.
* A browser window or dialog will pop up asking you to **Sign in with your browser** or use a **Personal Access Token (PAT)**.
* Once logged in, Windows Credential Manager securely saves your token so you never have to re-enter your password on that machine.

---

## 4. Setting Up Git & GitHub from A to Z (The Complete Step-by-Step)

Follow this complete checklist every time you start a brand new project.

### Phase 1: Local Preparation
1. **Open your project folder** in VS Code or your terminal.
2. **Create your `.gitignore` file** and add `.env`, `node_modules/`, etc. (See Section 2).
3. **Verify Git is not already initialized:**
   ```bash
   git status
   ```
   If it says `fatal: not a git repository`, you are ready for Phase 2.

---

### Phase 2: Initialize and Make Your First Local Commit
4. **Initialize Git repository:**
   ```bash
   git init
   ```
   *(This creates the invisible `.git` folder in your project).*

5. **Check status before adding:**
   ```bash
   git status
   ```
   *Look at the list of untracked files. Check that `.env` and `node_modules` are NOT in the list! If they are, check your `.gitignore`.*

6. **Stage all files for the first snapshot:**
   ```bash
   git add .
   ```

7. **Verify staged files:**
   ```bash
   git status
   ```
   *All files to be committed should now appear in **green**.*

8. **Make your first commit:**
   ```bash
   git commit -m "feat: initial commit"
   ```

9. **Ensure the branch is named `main`:**
   ```bash
   git branch -M main
   ```
   *(Git historically created `master`. GitHub uses `main` by default. This command guarantees you are on `main`).*

---

### Phase 3: Create GitHub Repo & Connect It
10. **Go to GitHub.com:**
    * Click the **`+`** icon in the top right -> **New repository**.
    * Enter repository name (e.g., `marvelam-frontend`).
    * Set visibility to **Public** or **Private**.
    * **IMPORTANT:** Leave **"Add a README file"**, **"Add .gitignore"**, and **"Choose a license"** **UNCHECKED**. (Checking them creates initial commits on GitHub that conflict with your local project).
    * Click **Create repository**.

11. **Copy the HTTPS URL:**
    * Copy the URL provided by GitHub:
    * `https://github.com/<your-username>/<your-repo-name>.git`

12. **Connect your local folder to GitHub:**
    ```bash
    git remote add origin https://github.com/<your-username>/<your-repo-name>.git
    ```

---

### Phase 4: Confirming the Connection
13. **Verify the remote connection:**
    ```bash
    git remote -v
    ```
    *Output should display both `(fetch)` and `(push)` URLs:*
    ```text
    origin  https://github.com/<your-username>/<your-repo-name>.git (fetch)
    origin  https://github.com/<your-username>/<your-repo-name>.git (push)
    ```

* **What if you typed the URL wrong?**
  Fix it anytime using:
  ```bash
  git remote set-url origin https://github.com/<correct-username>/<correct-repo>.git
  ```

* **What if you want to remove the link completely?**
  ```bash
  git remote remove origin
  ```

---

### Phase 5: The First Push
14. **Push your code and establish upstream tracking:**
    ```bash
    git push -u origin main
    ```
    * **`git push`**: upload local commits to the server.
    * **`-u` (or `--set-upstream`)**: Tells Git: *"Link my local `main` branch to the `main` branch on `origin` (GitHub)."*
    * Because of `-u`, every future push or pull on this branch only requires typing `git push` or `git pull`.

---

## 5. Daily Development Workflow (Add, Commit, Push)

Once your repository is connected, your daily workflow follows this cycle:

```
Write / Edit Code
       |
       v
git status                (See what files changed)
       |
       v
git add .                 (Stage changes for snapshot)
       |
       v
git commit -m "message"   (Take the snapshot locally)
       |
       v
git push                  (Upload snapshot to GitHub)
```

### The 3 Standard Daily Commands:
```bash
# 1. Stage changes
git add .

# 2. Commit with a meaningful message
git commit -m "feat: implement user registration form validation"

# 3. Push to GitHub
git push
```

---

## 6. How to Undo / Fix Common Mistakes

| Scenario | Command to Run | What it Does |
| :--- | :--- | :--- |
| **Accidentally staged a file with `git add`** | `git restore --staged <filename>` | Removes the file from staging area back to working directory. Does NOT delete your work. |
| **Discard unwanted changes in an uncommitted file** | `git restore <filename>` | Discards edits in your working directory and restores file back to the last commit. |
| **Change the message of your most recent commit** | `git commit --amend -m "new message"` | Updates the commit message of your last unpushed commit. |
| **Check recent commit history** | `git log --oneline` | Shows a clean, 1-line list of all previous commits and their IDs. |
| **Check which branch you are on** | `git branch` | Lists local branches; active branch marked with `*`. |
| **Pull latest changes from GitHub** | `git pull` | Downloads and merges changes from GitHub into your local folder. |

---

## 7. Good Commit Message Conventions

Writing clear commit messages helps you and collaborators understand your project history.

Common prefixes:
* `feat:` A new feature or capability (e.g., `feat: add dark mode toggle`)
* `fix:` A bug fix (e.g., `fix: resolve login redirect loop`)
* `style:` Formatting, CSS, or visual tweaks with no logic change (e.g., `style: adjust navbar padding`)
* `refactor:` Code restructuring without changing behavior (e.g., `refactor: clean up API service calls`)
* `docs:` Documentation changes (e.g., `docs: update setup instructions in README`)
* `chore:` Maintenance, updating packages, or `.gitignore` changes (e.g., `chore: ignore .env and build output`)

---

## 8. Repository Audit & Untracking Files from Git & GitHub

### A. Audit of Your Repository: Which Files Should NOT Be Tracked?

After inspecting your project (`Marvelam-Frontend`), here are the files that should **NOT** be tracked:

#### 1. `.env` (CRITICAL / HIGH SECURITY)
* **Status in your repo:** Currently tracked by Git and uploaded to GitHub.
* **Why it shouldn't be tracked:** `.env` contains your backend API URLs, secrets, private tokens, or database credentials. Tracking it exposes your private environment setup.
* **Note:** Even though `.env` was added to `.gitignore`, it remains tracked because it was staged *before* being ignored.

#### 2. Temporary Python Scratch Scripts (`*.py`)
* **Status in your repo:** There are 23 Python scripts in your root directory (`clean.py`, `compose.py`, `disable_eslint.py`, `final_eslint_fix.py`, `fix_buttons.py`, `fix_eslint.py`, `fix_eslint2.py`, `fix_flashsale.py`, `fix_hero_buttons.py`, `fix_hero_mobile.py`, `fix_import.py`, `fix_sliders.py`, `fix_teaser_cards.py`, `modify.py`, `remove_bag.py`, `remove_sidebar.py`, `restore_homepage.py`, `rewrite.py`, `safe_fix.py`, `script.py`, etc.).
* **Why they shouldn't be tracked:** This is a React/Vite frontend project. These Python scripts were one-off helper scripts used during code edits. They are not part of the website, do not run in production, and clutter your clean repository.

#### 3. General Checklist of Files You Should NEVER Track in Frontend Projects:
* **Secrets:** `.env`, `.env.local`, `.env.production`, `*.pem`, `*.key`
* **Dependencies:** `node_modules/` (always install fresh with `npm install`)
* **Build Artifacts:** `dist/`, `build/`, `.next/` (generated automatically during build/deployment)
* **OS / Editor junk:** `.DS_Store`, `Thumbs.db`, `.vscode/` (workspace settings), `.idea/`
* **Logs:** `*.log`, `npm-debug.log*`, `yarn-error.log*`
* **Temporary / Scratch files:** `test.*`, `temp/`, scratch scripts (`*.py` if not part of project pipeline)

---

### B. How to Check What Git is Currently Tracking

To print a complete list of every file Git is currently monitoring:
```bash
git ls-files
```

If you see `.env` or temporary files in this list, Git is actively tracking them.

---

### C. How to Untrack Files and Remove Them from GitHub (Without Deleting Them Locally)

The difference between normal deleting and untracking:
* **`git rm <file>`** : Deletes the file from your computer's hard drive AND deletes it from Git.
* **`git rm --cached <file>`** : **KEEPS** the file on your computer, but tells Git to delete it from version tracking and delete it from GitHub on your next push.

#### Step 1: Untrack the files
Run this in your terminal:
```bash
# 1. Untrack your .env file:
git rm --cached .env

# 2. (Optional) Untrack all one-off python helper scripts:
git rm --cached *.py
```

#### Step 2: Ensure they are in `.gitignore`
Make sure your `.gitignore` has these entries so Git never tries to re-track them:
```gitignore
.env
.env.*
*.py
```

#### Step 3: Commit the removal
```bash
git commit -m "fix: untrack .env and temporary helper scripts"
```

#### Step 4: Push the removal to GitHub
```bash
git push
```

#### Step 5: Verify
* Go to your repository on `github.com`.
* Check the file tree: `.env` and the untracked scripts will no longer be visible on GitHub!
* Check your local folder on your computer: your `.env` file is still safely on your computer.

---

### D. The Golden Rule of Leaked Secrets in Git History

> [!CAUTION]
> **Understanding Git History:**
> Running `git rm --cached .env` and pushing removes the file from the **current** repository state on GitHub.
> 
> However, because Git is a version control system, anyone viewing the repository's past commit history (e.g., clicking on the initial commit) could still see the file contents as it existed back then.
> 
> * **If your repository is PRIVATE:** You are generally safe once untracked, as only you have access to the repository history.
> * **If your repository is PUBLIC:** Assume that bots or scrapers have already recorded any API keys or passwords in that `.env`. **You must immediately rotate or change the exposed API keys/passwords.**
