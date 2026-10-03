# Lesson 1 — Know exactly what you are committing

Create your first local training repository, run a small PowerShell script, and understand the difference between saving, staging, and committing.

**Time:** approximately 60 minutes.  
**Cost:** none. No Azure connection or deployment is required.  
**Tools:** Git, VS Code, and PowerShell.  
**Shell:** PowerShell; the examples use your home directory.

This lesson uses `azure-infra-lab` as the local project name. Your GitHub repository may be named `cloud-devops`; the local folder and remote repository do not need matching names. Connecting to GitHub comes later.

If you already completed this lesson, keep this file as a reference. Do not overwrite your later script or recreate your existing repository to repeat it; use a separate practice folder if needed.

## 1. Practical outcome

By the end, you should be able to:

- Create and open a local repository in VS Code.
- Identify the project root and terminal's current directory.
- Add a README and a small PowerShell script.
- Review and make your first commit.
- Explain why saving, staging, and committing are separate operations.
- Demonstrate how a file can contain both staged and unstaged changes.

**Prerequisites:** Git, VS Code, and PowerShell are installed. You do not need a GitHub account or an Azure subscription for this lesson.

## 2. The concept and why it matters

### Git versus GitHub versus VS Code

| Component | Responsibility |
| --- | --- |
| VS Code | Provides an editor, integrated terminal, and tools for inspecting changes. |
| Git | Records version history and manages branches locally. |
| GitHub | Hosts shared repositories and provides collaboration features such as pull requests and reviews. |

VS Code's Git interface uses your installed Git. Its Source Control view and the terminal work with the same repository.

A **repository** is a project whose history Git manages. The **project root** is its top-level folder. Git stores its internal information in a hidden `.git` directory inside that folder for this lab.

### The three places to understand

| Place | Meaning | Infrastructure example |
| --- | --- | --- |
| Working directory | The project files you are editing on disk. | You change a script's default environment. |
| Staging area or index | The content selected for the next commit. | You select the reviewed script change. |
| Local history | Snapshots recorded by commits. | You record the change with an explanation. |

**Staging captures a file's content at that moment.** Editing and saving it again does not automatically update the staged version.

A **commit** records the staged snapshot with metadata such as an author and message. It does not upload your work to GitHub or deploy infrastructure.

In professional work, staging helps you select a coherent change even when your working directory contains other edits. Inspecting the staged diff shows exactly what you are about to record.

## 3. Worked example

Run one block at a time and read the output.

### A. Inspect your tools — about 5 minutes

Open VS Code, choose **Terminal → New Terminal**, and make sure the terminal is PowerShell.

```powershell
$PSVersionTable.PSVersion
git --version
code --version
Get-Location
```

`Get-Location` shows the terminal's current directory. Always know which directory you are in before running project commands.

### B. Create the project — about 5 minutes

Run this block only when creating a new practice project:

```powershell
$projectPath = Join-Path $HOME "source/personal/azure-infra-lab"

if (Test-Path $projectPath) {
    throw "This folder already exists. Stop and choose a new practice folder."
}

New-Item -ItemType Directory -Path $projectPath -Force | Out-Null
Set-Location $projectPath

git init -b main
code .
```

`git init` creates Git's internal repository data. `-b main` names the initial branch. `code .` opens the current folder in VS Code.

In that VS Code window, open a PowerShell terminal and run:

```powershell
Get-Location
git status
```

Expected state: you are in the project folder, the branch is `main`, and no commits exist yet.

Opening the project root gives the editor a useful boundary for files and tools. A multi-folder workspace is unnecessary for this first lab.

### C. Set the identity for this repository — about 5 minutes

Because you use both personal and work accounts, configure a repository-local identity for this exercise.

Replace both example values before running:

```powershell
git config --local user.name "Your Name"
git config --local user.email "YOUR_PERSONAL_COMMIT_EMAIL"

git config --local --get user.name
git config --local --get user.email
```

Use an email associated with your personal GitHub account, or copy your GitHub-provided `noreply` address from your account's email settings.

`--local` applies these settings to this repository. It does not change the identity configured for other repositories.

**Commit identity and authentication are different:** these values label your commits; they do not sign you into GitHub or choose the account used for a push.

### D. Create two files — about 10 minutes

In VS Code Explorer, create `README.md` at the project root. Copy the following content into it:

````markdown
# Azure Infrastructure Lab

A personal learning repository for Azure infrastructure automation.

## Current scope

- Practise Git and GitHub workflows.
- Maintain PowerShell scripts.
- Introduce Terraform and automated validation progressively.

## Run the local example

Open PowerShell at the repository root:

```powershell
./scripts/Get-LabSummary.ps1 -Environment dev
```

## Deployment status

This repository does not currently deploy Azure resources.
````

Create a folder named `scripts`. Inside it, create `Get-LabSummary.ps1` with this complete content:

```powershell
[CmdletBinding()]
param(
    [ValidateSet('dev', 'test', 'prod')]
    [string]$Environment = 'dev'
)

[pscustomobject]@{
    Project         = 'azure-infra-lab'
    Environment     = $Environment
    DeploymentReady = $false
}
```

Save both files. Run the script from the repository root:

```powershell
./scripts/Get-LabSummary.ps1 -Environment dev
```

Expected values:

