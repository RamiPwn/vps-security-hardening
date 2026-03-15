# 🧠 Phase 5 – Threat Hunting & MITRE ATT&CK Validation

## 📝 Executive Summary

This phase introduces a structured **threat hunting exercise conducted on a live Linux production server**.

Approach characteristics:

- Hypothesis-driven methodology
- Non-intrusive investigation
- Real production log analysis
- MITRE ATT&CK mapping
- Correlation with deployed security layers (CrowdSec + Suricata)

### Objectives

- Validate real-world exposure
- Confirm effectiveness of defensive layers
- Identify potential indicators of compromise (IOC)
- Assess overall security posture

All data has been anonymized.

---

# 🔬 Methodology

## Hypothesis-Driven Threat Hunting

This exercise follows a **structured hypothesis-driven approach** rather than reactive alerting. Each scenario begins with a defined hypothesis based on known attack techniques, followed by a targeted investigation using available telemetry.

```
Define Hypothesis (MITRE ATT&CK)
          │
          ▼
  Select Data Sources
  (logs, metrics, CLI)
          │
          ▼
  Execute Investigation
  (queries, commands)
          │
          ▼
  Analyze Evidence
          │
     ┌────┴────┐
     ▼         ▼
Confirmed   Not Confirmed
(IOC found) (clean — documented)
     │         │
     ▼         ▼
  Remediate  Record as
  & Document  Validated Baseline
```

## Data Sources Used

| Source | Tool | Data Type |
|---|---|---|
| SSH authentication logs | `journalctl` / Grafana Loki | Auth events, failed logins |
| NGINX access logs | Grafana Loki (LogQL) | HTTP requests, status codes |
| System auth logs | Grafana Loki | sudo usage, session events |
| Firewall decisions | `cscli decisions list` | Banned IPs, triggered scenarios |
| Cron and account state | CLI (`getent`, `ls`) | Persistence mechanisms |

## Investigative Principles

- **Non-intrusive:** No active scanning or service disruption
- **Evidence-based:** Every conclusion is backed by observable log data
- **Falsifiable:** A non-confirmed hypothesis is a valid and documented result
- **Correlated:** CLI findings are cross-checked against Grafana/Loki where possible

---

# 🔎 Scenario 1 — External SSH Brute Force Attempts

## 🎯 Hypothesis

An external attacker may attempt repeated SSH authentication failures (brute force or account scanning).

## 🧭 MITRE ATT&CK Mapping

