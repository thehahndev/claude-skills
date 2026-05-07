# My Claude Skills

My Claude Skills for automating repetitive tasks and personal AI workflows.

## Overview
This repository contains a collection of custom skills for **Claude Desktop** and **Claude Code**. By versioning these skills, I can track workflow improvements, roll back changes, and keep my AI automation in sync across different machines.

## Setup & Installation

To use these skills, you must link this repository to your local Claude configuration directory. Using a **Symbolic Link (Symlink)** is the recommended method because it allows Claude to "see" your repo updates instantly without needing to manually copy files.

### 1. Clone the Repository
Clone this repo to your preferred projects or documents folder:
```powershell
git clone https://github.com "C:\Users\YourName\Documents\Claude-Skills"
```

### 2. Link to Claude
1. **Backup/Move** any existing skills currently in `%userprofile%\.claude\skills`.
2. **Delete** the existing `skills` folder (Claude will use the link instead):
   ```powershell
   Remove-Item -Recurse -Force "$env:USERPROFILE\.claude\skills"
   ```
3. **Run PowerShell as Administrator** and create the link:
   ```powershell
   # IMPORTANT: Replace the Target path with your actual repo location
   New-Item -ItemType SymbolicLink -Path "$env:USERPROFILE\.claude\skills" -Target "C:\Users\YourName\Documents\Claude-Skills"
   ```

### 3. Verify Installation
Restart Claude Desktop. You can verify the skills are loaded by asking:
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
