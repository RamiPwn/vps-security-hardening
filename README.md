# 🛡️ VPS Security Hardening & Blue Team Operations

[![LinkedIn](https://img.shields.io/badge/LinkedIn-Rami%20Y.-blue?style=flat&logo=linkedin)](https://www.linkedin.com/in/rami-y-4286302a9/)
[![Framework](https://img.shields.io/badge/Framework-MITRE%20ATT%2CCK-red.svg)](#)
[![Stack](https://img.shields.io/badge/Stack-LGMA%20%2B%20VictoriaMetrics-orange.svg)](#)
[![IPS](https://img.shields.io/badge/IPS-CrowdSec%20%2B%20Suricata-purple.svg)](#)
[![Environment](https://img.shields.io/badge/Environment-Production-critical.svg)](#)
[![Downtime](https://img.shields.io/badge/Downtime-Zero-brightgreen.svg)](#)

---

## 📝 Executive Summary

Comprehensive security reinforcement of a **production VPS** with **zero downtime**. Implementation of a **Defense-in-Depth** architecture combining system hardening, advanced observability, and dual-layer intrusion prevention (HIPS + NIPS). This project documents the full transition from a passive security posture to an active detection, prevention, and threat hunting operation.

> **Note:** For security reasons related to the production environment, raw configuration files (`.yml`, `.conf`) are intentionally omitted. Only the technical methodology and sanitized implementation steps are shared.

---

## 🏗️ Global Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                        INTERNET TRAFFIC                         │
└──────────────────────────────┬──────────────────────────────────┘
                               │
                               ▼
┌─────────────────────────────────────────────────────────────────┐
│              PHASE 4 — NIPS · Suricata (Inline NFQUEUE)         │
│         Deep Packet Inspection · TCP NULL/XMAS DROP             │
└──────────────────────────────┬──────────────────────────────────┘
                               │
                               ▼
┌─────────────────────────────────────────────────────────────────┐
│           PHASE 1 — SYSTEM HARDENING · UFW · sysctl             │
│      Surface Reduction · SYN Flood Protection · IP Spoofing     │
└──────────────────────────────┬──────────────────────────────────┘
                               │
                               ▼
┌─────────────────────────────────────────────────────────────────┐
│            PHASE 3 — HIPS · CrowdSec + nftables Bouncer         │
│       Behavioral Detection (SSH/NGINX) · Automated Banning      │
└──────────────────────────────┬──────────────────────────────────┘
                               │
                               ▼
┌─────────────────────────────────────────────────────────────────┐
│                     PRODUCTION SERVICES                         │
│                    Nginx · SSH · Applications                   │
└──────────────────────────────┬──────────────────────────────────┘
                               │
                    ┌──────────▼──────────┐
                    │                     │
                    ▼                     ▼
     ┌──────────────────────┐  ┌─────────────────────────┐
     │  PHASE 2 — LGMA      │  │  PHASE 3 — VictoriaMetrics│
     │  Loki · Grafana      │  │  CrowdSec Alerts & Metrics│
     │  Prometheus · Alloy  │  │  SOC Dashboard           │
     └──────────────────────┘  └─────────────────────────┘
                    │
                    ▼
     ┌──────────────────────┐
     │  PHASE 5 — THREAT    │
     │  HUNTING             │
     │  MITRE ATT&CK        │
     │  5 Real Scenarios    │
     └──────────────────────┘
```

---

## 📂 Project Structure & Documentation

Click on the links below to access the detailed technical write-ups:

### 1. [Phase 1: System Hardening](./phase-1-hardening/hardening-guide.md)
- **Kernel Optimization:** `sysctl` hardening for network protection (SYN Flood, IP Spoofing, ICMP redirect)
- **Surface Reduction:** Service auditing, Apache2 removal, attack surface minimization
- **Access Control:** SSH hardening, root login disabled
- **Network Filtering:** UFW deny-by-default policy
- **Log Management:** Strict retention and rotation policies

### 2. [Phase 2: Observability Stack](./phase-2-observability/monitoring-setup.md)
- **Deployment:** LGMA Stack (Loki, Grafana, Prometheus, Alloy) via Docker
- **System Metrics:** Node Exporter dashboard (CPU, RAM, Disk, Network)
- **Container Metrics:** cAdvisor per-container monitoring
- **Log Centralization:** NGINX logs via Alloy → Loki pipeline
- **GeoIP Enrichment:** Country-level traffic analysis in Grafana

### 3. [Phase 3: HIPS with CrowdSec](./phase-3-hips-crowdsec/intrusion-prevention.md)
- **Behavioral Analysis:** Log-based detection (Nginx, SSH brute-force scenarios)
- **GeoIP Enrichment:** Country + ASN per alert
- **Alerting:** SMTP real-time notifications
- **Storage Optimization:** **VictoriaMetrics** for CrowdSec metrics ingestion
- **Remediation:** Automated blocking via **nftables Bouncers** (kernel-level)
- **Migration:** Full Fail2ban → CrowdSec transition

### 4. [Phase 4: NIPS with Suricata](./phase-4-nips-suricata/network-protection.md)
- **IDS → NIPS Transition:** Staged deployment with baseline benchmarking
- **Traffic Inspection:** Deep Packet Inspection (DPI) in **Inline mode**
- **Fail-Closed:** **NFQUEUE** configuration ensuring no traffic bypasses inspection
- **Zero-Downtime:** Hot rule reload without traffic interruption
- **Rollback:** Safe fallback procedure documented

### 5. [Phase 5: Threat Hunting Reports](./phase-5-threat-hunting/hunting-scenarios.md)
- **Methodology:** Hypothesis-driven, non-intrusive investigation framework
- **Scenarios:** 5 real-world attack scenarios analyzed on live production data
- **MITRE ATT&CK Mapping:** Full technique coverage per scenario
- **Defense Validation:** Cross-layer correlation (CrowdSec + Suricata + logs)

---

## 🛠️ Core Tech Stack

| Layer | Technology | Role |
|---|---|---|
| **Network IPS** | Suricata | Inline DPI, NFQUEUE DROP |
| **Host IPS** | CrowdSec | Behavioral detection, nftables banning |
| **Firewall** | nftables / UFW | Kernel-level enforcement |
| **Metrics** | Prometheus + VictoriaMetrics | System + CrowdSec metrics |
| **Logs** | Loki + Grafana Alloy | Centralized log pipeline |
| **Visualization** | Grafana | Dashboards + SOC views |
| **OS / Env** | Linux (Debian/Ubuntu), Docker, Bash | Runtime environment |

---

## 🎯 Key Highlights

- ✅ **Production environment** — not a lab or sandbox
- ✅ **Zero downtime** throughout all phases
- ✅ **Dual-layer IPS** — HIPS (host) + NIPS (network)
- ✅ **Full observability** — metrics, logs, GeoIP correlated
- ✅ **MITRE ATT&CK** validated threat hunting
- ✅ **No sensitive data exposed** — configs and URLs intentionally omitted

---

**Connect with me:** [Rami Y. on LinkedIn](https://www.linkedin.com/in/rami-y-4286302a9/)
