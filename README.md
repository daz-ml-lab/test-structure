# Project Workflow & Onboarding Guide

Welcome to the **Project** repository! This guide explains how to set up your environment, how to work in your personal sandbox using your machine’s hostname, and how code is eventually promoted to the shared library (`library/`) and official notebooks (`workflows/`). Automated commits ensure your sandbox work is backed up hourly, but you can still manually commit and push anytime.

## Table of Contents
1. [Overview](#overview)
2. [Repository Structure](#repository-structure)
3. [Requirements](#requirements)
4. [Initial Setup](#initial-setup)
5. [Working in Your Sandbox](#working-in-your-sandbox)
6. [Automated Commits (Hourly)](#automated-commits-hourly)
7. [Manual Commits](#manual-commits)
8. [Promoting Code to `library/` and `workflows/`](#promoting-code-to-library-and-workflows)
9. [Troubleshooting & Tips](#troubleshooting--tips)
10. [Further Reading](#further-reading)

---

## Overview

1. **Single Repository, Single Branch**  
   - We use one repository for everything. Most day-to-day work happens in the `sandbox/` folder, automatically committed every hour if there are changes.

2. **Per-User Sandbox Folders**  
   - Each user is identified by their **hostname** (the name of their compute instance).  
   - Your sandbox folder is `sandbox/<hostname>/`. You can create and modify notebooks and code in there without affecting anyone else’s work.

3. **Conda Environment**  
   - The project uses a Conda environment defined by `environment.yml`. Each user sets up their environment once to ensure consistent Python/package versions.

4. **Promotion to Shared Library**  
   - If your sandbox work is worth sharing more broadly, we do a manual “promotion” to `library/` or to a notebook in `workflows/`. This ensures only stable or validated code enters the shared library.

---

## Repository Structure (example)

A simplified view:

```
my_project/
├─ README.md               # This guide
├─ environment.yml         # Conda environment definition
├─ setup_and_cron.sh       # Script to set up your sandbox & auto-commit cron job
├─ library/                # Shared library code
│   ├─ data_loading.py
│   └─ processing.py
├─ workflows/              # "Official" or final notebooks
│   └─ final_use_cases.ipynb
├─ sandbox/                # Everyone’s personal subfolders for experiments
│   ├─ alice-machine/
│   ├─ bob-instance/
│   └─ ...
└─ tests/                  # Optional tests for shared code
    └─ test_processing.py
```

- **`library/`**: Python modules/functions used by multiple notebooks.  
- **`workflows/`**: Final or reference notebooks for proven workflows.  
- **`sandbox/`**: Each user’s personal space (named after the machine’s hostname).  
- **`environment.yml`**: Used to create/update a Conda environment consistently across all machines.

---

## Requirements

- **Git**: Installed and configured (with SSH keys or a credential manager for seamless pushes).
- **Conda** (Miniconda or Anaconda): For managing Python environments via `environment.yml`.
- **Cron** or equivalent scheduler: Used to run automated commits hourly (Linux/macOS).  
- **Access to the Repository**: Must have permissions to clone and push to the main repo.

---

## Initial Setup

Follow these steps **once** when you first get access to the project:

1. **Clone the Repository** (optional if already cloned) 
   - Choose a location on your machine where you want the project code, then:
     ```bash
     git clone https://github.com/yourorg/my_project.git
     cd my_project
     ```
     
2. **Run `setup_and_cron.sh`**  
   - This script will:
     1. Detect your hostname, e.g., `alice-machine`.
     2. Create (or verify) `sandbox/alice-machine/`.
     3. Create or update your Conda environment using `environment.yml`.
     4. Generate a small auto-commit script that only commits your sandbox folder.
     5. Install a cron job to run the auto-commit script hourly.  
   - Run:
     ```bash
     bash setup_and_cron.sh
     ```
   - **Note**: You may need to grant execution permissions (`chmod +x setup_and_cron.sh`) beforehand.
   - The script will output a success message if everything goes smoothly.

3. **Activate Conda Environment**  
   - After the environment is created, you can:
     ```bash
     conda activate <project_name>_env
     ```
   - This ensures you’re using the correct versions of Python and libraries.

---

## Working in Your Sandbox

1. **Navigate to Your Folder**  
   - After setup, a folder named `sandbox/<hostname>/` should exist.  
   - Put notebooks, experiments, or scratch scripts in that folder. Example:
     ```
     sandbox/alice-machine/
       └─ experiment_data_cleaning.ipynb
     ```

2. **Importing Shared Code**  
   - If you want to use the common library code from `library/`, add something like:
     ```python
     import sys, os
     
     PROJECT_ROOT = os.path.dirname(os.path.dirname(os.path.abspath('__file__')))
     sys.path.append(PROJECT_ROOT)
     
     from library.processing import clean_data
     ```
     This approach modifies your Python path so the `library/` folder is recognized.

3. **Environment Consistency**  
   - Always remember to `conda activate <project_name>_env` (or whatever the environment name is) before running your code.

---

## Automated Commits (Hourly)

1. **Cron Job**  
   - The setup script added a cron job that runs `auto_commit.sh` once every hour.  
   - This script:
     - Pulls the latest changes from the repository (`git pull --rebase`).
     - Checks if there are changes in **only** your sandbox folder.
     - If there are changes, it commits and pushes them with a timestamp message.

2. **Benefits**  
   - You don’t have to remember to commit your local sandbox changes; they’re regularly saved in Git.  
   - Minimizes risk of losing your work.

3. **Potential Conflicts**  
   - If the script encounters a merge conflict (e.g., if someone edited your same files remotely), it will stop and ask for manual conflict resolution.

---

## Manual Commits

You can always push changes **immediately** if you want:

1. **Stage & Commit**  
   ```bash
   cd /path/to/my_project
   git add sandbox/<hostname>/
   git commit -m "Some descriptive message about your changes"
   ```
2. **Push**  
   ```bash
   git push
   ```

This can be helpful if you need to share something right away, rather than waiting for the hourly cron job.

---

## Promoting Code to `library/` and `workflows/`

1. **Identify Useful Code**  
   - If you’ve developed something in your sandbox that would benefit the whole team (e.g., a new data-cleaning function), the team decides whether to promote it.

2. **Move or Copy**  
   - **Refactor** the relevant code from your sandbox into a Python module in `library/`.
   - If it’s a **finished workflow** or official example, copy your final notebook from `sandbox/<hostname>/` to `workflows/`.

3. **Manual Commit & Push**  
   - Because these changes affect shared code, do a normal:
     ```bash
     git add library/ workflows/
     git commit -m "Add new library function and official notebook"
     git push
     ```
4. **Notify the Team**  
   - Let everyone know they can now use the newly added library code or check out the new official notebook.

---

## Troubleshooting & Tips

1. **Merge Conflicts**  
   - If the auto-commit script fails with a conflict, run:
     ```bash
     git pull --rebase
     ```
     Resolve conflicts in your text editor, then commit and push again.

2. **Conda Environment Issues**  
   - If the environment fails to create, you might try an update:
     ```bash
     conda env update -f environment.yml -n <project_name>_env
     ```
   - Make sure you’re on a recent version of conda or try `mamba` if standard conda is slow.

3. **Adjusting Cron Frequency**  
   - By default, we commit hourly. If you prefer a different schedule, edit the crontab entry in `setup_and_cron.sh` or via `crontab -e`.

4. **Git Credentials**  
   - Ensure you’ve configured Git to push without requiring a password each time (SSH keys or credential caching).

5. **Manually Overriding**  
   - If you want to skip the auto-commit for any reason, you can temporarily remove the cron job or comment it out with `crontab -e`.

---

## Further Reading

- **Git Basics**: [Git documentation](https://git-scm.com/doc)  
- **Conda**: [Conda documentation](https://docs.conda.io/en/latest/)  
- **Project Wiki** (if you have one internally for advanced topics)

---

### Thank You!

If you have any questions, please reach out to the project maintainers. Enjoy exploring the project in your personal sandbox, and happy coding!
