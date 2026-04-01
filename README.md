# Ansible Onboarding Workstation

## Overview

This setup creates a production-ready Ansible development environment using an isolated Python virtual environment, VS Code tooling, SSH configuration, Git setup, and pre-commit hooks.

The goal is to ensure consistency, avoid system-level pollution, and eliminate “works on my machine” issues.

---

## Environment

- OS: WSL2 Ubuntu
- Python: `python3`
- Virtual Environment: `.venv`
- Workspace: `ansible-onboarding/`

---

## 1. Environment & Ansible Installation

I created a dedicated workspace and installed Ansible inside an isolated Python environment.

```bash
mkdir -p ~/ansible-onboarding && cd ~/ansible-onboarding
python3 -m venv .venv && source .venv/bin/activate
python -m pip install --upgrade pip
python -m pip install ansible ansible-lint yamllint
ansible --version
ansible-lint --version
```

I saved the dependencies for reproducibility:

```bash
pip freeze > requirements.txt

```
## 2. VS Code Setup
I installed the following extensions:

Red Hat Ansible
YAML (Red Hat)
Python
EditorConfig

I configured VS Code to use the project’s virtual environment and enable linting.

.vscode/settings.json
```json
{
  "python.defaultInterpreterPath": "${workspaceFolder}/.venv/bin/python",
  "ansible.python.interpreterPath": "${workspaceFolder}/.venv/bin/python",
  "ansibleLint.enabled": true,
  "yaml.validate": true,
  "files.trimTrailingWhitespace": true,
  "editor.formatOnSave": true
}
```

I added a shared formatting configuration using .editorconfig.

.editorconfig (opinionated but common)

```ini

root = true
[*]
charset = utf-8
end_of_line = lf
indent_style = space
indent_size = 2
insert_final_newline = true
trim_trailing_whitespace = true

```

I created a baseline ansible.cfg with team-friendly defaults.

ansible.cfg

```ini
[defaults]
inventory            = inventories/     # placeholder folder (used later)
roles_path           = roles:./.ansible/roles
host_key_checking    = True
retry_files_enabled  = False
interpreter_python   = auto_silent
forks                = 10
timeout              = 30
stdout_callback      = yaml
bin_ansible_callbacks = True

[ssh_connection]
pipelining = True
ssh_args   = -o ControlMaster=auto -o ControlPersist=60s

```

## 3. SSH Setup

I generated a new SSH key and configured the SSH agent.

```bash
ssh-keygen -t ed25519 -C "you@company" -N "" -f ~/.ssh/id_ed25519
eval "$(ssh-agent -s)" && ssh-add ~/.ssh/id_ed25519
```

I created an SSH config file with safe defaults:

```ini
Host *
  ServerAliveInterval 30
  ServerAliveCountMax 4
  AddKeysToAgent yes
  IdentityFile ~/.ssh/id_ed25519
  StrictHostKeyChecking ask
```
## 4. Git Setup

I initialized a Git repository and created a .gitignore file.

```bash
git config --global user.name  "Your Name"
git config --global user.email "your.email@company.com"
git config --global init.defaultBranch main
git init
```

## 5. Pre-commit Hooks

I installed pre-commit and added linting hooks for YAML and Ansible.

```bash
python -m pip install pre-commit
```

I set up standard hooks in .pre-commit-config.yaml:

```yaml
python -m pip install pre-commit
cat > .pre-commit-config.yaml <<'YAML'
repos:
  - repo: https://github.com/adrienverge/yamllint
    rev: v1.35.1
    hooks:
      - id: yamllint
  - repo: https://github.com/ansible/ansible-lint
    rev: v24.6.1
    hooks:
      - id: ansible-lint
YAML
```

I installed the pre-commit hooks:

```bash
pre-commit install
```

## 6. How to Use the Environment

Activate the virtual environment:

```bash
source .venv/bin/activate
```
