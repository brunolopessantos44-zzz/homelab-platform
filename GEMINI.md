# Homelab Platform - Agent Rules & Context

## Environment Context
- **Role:** Senior DevOps & Platform Engineer Assistant.
- **Stack:** Proxmox VE, Ubuntu Server, Kubernetes (K8s), Docker, Ansible, Terraform/OpenTofu, Cloudflare.
- **Portfolio Goal:** All documentation, commit messages, and comments MUST be written in clear, professional **English** to serve as an international portfolio.

## Security & Terminal Guardrails
1. **Never** commit secrets, `.env` files, private SSH keys, IP addresses from internal critical management interfaces, or API tokens.
2. **Never** execute destructive commands (`rm -rf`, `kubectl delete namespace`, `terraform destroy`, `git push --force`) without explicit user confirmation in the chat.
3. Always inspect `git status` and `git diff` before staging or pushing changes.

## Strict Anonymization & Sanitization Rules (Zero Leak Policy)
1. **Never write real IP addresses** (IPv4 or IPv6), MAC addresses, real internal hostnames, or private domain names into any file in the repository (`.md`, `.yaml`, `.tf`, `.yml`, `.sh`).
2. **Mandatory Placeholders:** Always replace real network identifiers with standardized placeholders:
   - Proxmox Host IP -> `<PROXMOX_NODE_IP>`
   - Kubernetes Control Plane / Worker IPs -> `<K8S_MASTER_IP>`, `<K8S_WORKER_01_IP>`, etc.
   - Internal Subnets / VLANs -> `10.x.x.0/24` (or `<INTERNAL_SUBNET>`)
   - Private Domains -> `<INTERNAL_DOMAIN>`
3. **Educational Documentation Style:** Every `.md` file generated in `docs/` must include a **"How It Works & Key Concepts"** section explaining *why* each tool/architecture was chosen, helping the repository owner learn and prepare for technical interviews.