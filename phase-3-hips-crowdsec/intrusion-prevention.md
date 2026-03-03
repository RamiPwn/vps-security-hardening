# 🛡️ Phase 3 – HIPS with CrowdSec (Behavioral Detection & Active Remediation)

## 📝 Executive Summary

After:

- ✔️ Attack surface reduction (Phase 1)  
- ✔️ Full LGMA observability stack (Phase 2)  

This phase introduces a **Host Intrusion Prevention System (HIPS)** powered by CrowdSec.

Objectives:

- Real-time behavioral detection (SSH & NGINX)
- GeoIP enrichment (Country / ASN)
- Instant alerting (SMTP)
- Metrics centralization via VictoriaMetrics
- Automated remediation via nftables (kernel-level)
- Zero service interruption

The transition was performed progressively:

Observation ➜ Validation ➜ Remediation activation ➜ Fail2ban removal

---

# 🔎 Section 1 — Behavioral Detection (Observation Mode)

## 🎯 Objective

Validate the engine’s ability to:

- Properly parse logs
- Trigger scenarios (ssh-bf, ssh-slow-bf, http-probing…)
- Generate enriched alerts
- Without enabling immediate banning

---

## 📥 Log Ingestion Validation

Verification command:

```bash
sudo cscli metrics
```

![CrowdSec Metrics](../assets/crowdsec_1.png)

### 🔎 Analysis

This view confirms:

- Active ingestion of SSH & NGINX logs
- Functional parsers
- Buckets properly populated
- No critical parsing errors

👉 The engine correctly processes live production traffic.

---

# 🚨 Section 2 — Alerts & GeoIP Enrichment

## 📊 Reviewing Detected Alerts

```bash
sudo cscli alerts list
```

![CrowdSec Alerts](../assets/crowdsec_2.png)

### Enriched Data Includes:

- Source IP
- Country of origin
- ASN (Autonomous System Number)
- Triggered scenario (e.g., ssh-bf)

👉 This enrichment enables immediate SOC-level analysis.

---

## 📧 SMTP Alerting (Mobile Monitoring)

Notification configured via Gmail SMTP (application password required).

Test command:

```bash
sudo cscli notifications test email_default
```

![CrowdSec Email Alert](../assets/crowdsec_3.png)

Each alert contains:

- Banned IP address
- Triggered scenario
- Geolocation data
- Precise timestamp

🎯 Enables remote monitoring without direct SSH access.

---

# 📊 Section 3 — SOC Visualization (Grafana + VictoriaMetrics)

CrowdSec alerts are:

1. Written in NDJSON format
2. Collected via a dedicated Alloy pipeline
3. Ingested into VictoriaMetrics
4. Visualized in Grafana

---

## 🌍 Global Threat Overview Dashboard

![CrowdSec Overview](../assets/crowdsec_4.png)

This dashboard provides:

- Consolidated threat overview
- World threat map (Geomap)
- 24-hour attack volume
- Most active ASNs

---

## 🔐 SSH Brute Force Focus

![SSH Detection Panel](../assets/crowdsec_5.png)

Analysis includes:

- Time-based spikes on port 22
- Fast vs slow brute-force identification
- Correlation with time periods

👉 Critical tool for remediation decision-making.

---

# 🛡️ Section 4 — Remediation Activation (nftables Bouncer)

After validation in observation mode, the firewall bouncer was activated.

---

## 🧾 Strategic Allowlist Creation

Before activation:

- Administrator IPs
- VPS provider IPs
- External monitoring IPs

```bash
sudo cscli allowlists create admin_ips
sudo cscli allowlists add admin_ips "X.X.X.X"
```

---

## 🔑 Bouncer Creation

```bash
sudo cscli bouncers add nftables-prod
```

---

## ✅ Bouncer Registration Verification

```bash
sudo cscli bouncers list
```

![Bouncer Valid](../assets/crowdsec_6.png)

Expected status:

```
Valid
```

👉 The engine is now authorized to enforce decisions.

---

## 🚫 Active Decisions

```bash
sudo cscli decisions list
```

![Active Decisions](../assets/crowdsec_7.png)

Observed data:

- Banned IP addresses
- Triggering scenario
- Remaining ban duration
- Action type: ban

---

## ⚙️ Kernel-Level Injection (nftables)

```bash
sudo nft list table ip crowdsec
```

![nftables Injection](../assets/crowdsec_8.png)

Banned IPs appear directly inside nftables sets.

🎯 Enforcement characteristics:

- Kernel-level blocking
- No intermediate service
- Minimal CPU impact

High-performance prevention.

---

# ⚠️ Real Incident — Managing Rule Volume

An excessive number of dynamic decisions temporarily:

- Increased nftables rule complexity
- Caused firewall overhead

### Remediation applied:

```bash
sudo cscli decisions delete --all
```

Effects:

- Significant reduction of dynamic rules
- Firewall stabilization
- Continued detection capability

👉 Production-oriented pragmatic decision.

---

# 🔄 Final Transition

Once remediation was validated:

```bash
sudo systemctl stop fail2ban
sudo systemctl disable fail2ban
sudo apt remove --purge fail2ban
```

CrowdSec fully replaced Fail2ban with:

- Advanced behavioral detection
- Network enrichment
- Full observability integration
- High-performance automated blocking

---

# ✅ Conclusion

The VPS now benefits from:

- ✔️ Behavioral host intrusion prevention
- ✔️ Real-time enriched alerts
- ✔️ SOC-level visualization
- ✔️ Automated kernel-level blocking
- ✔️ Scalable and maintainable architecture

The Host IPS layer is fully operational.

The system is now ready for:

➡️ Phase 4 – NIPS with Suricata (Inline Deep Packet Inspection)
