# Homelab Platform & Cloud-Native Infrastructure

Automated home laboratory environment designed to simulate enterprise-grade Platform Engineering, Infrastructure as Code (IaC), GitOps workflows, and AI-assisted DevOps operations.

## Architecture Overview

- **Hypervisor & Compute:** Proxmox VE virtualization layer hosting isolated Linux nodes (Ubuntu Server).
- **Container Orchestration:** Multi-node Kubernetes cluster running core platform services and containerized workloads.
- **Container Registry & CI/CD:** Self-hosted container registry integrated with automated build and deployment pipelines.
- **Networking & Edge Security:** DNS management, reverse proxy/Ingress routing, and SSL/TLS termination via Cloudflare.
- **AI-Assisted Operations:** Custom agentic workflows built with Google Antigravity (`.agents/skills/`) enforcing automated Markdown documentation logging and pre-commit security checks.

## Repository Structure

```text
homelab-platform/
├── .agents/skills/       # Custom AI agent skills for automated docs and Git workflows
├── docs/
│   ├── architecture/     # Network diagrams, cluster topology, and ADRs
│   └── changelog/        # Automated chronological log of infrastructure changes
├── infrastructure/       # Proxmox provisioning, Ansible playbooks, and Terraform/OpenTofu
└── kubernetes/           # Kubernetes manifests, Helm values, and GitOps configs