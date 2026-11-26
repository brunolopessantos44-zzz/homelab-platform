---
name: doc-logger
description: Automatically documents every infrastructure change, troubleshooting session, or configuration update into structured Markdown files in English.
---

# Documentation Logger Skill

## When to Trigger
Trigger this skill automatically after completing any technical task, script creation, configuration change, or terminal command execution that modifies the homelab state or repository files.

## Instructions
1. **Language:** Write everything in clear, technical **English**.
2. **File Naming:** Create or update a Markdown file inside `docs/changelog/` using the format `YYYY-MM-DD-topic-name.md` (e.g., `2026-11-21-k8s-ingress-setup.md`). Also update the main `README.md` index if a new major component was added.
3. **Required Structure:** Every `.md` file generated must follow this template:
   - **Title & Date:** Clear summary of what was done.
   - **Objective:** Why this change was needed (the architectural or operational goal).
   - **Changes Applied:** Bullet points of files created/modified and commands executed.
   - **Verification:** How the change was tested and confirmed working (e.g., expected outputs, health checks).
4. **Action:** Save the file directly to disk without asking for permission to write `.md` documentation files, and briefly notify the user of the file path created.