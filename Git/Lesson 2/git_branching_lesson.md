# Git Branching, Merging, and Linear History

This document summarizes core Git concepts for isolated development, merging, and maintaining an enterprise-grade linear history, along with the practical lab steps and code snippets to execute this workflow.

## 1. Branching Basics
* **Branches are pointers, not folders.** Creating a branch does not duplicate your project folder. It creates a lightweight label that points to a specific commit. 
* **Branch isolation.** When you switch branches using `git switch <branch-name>`, Git actively updates the files in your working directory to match that branch's exact state in history.
* **Git vs. Infrastructure.** Merging code (like Terraform or PowerShell) into `main` only updates the local Git history. It does not inherently deploy anything to Azure unless a CI/CD pipeline is configured to do so. Testing code is still required after a successful Git merge.

**Common Commands:**
```bash
git branch                  # List all local branches
git switch -c <name>        # Create a new branch and switch to it
git switch <name>           # Switch to an existing branch
git branch -d <name>        # Delete a branch safely after merging
```

## 2. Merging and Fast-Forwards (`--ff-only`)
When integrating a feature branch back into `main`, direction matters. You must switch to the destination branch (`main`) before running the merge command.

We enforce the `--ff-only` (fast-forward only) flag to keep our project history linear and readable.

**How Fast-Forward Works:**
If `main` has not changed since you branched off, Git simply slides the `main` pointer forward in a straight line to catch up to your new commits. No extra "merge commits" are created.

```bash
git switch main
git merge --ff-only feature/my-new-script
```

## 3. Diverged Timelines and standard `git merge`
If someone else updates `main` while you are working on your feature branch, the timelines diverge. 

* **Without `--ff-only`:** Git will automatically generate a new **Merge Commit** to stitch the two timelines back together. While the code merges successfully, this creates a "bubble" in the Git log. At an enterprise scale, this turns the history into an unreadable web, making troubleshooting and rollbacks highly difficult.
* **With `--ff-only`:** Git refuses to merge and throws an error (`fatal: Not possible to fast-forward, aborting.`). It acts as a safety brake, forcing you to resolve the divergence cleanly.

## 4. Fixing Diverged Timelines with `git rebase`
When a fast-forward is rejected, use `git rebase` to restore a straight line. 

Rebasing detaches your feature branch from its original starting point and reapplies your commits onto the very tip of the updated `main` branch. It rewrites your local history to look as if you started your work *after* your coworkers finished theirs.

**The Rebase Workflow:**
```bash
# 1. Ensure your feature branch is up to date with the new main
git switch feature/my-new-script
git rebase main

# 2. Switch back to main and perform the fast-forward merge
git switch main
git merge --ff-only feature/my-new-script
```

---

## 5. Lab Exercise Part 1: The Feature Branch Workflow

This is the standard start-to-finish workflow for adding a new feature, testing it in isolation, and cleanly integrating it.

### Step 1: Start from a clean baseline
Always ensure you are on `main` and your working directory is clean before branching.
```bash
git switch main
git status
```

### Step 2: Create the feature branch
Create a dedicated space for your work.
```bash
git switch -c feature/add-region
```

### Step 3: Implement and test the change
Open `scripts/Get-LabSummary.ps1` and replace its entire contents with this updated code to add the Region parameter:

```powershell
[CmdletBinding()]
param(
    [ValidateSet('dev', 'test', 'prod')]
    [string]$Environment = 'dev',

    [ValidateSet('canadacentral', 'canadaeast')]
    [string]$Region = 'canadacentral'
)

[pscustomobject]@{
    Project         = 'azure-infra-lab'
    Environment     = $Environment
    Region          = $Region
    DeploymentReady = $false
}
```

Save the file, then test the new logic locally:
```bash
./scripts/Get-LabSummary.ps1 -Environment test -Region canadaeast
```

### Step 4: Stage and commit your work
Save your verified changes to the feature branch's history.
```bash
git add scripts/Get-LabSummary.ps1
git commit -m "Add validated region parameter to lab summary"
```

### Step 5: Observe branch isolation
Prove the changes only exist on your feature branch by switching back to `main` and looking at your file in the editor. The new code will vanish, then return when you switch back.
```bash
git switch main
git switch feature/add-region
```

### Step 6: Review the changes
Compare your feature branch against `main` to ensure you are only introducing exactly what you intended.
```bash
git log --oneline main..feature/add-region
git diff main...feature/add-region
```

### Step 7: Fast-forward merge into main
Switch to your destination branch (`main`) and pull the feature branch in using a fast-forward merge.
```bash
git switch main
git merge --ff-only feature/add-region
```

---

## 6. Lab Exercise Part 2: Independent Documentation Branch

Now document the script without modifying `main` directly.

### Step 1: Create the documentation branch
```bash
git switch main
git switch -c docs/explain-script-usage
```

### Step 2: Update the README
Open `README.md` and append the following documentation to the bottom of the file:

```markdown
## Script Usage: Get-LabSummary.ps1

This script outputs a local summary object of our lab configuration. **Note: This script runs locally and does not connect to Azure or deploy any infrastructure.**

### Parameters
* **Environment**: Target environment. Allowed values: `dev`, `test`, `prod`. (Default: `dev`)
* **Region**: Target deployment region. Allowed values: `canadacentral`, `canadaeast`. (Default: `canadacentral`)

### Examples

**Example 1: Run with defaults**
./scripts/Get-LabSummary.ps1

**Example 2: Run with specific parameters**
./scripts/Get-LabSummary.ps1 -Environment prod -Region canadaeast
```

### Step 3: Stage, commit, and verify
Save the file, commit the changes, and verify your branch diverges cleanly from `main`.

```bash
git add README.md
git commit -m "Add usage documentation for Get-LabSummary.ps1"
git status
git log --oneline --decorate --graph --all -8
git diff main...docs/explain-script-usage
```