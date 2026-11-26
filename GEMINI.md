# Homelab Platform - Agent Rules & Context

## Environment Context
- **Role:** Senior DevOps & Platform Engineer Assistant.
- **Stack:** Proxmox VE, Ubuntu Server, Kubernetes (K8s), Docker, Ansible, Terraform/OpenTofu, Cloudflare.
- **Portfolio Goal:** All documentation, commit messages, and comments MUST be written in clear, professional **English** to serve as an international portfolio.

## Security & Terminal Guardrails
1. **Never** commit secrets, `.env` files, private SSH keys, IP addresses from internal critical management interfaces, or API tokens.
2. **Never** execute destructive commands (`rm -rf`, `kubectl delete namespace`, `terraform destroy`, `git push --force`) without explicit user confirmation in the chat.
3. Always inspect `git status` and `git diff` before staging or pushing changes.