| Property | Value |
| --- | --- |
| Project | `azure-infra-lab` |
| Environment | `dev` |
| DeploymentReady | `False` |

The script returns a local information object. `ValidateSet` restricts the environment parameter to the three listed values. The default is `dev` if you omit the parameter.

The environment is only a label; even choosing `prod` connects to nothing. `DeploymentReady` is a fixed value, not the result of a readiness assessment.

### E. Inspect, stage, and commit — about 10 minutes

```powershell
git status
git diff
```

You should see untracked files in `git status`. An empty `git diff` is expected: ordinary diff does not display the contents of new, untracked files.

Stage the two files explicitly:

```powershell
git add README.md scripts/Get-LabSummary.ps1
git diff --staged
```

Read the staged content, then commit:

```powershell
git commit -m "Add lab overview and local summary script"

git status
git log --oneline -3
```

Expected state: one commit exists, and Git reports a clean working directory if no additional edits were made.

| Command | What it shows |
| --- | --- |
| `git status` | Current branch and the state of tracked and untracked files. |
| `git diff` | Differences between tracked working files and the staging area. |
| `git diff --staged` | Differences between the staging area and the last commit; before the first commit, staged additions. |
| `git log --oneline -3` | Up to three recent commits in a compact format. |

## 4. What happens behind the scenes

| Operation | Working directory | Staging area | Local history |
| --- | --- | --- | --- |
| Create and save files | New files exist on disk. | Files are not staged yet. | No new commit. |
| Run `git add` | Files remain on disk. | Their current content is selected. | No new commit. |
| Run `git commit` | Files remain on disk. | The selected snapshot matches the new commit. | A new snapshot is recorded. |
| Edit and save again | File content changes. | The previously staged content stays unchanged until staged again. | Existing commits remain unchanged. |

Git also records commit metadata, including the author and message. Everything remains on your laptop during this lesson.

The professional habit is to **inspect the exact staged change before committing**. A useful commit contains a coherent change that you can explain.

Using `main`, `scripts`, and `source/personal` are course conventions. Reviewing what you commit is a broadly applicable practice.

## 5. Your hands-on exercise — about 15 minutes

Complete this without making another commit yet:

1. Add a `## Learning notes` section to the README with one sentence explaining what Git does.
2. Save the file.
3. Run `git diff`, then stage **only the README**.
4. Add a second sentence explaining what GitHub does.
5. Save again, but **do not stage again**.
6. Run these commands:

```powershell
git status --short
git diff
git diff --staged
```

Open VS Code's **Source Control** view. Look for the README under both staged and unstaged changes, and inspect each diff.

**Pause here.** Explain which sentence a commit would contain if you committed now. Use the two diffs as evidence rather than guessing from what the editor displays.

Leave these exercise changes uncommitted for review. Lesson 2 starts by inspecting and finishing them before creating a feature branch.

## 6. Common mistakes and troubleshooting

| Symptom | What to check |
| --- | --- |
| `git` is not recognized | Restart VS Code after installation. If it persists, share the error so the installation and PATH can be checked. |
| `code` is not recognized | Use **File → Open Folder** and select the project folder. |
| `Not a git repository` | Run `Get-Location` and navigate to the project root. |
| The practice folder already exists | Open the existing project or choose a separate practice folder. Do not overwrite completed work. |
| `Author identity unknown` | Set the two repository-local identity values from section C. |
| Script execution is blocked | Share the error text so the applicable execution policy can be inspected before changing settings. |
| A saved edit is missing from the staged diff | Saving and staging are separate. Inspect `git diff` for the unstaged edit. |
| `git diff` is empty for a new file | Check `git status`. Untracked content is not included in ordinary diff. |
| A diff fills the terminal and appears stuck | If Git opened a pager, press `q` to exit. |

## 7. Knowledge check

Answer in your own words:

1. Can you create commits without a GitHub account or internet connection?
2. After staging a file and then editing it again, which content would a commit record?
3. Does setting `user.email` select the GitHub account used to push?
4. Why might you intentionally leave part of your current work out of a commit?

## 8. Completion checklist and official references

- [ ] I can identify my project's root folder.
- [ ] I checked my Git and PowerShell versions.
- [ ] I configured the commit identity for this repository.
- [ ] The script runs locally and returns the expected values.
- [ ] I created one commit after reviewing its staged content.
- [ ] I can explain working files, staged content, and local history.
- [ ] I completed the exercise without committing its changes yet.
- [ ] I answered the knowledge-check questions.

### Official references

- [Git init](https://git-scm.com/docs/git-init)
- [Git add](https://git-scm.com/docs/git-add)
- [Git diff](https://git-scm.com/docs/git-diff)
- [VS Code Source Control](https://code.visualstudio.com/docs/sourcecontrol/overview)
- [GitHub commit email](https://docs.github.com/en/account-and-profile/how-tos/email-preferences/setting-your-commit-email-address)

### Submit for review

Send your operating system, Git and PowerShell versions, knowledge-check answers, and the output of:

```powershell
git status --short
git diff
git diff --staged
```

You may redact personal paths. Do not share passwords, access tokens, private keys, or confidential employer content. There is no need to share your email address.

**Next lesson:** create a feature branch, add a region parameter to the script, review the change, and merge it into `main`.
