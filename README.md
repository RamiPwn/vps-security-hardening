# vps-security-hardening
Zero-Downtime VPS Hardening. Multi-layer defense-in-depth: Linux Hardening, Observability (LGMA Stack + VictoriaMetrics), and dual-IPS deployment: CrowdSec (HIPS via nftables bouncer) for IP remediation, and Suricata (NIPS Fail-Closed mode) for network filtering. Proactive Threat Hunting based on MITRE ATT&amp;CK. Real-world production case study.
## 📂 Project Structure & Documentation

Click on the sections below to access the detailed technical write-ups (in French):

* **[Phase 1: System Hardening](./phase-1-hardening/hardening-guide.md)**
    * *Kernel optimization (sysctl), service cleanup, and security auditing.*
* **[Phase 2: Observability Stack](./phase-2-observability/monitoring-setup.md)**
    * *Deployment of LGMA Stack + VictoriaMetrics via Docker.*
* **[Phase 3: HIPS with CrowdSec](./phase-3-hips-crowdsec/intrusion-prevention.md)**
    * *Behavioral analysis and automated remediation with nftables bouncers.*
* **[Phase 4: NIPS with Suricata](./phase-4-nips-suricata/network-protection.md)**
    * *Deep Packet Inspection and Fail-Closed network protection.*
* **[Phase 5: Threat Hunting Reports](./phase-5-threat-hunting/hunting-scenarios.md)**
    * *Analysis of real attack scenarios based on MITRE ATT&CK.*
