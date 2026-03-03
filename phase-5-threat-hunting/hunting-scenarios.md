
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

✔ Hypothesis confirmed  
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

✔ Web reconnaissance activity confirmed  
✔ Risk level assessed as low  

---

# 🔐 Scenario 3 — Successful SSH Access Verification

## 🎯 Hypothesis

An attacker may have successfully authenticated via SSH.

## 🔍 Investigation (Grafana / Loki)

```
{job="system", stream="auth"} |= "Accepted"
```

![SSH Accepted Events](../assets/hunting_4.png)

### Findings

- Only legitimate administrator logins observed  
- No suspicious post-authentication activity  

### Conclusion

❌ Hypothesis not confirmed  
✔ No evidence of compromise  

Absence of detection is a valid threat hunting outcome.

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

❌ Hypothesis not confirmed  
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

❌ Hypothesis not confirmed  
✔ No persistence detected  

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

- CrowdSec mitigates opportunistic attacks  
- Suricata NIPS reduces reconnaissance noise  
- Defense-in-depth strategy validated  

---

# 🏁 Final Conclusion

The threat hunting exercise confirms:

- Real exposure to opportunistic attacks  
- No evidence of compromise  
- Defensive layers operating effectively  
- Security posture validated through structured hypothesis testing  

The environment demonstrates a controlled and monitored security architecture.

---

# 🚀 End of Phase 5 — Security Posture Validation Complete
