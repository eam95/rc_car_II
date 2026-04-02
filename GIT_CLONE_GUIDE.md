# Git Clone Guide — Cloning Projects & Recovering Work from Branches

> **Real-world example:** This guide documents exactly what we did on April 2, 2026 to clone the `rc_car_II` repo onto a new Debian machine, set up SSH, and recover work from the `feature/rf_timing` branch.

---

## Table of Contents

- [1. SSH Key Setup (New Machine)](#1-ssh-key-setup-new-machine)
- [2. Cloning a Repo](#2-cloning-a-repo)
- [3. Cloning into a Specific Folder Name](#3-cloning-into-a-specific-folder-name)
- [4. Replacing an Existing Folder with a Fresh Clone](#4-replacing-an-existing-folder-with-a-fresh-clone)
- [5. Checking Out a Remote Branch](#5-checking-out-a-remote-branch)
- [6. Recovering Commits from a Remote Branch](#6-recovering-commits-from-a-remote-branch)
- [7. Importing into STM32CubeIDE After Cloning](#7-importing-into-stm32cubeide-after-cloning)
- [8. Common Mistakes & Fixes](#8-common-mistakes--fixes)
- [9. Full Example: What We Did Step by Step](#9-full-example-what-we-did-step-by-step)
- [Quick Reference Table](#quick-reference-table)

---

## 1. SSH Key Setup (New Machine)

When cloning via SSH (`git@github.com:...`) from a new computer, you need an SSH key registered with GitHub.

### Step 1: Generate the key

```bash
ssh-keygen -t ed25519 -C "eam95@users.noreply.github.com"
```

- Press **Enter** to accept the default file location (`~/.ssh/id_ed25519`)
- Enter a passphrase (optional but recommended)

### Step 2: Start the SSH agent and add your key

```bash
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_ed25519
```

### Step 3: Copy the public key

```bash
cat ~/.ssh/id_ed25519.pub
```

Copy the entire output (starts with `ssh-ed25519` and ends with your email).

### Step 4: Add the key to GitHub

1. Go to [https://github.com/settings/keys](https://github.com/settings/keys)
2. Click **New SSH key**
3. Give it a title (e.g., "Debian mongoose workstation")
4. Paste the public key
5. Click **Add SSH key**

### Step 5: Test the connection

```bash
ssh -T git@github.com
```

Expected output:

```
Hi eam95! You've successfully authenticated, but GitHub does not provide shell access.
```

> **Note:** If you already have a key (`~/.ssh/id_ed25519 already exists`), you can either reuse it or overwrite it. If you overwrite, you must re-add the new public key to GitHub.

---

## 2. Cloning a Repo

### Basic clone (creates a folder with the repo name)

```bash
git clone git@github.com:eam95/rc_car_II.git
```

This creates a folder called `rc_car_II/` in your current directory.

### Clone via HTTPS (no SSH key needed, but requires password/token)

```bash
git clone https://github.com/eam95/rc_car_II.git
```

---

## 3. Cloning into a Specific Folder Name

If your repo is named `rc_car_II` but you want the folder to be called `rc_car_integrated_II`:

```bash
git clone git@github.com:eam95/rc_car_II.git rc_car_integrated_II
```

This clones into `rc_car_integrated_II/` instead of `rc_car_II/`.

---

## 4. Replacing an Existing Folder with a Fresh Clone

If you already have a folder and want to start fresh from the repo:

```bash
cd ~/Desktop/STM32_Workspace
rm -rf rc_car_integrated_II
git clone git@github.com:eam95/rc_car_II.git rc_car_integrated_II
```

> ⚠️ **Warning:** `rm -rf` permanently deletes the folder and all its contents. Only use this if everything you need is already pushed to the remote repo.

### Safer alternative (preserve local files, bring in repo history)

```bash
cd ~/Desktop/STM32_Workspace
git clone git@github.com:eam95/rc_car_II.git rc_car_II_temp
cp -r rc_car_II_temp/.git rc_car_integrated_II/
cd rc_car_integrated_II
git reset --hard HEAD
rm -rf ~/Desktop/STM32_Workspace/rc_car_II_temp
```

This copies the `.git` history into your existing folder and resets files to match the repo.

---

## 5. Checking Out a Remote Branch

After cloning, you land on the default branch (usually `main`). Your work may be on a different branch.

### See all remote branches

```bash
git branch -r
```

Example output:

```
  origin/HEAD -> origin/main
  origin/feature/rf_timing
  origin/main
```

### Check out a remote branch locally

```bash
git checkout feature/rf_timing
```

Git will automatically track `origin/feature/rf_timing`.

### Create a new local branch (does NOT track the remote)

```bash
git checkout -b feature/rf_timing
```

> ⚠️ **This is what caused our issue!** Using `-b` creates a **new** branch from your current position (`main`). It does **NOT** pull commits from the remote `feature/rf_timing` branch.

### The difference:

| Command | What it does |
|---------|-------------|
| `git checkout feature/rf_timing` | Checks out the **existing remote branch** with all its commits |
| `git checkout -b feature/rf_timing` | Creates a **new branch** from current HEAD (ignores remote) |

---

## 6. Recovering Commits from a Remote Branch

If you used `git checkout -b` and your branch is missing commits, here's how to recover them:

### Step 1: Fetch all remote data

```bash
git fetch --all
```

### Step 2: See all remote branches

```bash
git branch -r
```

### Step 3: See commits across ALL branches

```bash
git log --oneline --all -10
```

Example output showing the missing commits:

```
16d2a35 (origin/feature/rf_timing) Adjusted the data into 32 byte format...
8ae069f Added red led status to check if transmitter mode is set...
7a18721 (HEAD -> feature/rf_timing, origin/main, main) Update NRF24 library
```

### Step 4: Reset your local branch to match the remote

```bash
git checkout feature/rf_timing
git reset --hard origin/feature/rf_timing
```

Now your local branch matches the remote exactly, including all the missing commits.

---

## 7. Importing into STM32CubeIDE After Cloning

After cloning or re-cloning, Eclipse/STM32CubeIDE may not see the updated files.

### If the project is already in Eclipse:

1. **Right-click** the project in Project Explorer → **Refresh** (or press **F5**)
2. If that doesn't work, close the `main.c` tab and reopen it

### If Eclipse lost the project reference (you deleted and re-cloned the folder):

1. **Right-click** the project → **Delete**
   - ⚠️ **Uncheck** "Delete project contents on disk" → Click **OK**
2. **File → Import → General → Existing Projects into Workspace**
3. Set root directory to your project folder (e.g., `~/Desktop/STM32_Workspace/rc_car_integrated_II`)
4. Make sure the project is checked → Click **Finish**

### If the above doesn't find the project:

Verify the `.project` and `.cproject` files exist:

```bash
ls -la ~/Desktop/STM32_Workspace/rc_car_integrated_II/.project
ls -la ~/Desktop/STM32_Workspace/rc_car_integrated_II/.cproject
```

These files are required for Eclipse to recognize the project.

---

## 8. Common Mistakes & Fixes

### ❌ Mistake: `cd cd ~/some/path` (double cd)

```bash
# WRONG — gives "bash: cd: too many arguments"
cd cd ~/Desktop/STM32_Workspace

# CORRECT
cd ~/Desktop/STM32_Workspace
```

### ❌ Mistake: `git checkout -b` when you meant to track a remote branch

```bash
# WRONG — creates a new empty branch from main
git checkout -b feature/rf_timing

# CORRECT — checks out the existing remote branch
git checkout feature/rf_timing

# FIX — if you already made the mistake
git fetch --all
git reset --hard origin/feature/rf_timing
```

### ❌ Mistake: Cloning but only seeing `main` commits

After cloning, you're on `main` by default. Your work is on another branch. You need to:

```bash
git fetch --all
git branch -r              # see what branches exist
git checkout feature/rf_timing   # switch to your branch (without -b)
```

### ❌ Mistake: Eclipse shows stale files after re-cloning

Delete the project from Eclipse (keep files on disk), then re-import:

1. Right-click project → **Delete** (uncheck "delete contents on disk")
2. **File → Import → Existing Projects into Workspace**
3. Browse to the project folder → **Finish**

---

## 9. Full Example: What We Did Step by Step

This is the exact sequence we followed on April 2, 2026 to set up a new Debian machine and recover work from the `feature/rf_timing` branch.

### Step 1: Generated SSH key on the new machine

```bash
ssh-keygen -t ed25519 -C "eam95@users.noreply.github.com"
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_ed25519
cat ~/.ssh/id_ed25519.pub
```

Added the public key to GitHub at https://github.com/settings/keys.

### Step 2: Tested SSH connection

```bash
ssh -T git@github.com
# Output: Hi eam95! You've successfully authenticated...
```

### Step 3: Cloned the repo into the project folder

```bash
cd ~/Desktop/STM32_Workspace
rm -rf rc_car_integrated_II
git clone git@github.com:eam95/rc_car_II.git rc_car_integrated_II
```

### Step 4: Verified the clone

```bash
cd ~/Desktop/STM32_Workspace/rc_car_integrated_II
git status
git log --oneline -5
```

Output showed we were on `main` with latest commit `7a18721 Update NRF24 library`.

### Step 5: Created a local branch (THIS WAS THE MISTAKE)

```bash
git checkout -b feature/rf_timing
```

This created a **new** branch from `main` instead of tracking the remote `origin/feature/rf_timing`. The commits from last night were missing!

### Step 6: Discovered the missing commits

```bash
git fetch --all
git branch -r
```

Output:

```
  origin/HEAD -> origin/main
  origin/feature/rf_timing    ← our work is here!
  origin/main
```

### Step 7: Verified the commits existed on the remote

```bash
git log --oneline --all -10
```

Output:

```
16d2a35 (origin/feature/rf_timing) Adjusted the data into 32 byte format...
8ae069f Added red led status to check if transmitter mode is set...
7a18721 (HEAD -> feature/rf_timing, origin/main, main) Update NRF24 library
```

### Step 8: Recovered the commits

```bash
git checkout feature/rf_timing
git reset --hard origin/feature/rf_timing
```

Output:

```
HEAD is now at 16d2a35 Adjusted the data into 32 byte format for transmitt mode...
```

✅ All commits recovered!

### Step 9: Refreshed the project in STM32CubeIDE

1. Right-click project → **Delete** (unchecked "delete contents on disk")
2. **File → Import → General → Existing Projects into Workspace**
3. Browsed to `~/Desktop/STM32_Workspace/rc_car_integrated_II`
4. Clicked **Finish**

### The correct way (what we should have done at Step 5):

```bash
# Instead of: git checkout -b feature/rf_timing  ← WRONG
# Use:
git checkout feature/rf_timing                    # ← CORRECT (tracks remote)
```

---

## Quick Reference Table

| Task | Command |
|------|---------|
| Generate SSH key | `ssh-keygen -t ed25519 -C "email@example.com"` |
| Start SSH agent | `eval "$(ssh-agent -s)" && ssh-add ~/.ssh/id_ed25519` |
| Copy public key | `cat ~/.ssh/id_ed25519.pub` |
| Test SSH connection | `ssh -T git@github.com` |
| Clone a repo | `git clone git@github.com:user/repo.git` |
| Clone into custom folder | `git clone git@github.com:user/repo.git my_folder_name` |
| Delete & re-clone | `rm -rf folder && git clone git@github.com:user/repo.git folder` |
| List remote branches | `git branch -r` |
| Checkout existing remote branch | `git checkout branch_name` |
| Fetch all remote data | `git fetch --all` |
| See all commits (all branches) | `git log --oneline --all -10` |
| Reset local branch to remote | `git reset --hard origin/branch_name` |
| Refresh Eclipse project | Right-click → Refresh (F5) |
| Re-import Eclipse project | File → Import → Existing Projects into Workspace |

---

## Your Repo Info

- **GitHub URL:** https://github.com/eam95/rc_car_II
- **SSH Clone:** `git clone git@github.com:eam95/rc_car_II.git`
- **Branches:** `main`, `feature/rf_timing`
- **Board:** STM32H723ZG (Nucleo-H723ZG)
- **IDE:** STM32CubeIDE
- **Local Path:** `~/Desktop/STM32_Workspace/rc_car_integrated_II`
