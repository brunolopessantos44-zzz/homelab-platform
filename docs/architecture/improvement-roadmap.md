# Infrastructure Improvement Roadmap

> **Last Updated:** 2026-11-26  
> **Baseline:** Single-node Proxmox VE host running 1 LXC (`cloudflared`) and a 3-node Kubernetes `v1.29.15` cluster (`k8s-master`, `k8s-worker-1`, `k8s-worker-2`).

This roadmap translates the current architecture audit into focused, learning-oriented milestones across four key platform engineering domains: **Container Registry**, **Ingress & Networking**, **Observability**, and **GitOps & CI/CD**.

---

## Summary of Observed Gaps

| Area | Current State | Identified Gap / Risk |
| :--- | :--- | :--- |
| **Container Registry** | In-cluster `local-registry` exposed on HTTP `NodePort 30500` with `local-path` storage | No TLS, no authentication/RBAC, no image vulnerability scanning, single-node disk coupling; `omr-api` in `omr-saas` is stuck in `ImagePullBackOff` |
| **Ingress & Routing** | Services exposed via `NodePort` (`30500`, `30777`, `32000`); `nginx` namespace is empty | No centralized L7 Ingress Controller or automatic TLS (`cert-manager`), forcing `cloudflared` to target static node ports |
| **Observability** | `kube-prom-stack` (Prometheus, Grafana, Alertmanager) + `pve-exporter` deployed | Missing centralized log aggregation (Loki), no distributed tracing, ephemeral dashboard changes, and no automated alert routing |
| **GitOps & CI/CD** | Manual `kubectl` deployments; `ubuntu-2204-template` (VMID `9000`) exists on Proxmox | No declarative GitOps reconciliation (ArgoCD), no CI build/push pipelines, and no IaC automation (OpenTofu/Ansible) managing VMs |
| **Storage & Resilience** | `local-path` StorageClass (`Delete` reclaim policy); `local-lvm` on single SSD (`476.9 GB`) | Volumes are pinned to individual worker nodes; `Delete` reclaim policy risks data loss on PVC deletion; no off-host backups |

---

## Milestone 1: Stabilize Container Registry & Fix `omr-api`

**Goal:** Move from an unauthenticated HTTP registry to a secure, production-like artifact store and resolve the `ImagePullBackOff` failure in `omr-saas`.

- **1.1 Diagnose and Fix `omr-api` (`omr-saas`):**
  - Inspect pod events (`kubectl describe pod -n omr-saas`) to confirm whether `ImagePullBackOff` is caused by an insecure HTTP `containerd` mirror config, missing image tag, or unreachable `local-registry`.
  - Configure `containerd` registry mirrors or TLS so worker nodes trust the internal registry securely.
- **1.2 Upgrade Registry Capabilities:**
  - Introduce **Harbor** (or secure the existing OCI registry with TLS + basic auth + `Retain` PersistentVolume).
  - Enable image retention policies to prevent `local-lvm` (`348.8 GB` thin pool) exhaustion.
  - Integrate **Trivy** scanning to flag vulnerable OS/library packages in pushed images.

---

## Milestone 2: Deploy L7 Ingress Controller & Automated TLS

**Goal:** Eliminate direct `NodePort` exposure (`30500`, `30777`, `32000`) and unify traffic routing from `cloudflared` (LXC `100`) into the cluster.

- **2.1 Install an Ingress Controller (Traefik or NGINX Ingress):**
  - Deploy into the existing `nginx` namespace (or a dedicated `ingress-nginx` / `traefik` namespace) and configure a predictable internal endpoint for `cloudflared`.
- **2.2 Add `cert-manager` for Internal TLS:**
  - Automate TLS certificate issuance for internal `<INTERNAL_DOMAIN>` routes using a self-signed ClusterIssuer or Cloudflare DNS-01 challenge.
- **2.3 Clean Up Legacy `default` Workloads:**
  - Audit test deployments (`httpd`, `nginx-1`, `mymysql` service) and unused namespaces (`backend`) to separate sandbox experiments from platform services.

---

## Milestone 3: Complete the Observability Stack (Logs & Alerting)

**Goal:** Pair existing Prometheus/Grafana metrics (`kube-prom-stack` + `pve-exporter`) with centralized logging and actionable alerts.

- **3.1 Deploy Grafana Loki + Alloy/Promtail:**
  - Collect container stdout/stderr logs across `k8s-master`, `k8s-worker-1`, and `k8s-worker-2` so pod crashes and `ImagePullBackOff` events are searchable alongside CPU/memory metrics in Grafana.
- **3.2 Harden Alertmanager & Dashboards as Code:**
  - Configure Alertmanager receivers (Discord, Slack, or email via secret manager) for node downtime, disk pressure on `local-lvm`, and pods stuck in `CrashLoopBackOff`/`ImagePullBackOff`.
  - Version-control custom Proxmox (`pve-exporter`) and Kubernetes Grafana dashboards in `kubernetes/monitoring/`.
- **3.3 Monitor Host & Node Stability:**
  - Investigate recent cluster-wide pod restarts (~9 minutes before the audit) to verify whether they resulted from a planned host reboot or resource pressure (`k8s-worker-2` has only `2048 MB` RAM).

---

## Milestone 4: Adopt GitOps, CI/CD & Infrastructure as Code

**Goal:** Replace imperative `kubectl` commands and manual VM cloning with reproducible Git-driven workflows.

- **4.1 Automate Proxmox Provisioning (OpenTofu + Ansible):**
  - Codify VM creation from `ubuntu-2204-template` (VMID `9000`) inside `infrastructure/` using OpenTofu/Terraform (`bpg/proxmox` or `telmate/proxmox`) and configure Kubernetes prerequisites with Ansible.
- **4.2 Implement CI Pipelines (GitHub Actions):**
  - Build container images on push, run linting/vulnerability scans (Trivy + `kubeconform`), and push signed/tagged artifacts to the container registry.
- **4.3 Deploy ArgoCD for GitOps Continuous Delivery:**
  - Install **ArgoCD** to watch `kubernetes/` in this repository, automatically syncing manifests and Helm values (with Sealed Secrets or External Secrets Operator for credentials).
- **4.4 Improve Storage Durability:**
  - Evaluate changing critical PVCs to a `Retain` reclaim policy or deploying **Longhorn** / **NFS CSI** backed by dedicated storage, plus scheduled Proxmox backups (`vzdump`).

---

## How It Works & Key Concepts

- **Why a Phased Roadmap Matters (Interview Perspective):** Platform engineering improvements must respect dependency order. Fixing image pull trust and L7 Ingress before introducing ArgoCD prevents GitOps from continuously reconciling broken workloads or insecure `NodePort` endpoints.
- **Metrics vs. Logs vs. Traces (The Three Pillars of Observability):** `kube-prom-stack` already provides **metrics** (time-series numbers via Prometheus and `pve-exporter`). Adding **Loki** supplies **logs** (contextual event text) without indexing full log bodies—only labels—making it lightweight enough for a `2–4 GB` RAM homelab node.
- **GitOps Pull Model (ArgoCD) vs. Push CI:** Rather than giving an external CI runner direct `kubeconfig` cluster-admin access (a security risk), CI only builds and pushes images/manifests to Git, while **ArgoCD** runs *inside* the cluster and pulls desired state from Git.
- **Storage Reclaim Policies (`Delete` vs. `Retain`):** `rancher.io/local-path` defaults to `Delete`, meaning deleting a PersistentVolumeClaim (PVC) immediately wipes the underlying directory on the worker node. Switching stateful apps (Prometheus, Harbor, databases) to `Retain` or distributed storage protects against accidental data loss.
