# Homelab Infrastructure

This repository documents a high-performance, local-first private cloud environment designed for data science workloads, secure synchronization, and automated infrastructure management.

## 🎓 Academic Competency Mapping
This infrastructure is architected to exceed the learning outcomes of CSC5090 (Systems Administration) and CSC5520 (Foundations of System Security).

### 🏛️ Pillar 1: Systems Administration & Virtualization (CSC5090)
- **Hypervisor Management:** Administration of a Proxmox VE cluster on GMKtec M5 Plus hardware, utilizing Debian 12 LXCs and specialized systemd service persistence.
- **Network Orchestration:** Management of complex multi-container stacks using Docker Compose, including health checks, volume persistence, and isolated bridge networks (e.g., `proxy-tier`).
- **Service Discovery:** Implementation of an Nginx Proxy Manager instance to handle SNI-based routing and internal name resolution.

### 🛡️ Pillar 2: System Security & Cloud Ingress (CSC5520 / CSC6711)
- **Zero-Trust Edge Ingress:** Utilization of Cloudflare Tunnels to expose specific internal services (e.g., University Marks Manager) securely without opening inbound firewall ports.
- **Identity & Access Management (IAM):** Enforcement of SSH ed25519 key-pair authentication and Tailscale VPN for all administrative traffic; password-based and root logins are strictly disabled.
- **Secure Containerization:** Implementation of multi-stage Docker builds (e.g., `Dockerfile.fastapi`) to minimize attack surfaces by removing build-time dependencies from the final production image.

---

## 🛡️ Network Security & Zero-Trust
My security strategy follows a **Zero-Trust** and **Defense-in-Depth** model to secure internal services within a localized environment.

- **Edge Ingress:** Outbound-only **Cloudflare Tunnel** deployed within an isolated Proxmox LXC (`cloudflare-gateway`) to publish authorized services without port forwarding.
- **Management Access:** All administrative traffic is strictly restricted to an encrypted **Tailscale mesh network**, ensuring management interfaces are never exposed to the public internet or external port scans.
- **Identity & Access:** Strict enforcement of **ed25519 SSH keys** with root login and password authentication disabled at the host OS level.
- **Network Segmentation:** Services are isolated across dedicated Proxmox LXCs and container network tiers (`proxy-tier`), preventing lateral movement in the event of container compromise.
- **Storage & Sync:** Sensitive academic files, datasets, and system backups synchronize via encrypted network mounts to a centralized Synology NAS.

---

## 🛠️ Hardware Stack
This environment is built on a dedicated local "Private Cloud" architecture:

- **Primary Hypervisor:** **GMKtec M5 Plus** running **Proxmox VE**, hosting data science pipelines and security services.
- **Storage Provider:** Centralized **Synology NAS** providing persistent NFS/SMB storage for university archives and datasets.
- **Workstations:** Chassis-aware **Fedora** environments (Desktop & Laptop) integrated via a private Tailscale mesh.

---

## 📂 Project Structure
- **`automation/fedora-setup/`**: Chassis-aware workstation hardening and sync logic.
  - `setup-fedora.sh`: Master script for hardware detection and baseline security.
  - `hosts/`: Machine-specific logic for discrete GPU configurations and laptop battery power profiles.
- **`infrastructure/docker/`**: Container definitions and compose specifications (`nginx-proxy-manager`, application stacks).
- **`infrastructure/scripts/`**: Automated maintenance routines, including `uni-archive-sync.sh` for versioned rclone backups.
- **`docs/`**: Threat models, architecture diagrams, and hardware specifications.

---

## 🧪 Security Testing & Verification (ICTNWK421)
To meet the verification requirements of ICTNWK421, automated scripts validate network integrity and perimeter security:

- **Automated Health Audits:** `status_check.sh` and `heartbeat.sh` collect real-time telemetry on container states and tunnel connectivity.
- **Ingress Validation:** Routine port scans verify zero listening external ports on the WAN interface, validating outbound-only tunnel routing.
- **Log Telemetry & Auditing:** Continuous inspection of `failover_log.txt` and `rsync_log.txt` to monitor backup integrity and detect unauthorized access anomalies.

---

## 🤖 Systems Automation & IaC
This lab applies an **Infrastructure as Code (IaC)** approach to maintain reproducible configurations across all nodes:

- **Chassis-Aware Hardening:** Automated detection of hardware profiles to apply tailored driver and power policies.
- **Data Integrity:** Fast bidirectional file sync using **Mutagen** and **rsync** between Proxmox worker nodes and the Synology NAS.
- **Disaster Recovery:** Automated archival routines maintaining a rolling 5-version rotation policy to protect against data corruption or hardware loss.

---

## 📊 Service Inventory & Security Matrix

| Service | Role | Network Tier | Security Layer |
| :--- | :--- | :--- | :--- |
| `marks-manager-web` | Academic Tracking App | `proxy-tier` (Isolated) | No exposed ports; routed via authenticated CF Tunnel |
| `nginx-proxy-manager` | Reverse Proxy / SNI | `proxy-tier` (Isolated) | SSL termination; SNI hostname matching |
| `adguard-home` | Network DNS / Filtering | Host (LXC) | Wildcard DNS rewrites for local hostnames; upstream DoH |
| `cloudflare-gateway` | Edge Ingress Tunnel | Host (LXC) | Outbound-only tunnel daemon (`cloudflared`); Zero-Trust edge |
| `synology-nas` | Persistent Storage | Management (Mesh) | Tailscale encrypted traffic; restricted NFS exports |