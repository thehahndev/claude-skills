# My Claude Skills

Custom skills for automating repetitive tasks and personal AI workflows in **Claude Desktop** and **Claude Code**.

## Overview

Claude supports **skills** — bundles of instructions that extend what Claude can do. This repository stores those skills as files so they can be version-controlled, shared across machines, and rolled back if something breaks.

Skills live in the `skills` subfolder of Claude's configuration directory:

| Platform | Path |
|----------|------|
| Windows | `C:\Users\<windows-username>\.claude\skills\` |
| macOS | `~/.claude/skills/` |
| Linux | `~/.claude/skills/` |

Claude reads skills automatically from that location.

## Setup & Installation

There are two ways to install these skills depending on whether you use Git:

---

### Option A — Direct Download (no Git required)

1. Download the repository as a ZIP file. On the GitHub repository page, click the green **Code** button, then choose **Download ZIP**.
2. Extract the ZIP. Inside you will find a folder with all the skill subfolders — for example `skill-name-1/`, `skill-name-2/`, etc.
3. Copy those skill subfolders into your Claude skills directory. If the `skills` folder does not exist yet, create it first.

   **Windows:**
   ```
   C:\Users\<windows-username>\.claude\skills\
   ```

   **macOS / Linux:**
   ```
   ~/.claude/skills/
   ```

   After copying, the structure should look like this (using the Windows path as an example):
   ```
   C:\Users\<windows-username>\.claude\skills\
   ├── skill-name-1\
   │   └── SKILL.md
   └── skill-name-2\
       └── SKILL.md
   ```

4. Restart Claude Desktop. Skills are now active.

> **To update later**, repeat from step 1 — download the ZIP again and overwrite the folders.

---

### Option B — Git Clone + Symbolic Link (recommended for syncing across machines)

This method keeps your skills folder pointing directly at your local Git repository. Any time you `git pull` to fetch updates, Claude sees the changes immediately — no copying needed.

#### 1. Get your repository's clone URL

On the GitHub repository page, click the green **Code** button and copy the URL shown under **HTTPS**. It will look like:
```
https://github.com/<github-username>/<repo-name>.git
```

#### 2. Clone the repository

**Windows** — open PowerShell:
```powershell
# Replace the URL with the one you copied from GitHub
# Replace the destination path with wherever you keep your projects
git clone https://github.com/<github-username>/<repo-name>.git "C:\Users\<windows-username>\Documents\Claude-Skills"
```

**macOS / Linux** — open Terminal:
```bash
# Replace the URL with the one you copied from GitHub
# Replace the destination path with wherever you keep your projects
git clone https://github.com/<github-username>/<repo-name>.git ~/Documents/Claude-Skills
```

#### 3. Create a symbolic link

A symbolic link makes the Claude skills directory point to your cloned repo folder instead of being its own separate folder.

**Windows** — first remove the existing `skills` folder if one exists (back up anything inside it first):
```powershell
Remove-Item -Recurse -Force "$env:USERPROFILE\.claude\skills"
```

Then **open a new PowerShell window as Administrator** (right-click the Start button → "Windows PowerShell (Admin)" or "Terminal (Admin)") and run:
```powershell
# Replace the -Target path with the actual path where you cloned the repo in step 2
New-Item -ItemType SymbolicLink `
  -Path "$env:USERPROFILE\.claude\skills" `
  -Target "C:\Users\<windows-username>\Documents\Claude-Skills"
```
> Administrator rights are required because Windows restricts who can create symbolic links.

**macOS / Linux** — open Terminal:
```bash
# Remove the existing skills folder if one exists (back up anything inside it first)
rm -rf ~/.claude/skills

# Replace the source path with the actual path where you cloned the repo in step 2
ln -s ~/Documents/Claude-Skills ~/.claude/skills
```

#### 4. Verify installation

Restart Claude Desktop. You can confirm the skills loaded by asking:
> "List your available skills and their purpose."

## Repository Structure
Each skill is organized into its own folder containing a `SKILL.md` file:
```text
.
├── skill-name-1/
│   └── SKILL.md       # The core logic and instructions
├── skill-name-2/
│   ├── SKILL.md
│   └── scripts/       # Any helper scripts the skill calls
└── .gitignore
```

## Security & Privacy
- **Secrets:** Never commit API keys, passwords, or personal credentials.
- **Local Paths:** Use relative paths or environment variables within your skills to ensure they work across different machines.