- **Tactic:** Initial Access
- **[T1110.001 – Brute Force: SSH ↗](https://attack.mitre.org/techniques/T1110/001/)**
- **[T1087 – Account Discovery ↗](https://attack.mitre.org/techniques/T1087/)**

## 🔍 Investigation (CLI Analysis)

```bash
sudo journalctl -u ssh --since today | grep -E "Failed password" | head
```

Identification of most active IPs:

```bash
sudo journalctl -u ssh --since today \
| grep -E "Failed password|Invalid user" \
| awk '{for(i=1;i<=NF;i++) if($i=="from") print $(i+1)}' \
| sort | uniq -c | sort -nr | head
```

![SSH Brute Force Evidence](../assets/hunting_1.png)

### Findings

- Multiple failed SSH attempts observed
- IP-based hierarchy established
- Clear brute-force and account scanning patterns

## 🛡 Defensive Correlation

```bash
sudo cscli decisions list
```

![CrowdSec Decisions](../assets/hunting_3.png)

CrowdSec automatically banned the most aggressive IP addresses.

### Conclusion

✔ Hypothesis **confirmed**
✔ Real SSH exposure observed
✔ Defensive controls partially mitigate risk

---

# 🌐 Scenario 2 — Web Reconnaissance Activity

## 🎯 Hypothesis

An attacker may perform reconnaissance against exposed NGINX services.

## 🧭 MITRE ATT&CK Mapping

- **[T1595.001 – Active Scanning: Web ↗](https://attack.mitre.org/techniques/T1595/001/)**
- **[T1046 – Network Service Discovery ↗](https://attack.mitre.org/techniques/T1046/)**
- **[T1083 – File and Directory Discovery ↗](https://attack.mitre.org/techniques/T1083/)**

## 🔍 Investigation (NGINX Logs)

```bash
grep -E "wp-admin|\.git|\.env|phpinfo|server-status|\.bak|\.old" /var/log/nginx/access.log
```

![Web Recon Evidence](../assets/hunting_2.png)

### Findings

- Automated probing of `.git`, `.env`, `wp-admin`
- Majority returned HTTP 404
- No confirmed exploitation observed

### Conclusion

✔ Hypothesis **confirmed**
✔ Web reconnaissance activity confirmed
✔ Risk level assessed as low

---

# 🔐 Scenario 3 — Successful SSH Access Verification

## 🎯 Hypothesis

An attacker may have successfully authenticated via SSH.

## 🧭 MITRE ATT&CK Mapping

- **[T1078 – Valid Accounts ↗](https://attack.mitre.org/techniques/T1078/)**

## 🔍 Investigation (Grafana / Loki)

```
{job="system", stream="auth"} |= "Accepted"
```

![SSH Accepted Events](../assets/hunting_4.png)

### Findings

- Only legitimate administrator logins observed
- No suspicious post-authentication activity

### Conclusion

❌ Hypothesis **not confirmed**
✔ No evidence of compromise

> Absence of detection is a valid threat hunting outcome. A clean result validates the defensive posture rather than indicating an investigative failure.

---

# 🧑‍💻 Scenario 4 — Privilege Escalation Attempt (sudo)

## 🎯 Hypothesis

A user may attempt privilege escalation via sudo misuse.

## 🧭 MITRE ATT&CK Mapping

- **[T1548 – Abuse Elevation Control Mechanism ↗](https://attack.mitre.org/techniques/T1548/)**
- **[T1068 – Exploitation for Privilege Escalation ↗](https://attack.mitre.org/techniques/T1068/)**

## 🔍 Investigation

```
{job="system", stream="auth"} |= "sudo"
```

```
{job="system"} |~ "permission denied|access denied"
```

### Findings

- Legitimate sudo usage only
- No abnormal elevation attempts

### Conclusion

❌ Hypothesis **not confirmed**
✔ No privilege escalation detected

---

# 🕒 Scenario 5 — Persistence Mechanisms (Cron / Accounts)

## 🎯 Hypothesis

An attacker may establish persistence via scheduled tasks or account manipulation.

## 🧭 MITRE ATT&CK Mapping

- **[T1053.003 – Scheduled Task/Job: Cron ↗](https://attack.mitre.org/techniques/T1053/003/)**
- **[T1136 – Create Account ↗](https://attack.mitre.org/techniques/T1136/)**
- **[T1098 – Account Manipulation ↗](https://attack.mitre.org/techniques/T1098/)**

## 🔍 Investigation

```bash
ls -l /etc/cron.*
```

```bash
getent passwd | cut -d ":" -f 1
```

![Cron Inspection](../assets/hunting_5.png)

### Findings

- Only standard system cron jobs present
- No suspicious accounts created
- No unauthorized persistence mechanisms identified

### Conclusion

❌ Hypothesis **not confirmed**
✔ No persistence detected

---

# 📊 MITRE ATT&CK Coverage Summary

| Scenario | Tactic | Technique | Result | Mitigated By |
|---|---|---|---|---|
| SSH Brute Force | Initial Access | T1110.001 | ✔ Confirmed | CrowdSec (auto-ban) |
| Web Reconnaissance | Reconnaissance | T1595.001, T1046, T1083 | ✔ Confirmed | Suricata NIPS + 404 returns |
| Successful SSH Access | Initial Access | T1078 | ❌ Not confirmed | SSH hardening (Phase 1) |
| Privilege Escalation | Privilege Escalation | T1548, T1068 | ❌ Not confirmed | Restricted sudo policy |
| Persistence | Persistence | T1053.003, T1136, T1098 | ❌ Not confirmed | Minimal user accounts |

---

# 📊 Global Security Assessment

### Observed

- Opportunistic SSH brute-force attempts
- Automated web reconnaissance

### Not Observed

- Successful compromise
- Privilege escalation
- Persistence mechanisms

### Defensive Impact

| Layer | Contribution |
|---|---|
| **CrowdSec (HIPS)** | Automated banning of brute-force sources |
| **Suricata (NIPS)** | Reduction of reconnaissance noise at network level |
| **SSH Hardening** | Prevention of unauthorized access |
| **UFW / nftables** | Exposure minimization |
| **Observability (LGMA)** | Full log visibility enabling investigation |

---

# 🏁 Final Conclusion

The threat hunting exercise confirms:

- ✔️ Real exposure to opportunistic attacks
- ✔️ No evidence of compromise
- ✔️ Defensive layers operating effectively
- ✔️ Security posture validated through structured hypothesis testing
- ✔️ Full MITRE ATT&CK coverage across 5 realistic attack scenarios

The environment demonstrates a **controlled, monitored, and validated security architecture** — built on a production system with zero downtime throughout all phases.

---

# 🚀 End of Phase 5 — Security Posture Validation Complete
