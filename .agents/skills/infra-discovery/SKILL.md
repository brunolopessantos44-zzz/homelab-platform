---
name: infra-discovery
description: Scans Proxmox VE and Kubernetes environments in read-only mode, sanitizes sensitive network data, generates architecture topology diagrams, and proposes a learning-oriented roadmap.
---

# Infrastructure Discovery & Topology Skill

## When to Trigger
Trigger when the user asks to scan, audit, map, or discover the current state of the Proxmox VE host and/or Kubernetes cluster.

## Safety & Read-Only Constraint
- Execute **ONLY read-only** inspection commands (e.g., `kubectl get`, `kubectl describe`, `helm list`, `pvesh get`, `qm list`, `pct list`, `lsblk`, `ip -br a`).
- **NEVER** modify cluster state, restart services, or alter configurations during discovery.

## Execution Workflow
1. **Cluster & Hypervisor Inspection:**
   - Check Kubernetes nodes, namespaces, pods, services, ingress resources, storage classes, and Helm releases (`kubectl get nodes -o wide`, `kubectl get ns`, `kubectl get pods -A`, `kubectl get svc,ing -A`, `helm list -A`).
   - Check Proxmox VMs, LXC containers, storage pools, and network bridges (via local commands or SSH if configured by the user).
2. **Data Sanitization (CRITICAL):**
   - Strip all real IP addresses, serial numbers, tokens, and internal domains from the collected output before writing to disk. Replace them with `<PLACEHOLDER>` tags defined in `GEMINI.md`.
3. **Generate Topology & Learning Guide:**
   - Create/update `docs/architecture/current-topology.md` containing:
     - **Mermaid Architecture Diagram** showing traffic flow from Edge/Cloudflare -> Proxmox -> VMs -> K8s Ingress -> Workloads.
     - **Inventory Table** (sanitized) of VMs, K8s nodes, and running services.
     - **Educational Breakdown:** Clear explanation in English of how the components interact.
4. **Generate Improvement Roadmap:**
   - Create `docs/architecture/improvement-roadmap.md` analyzing gaps in:
     - Container Registry setup
     - Observability (Prometheus, Grafana, Loki)
     - GitOps & CI/CD Pipelines (ArgoCD, GitHub Actions/GitLab CI)
   - Break each improvement into small, educational milestones.