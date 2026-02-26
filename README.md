# 🛡️ VPS Security Hardening & Blue Team Operations

[![LinkedIn](https://img.shields.io/badge/LinkedIn-Rami%20Y.-blue?style=flat&logo=linkedin)](https://www.linkedin.com/in/rami-y-4286302a9/)
[![Framework](https://img.shields.io/badge/Framework-MITRE%20ATT%2CCK-red.svg)](#)
[![Stack](https://img.shields.io/badge/Stack-LGMA%20%2B%20VictoriaMetrics-orange.svg)](#)

### 📝 Executive Summary
Comprehensive security reinforcement of a production VPS (Zero-Downtime). Implementation of a **Defense-in-Depth** architecture combining system hardening, advanced observability, and dual-layer interception (HIPS/NIPS). This project documents the transition from passive security to an active detection and remediation posture.

> **Note:** For security reasons related to the production environment, raw configuration files (.yml, .conf) are intentionally omitted. Only the technical methodology and sanitized implementation steps are shared.

---

## 📂 Project Structure & Documentation

Click on the links below to access the detailed technical write-ups (Original reports in French):

### 1. [Phase 1: System Hardening](./phase-1-hardening/hardening-guide.md)
* **Kernel Optimization:** `sysctl` hardening for network protection.
* **Surface Reduction:** Service auditing and attack surface minimization.
* **Log Management:** Implementation of strict retention and rotation policies.

### 2. [Phase 2: Observability Stack](./phase-2-observability/monitoring-setup.md)
* **Deployment:** LGMA Stack (Loki, Grafana, Prometheus, Alloy) via Docker.
* **Visualization:** Custom dashboards and traffic monitoring.

### 3. [Phase 3: HIPS with CrowdSec](./phase-3-hips-crowdsec/intrusion-prevention.md)
* **Behavioral Analysis:** Log-based detection (Nginx, SSH).
* **Storage Optimization:** Integration of **VictoriaMetrics** specifically for CrowdSec logs/alerts ingestion.
* **Remediation:** Automated blocking via **nftables Bouncers** (Host IPS).

### 4. [Phase 4: NIPS with Suricata](./phase-4-nips-suricata/network-protection.md)
* **Traffic Inspection:** Deep Packet Inspection (DPI) in **Inline mode**.
* **Zero-Downtime:** **NFQUEUE** configuration with **Fail-Closed** strategy.
* **Automation:** Rule management and continuous monitoring.

### 5. [Phase 5: Threat Hunting Reports](./phase-5-threat-hunting/hunting-scenarios.md)
* **Scenarios:** Analysis of 5 real-world attack scenarios on the server.
* **Mapping:** Detection validation using the **MITRE ATT&CK** framework.
* **Defense Validation:** Evaluating the efficiency of the implemented security layers.

---

## 🛠️ Core Tech Stack
* **Security:** Suricata (NIPS), CrowdSec (HIPS), nftables.
* **Monitoring:** Grafana, Loki, **VictoriaMetrics (CrowdSec logs)**, Grafana Alloy.
* **OS/Env:** Linux (Debian/Ubuntu), Docker, Bash.

---
**Connect with me:** [Rami Y. on LinkedIn](https://www.linkedin.com/in/rami-y-4286302a9/)
