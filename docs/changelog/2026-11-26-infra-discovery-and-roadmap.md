# Infrastructure Discovery Scan & Improvement Roadmap

**Date:** 2026-11-26

## Objective

Document the current state of the Proxmox VE host and the 3-node Kubernetes (`v1.29.15`) cluster in read-only mode, enforce the Zero Leak Policy (`GEMINI.md`) across all collected telemetry, and establish a milestone-based platform engineering roadmap (`docs/architecture/improvement-roadmap.md`) alongside the `infra-discovery` skill.

## Changes Applied

- **Current Architecture Topology ([`docs/architecture/current-topology.md`](../architecture/current-topology.md)):**
  - Mapped the end-to-end traffic flow from Cloudflare Edge through `cloudflared` (Proxmox LXC `100`) and bridge `vmbr0` (`<INTERNAL_SUBNET>`) to Kubernetes nodes `k8s-master`, `k8s-worker-1`, and `k8s-worker-2` using a Mermaid flowchart.
  - Added sanitized inventory tables covering Proxmox VMs/LXCs (`VMID 100`, `200`, `201`, `202`, `9000`), storage (`local` and `local-lvm` on `476.9 GB` `sda`), host networking (`vmbr0`, `nic0`, `veth100i0`, `tap200i0`–`tap202i0`), Kubernetes nodes, and active workloads/services (`kube-prom-stack`, `pve-exporter`, `local-registry`, `portainer`, `omr-saas`).
  - Replaced all IPv4/IPv6 addresses, link-local identifiers, and pod/service CIDR IPs with standardized placeholders (`<PROXMOX_NODE_IP>`, `<K8S_MASTER_IP>`, `<K8S_WORKER_01_IP>`, `<K8S_WORKER_02_IP>`, `<INTERNAL_SUBNET>`, `<POD_CIDR>`).
- **Platform Improvement Roadmap ([`docs/architecture/improvement-roadmap.md`](../architecture/improvement-roadmap.md)):**
  - Analyzed architectural gaps across Container Registry security (`omr-api` `ImagePullBackOff`, HTTP-only `local-registry`), L7 Ingress & TLS (absence of Ingress Controller/`cert-manager`), Observability (missing Loki log aggregation and Alertmanager receivers), Storage durability (`local-path` `Delete` reclaim policy), and GitOps/IaC automation (ArgoCD, OpenTofu, Ansible, GitHub Actions).
  - Structured remediation into four sequential, learning-oriented milestones.
- **Agent Rules & Discovery Skill:**
  - Updated `GEMINI.md` with the Strict Anonymization & Sanitization Rules (Zero Leak Policy) and educational documentation requirement.
  - Added `.agents/skills/infra-discovery/SKILL.md` for repeatable read-only Proxmox and Kubernetes audits.

## Verification

- Executed read-only Kubernetes queries (`kubectl config current-context`, `kubectl get nodes -o wide`, `kubectl get ns`, `kubectl get pods -A -o wide`, `kubectl get svc,ing -A`, `kubectl get sc`) without modifying cluster state.
- Combined live Kubernetes state with sanitized Proxmox host output (`qm`, `pct`, storage, network, and disk layout).
- Verified Zero Leak Policy compliance across `docs/` via regex scan confirming zero IPv4/IPv6 addresses, MAC addresses, or secrets were written to disk.

## How It Works & Key Concepts

- **Read-Only Discovery Before Automation:** Performing non-destructive inspection (`kubectl get`, `pvesh`/`qm list`) establishes an accurate source-of-truth baseline before writing Terraform/OpenTofu or Kubernetes manifests, preventing configuration drift.
- **Zero Leak Policy in Public Portfolio Repos:** Replacing real internal IPs and EUI-64 IPv6 link-local addresses (which encode hardware MAC addresses) with standardized placeholders preserves architectural clarity for technical interviews without exposing home network topology.
