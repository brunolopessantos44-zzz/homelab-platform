---
name: github-publisher
description: Automates staging, security scanning, committing, and pushing homelab code and documentation to GitHub via terminal.
---

# GitHub Publisher Skill

## When to Trigger
Trigger when the user asks to "push to GitHub", "save to repo", "publish changes", or finishes a milestone and wants to sync with remote.

## Execution Workflow (Terminal)
1. **Security Pre-Check:**
   - Run `git status` and `git diff` in the terminal.
   - Scan the diff for hardcoded passwords, tokens, bearer keys, or sensitive internal data.
   - **STOP IMMEDIATELY** if any secret is detected, warn the user, and suggest moving the value to `.gitignore` or a secret manager (e.g., Sealed Secrets / Vault).
2. **Ensure Documentation:**
   - Verify if the `doc-logger` skill has already documented the current changes in `.md` format. If not, generate the `.md` summary before committing.
3. **Stage & Commit:**
   - Run `git add .` (respecting `.gitignore`).
   - Generate a concise commit message in **English** following **Conventional Commits** (e.g., `feat(k8s): configure traefik ingress controller`, `docs(homelab): add proxmox cluster architecture`).
   - Execute `git commit -m "<message>"`.
4. **Push:**
   - Run `git push origin main` (or the current active branch).
   - Confirm the remote URL and commit hash in the final response.