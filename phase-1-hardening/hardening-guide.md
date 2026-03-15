# 🛡️ Phase 1 – System Hardening & Surface Reduction

## 📝 Executive Summary

The objective of this phase is to secure a production **Nginx web server** hosting multiple websites, using a progressive and structured hardening approach — while guaranteeing **zero service interruption** throughout.

This strategy is built on four main pillars:

- **Service Audit & Cleanup** — reduce the attack surface
- **Restrictive Network Filtering** — UFW deny-by-default policy
- **Anti-Brute-Force Protection** — Fail2ban (later replaced by CrowdSec in Phase 3)
- **Linux Kernel Hardening** — via `sysctl`

> [!IMPORTANT]
> All actions were performed **without service interruption**, ensuring full availability of hosted websites throughout the entire hardening process.

---

## 🧠 Hardening Philosophy

Before executing any changes, a guiding principle was established:

> **Reduce first. Restrict second. Harden third.**

Each action follows this order of priority:

1. **Eliminate** what is unnecessary (services, packages, open ports)
2. **Restrict** what remains (firewall rules, SSH policy)
3. **Harden** what is exposed (kernel parameters, brute-force protection)

This avoids the common mistake of layering security tools on top of a bloated, unaudited system.

---

## 🏗️ Phase 1 Architecture Overview

```
┌──────────────────────────────────────────────────────┐
│                    INTERNET TRAFFIC                  │
└──────────────────────────┬───────────────────────────┘
                           │
                           ▼
              ┌────────────────────────┐
              │   UFW (Deny-by-Default)│
              │   Allow: 22, 80, 443   │
              └────────────┬───────────┘
                           │
                           ▼
              ┌────────────────────────┐
              │      Fail2ban          │
              │   SSH Brute-Force      │
              │   Protection           │
              └────────────┬───────────┘
                           │
                           ▼
              ┌────────────────────────┐
              │   Linux Kernel         │
              │   sysctl Hardening     │
              │   SYN / Spoof / ICMP   │
              └────────────┬───────────┘
                           │
                           ▼
              ┌────────────────────────┐
              │   Nginx (Production)   │
              │   Minimal Services     │
              └────────────────────────┘
```

---

# 🔍 Section 1 – Service Analysis & Attack Surface Reduction

The first step is to identify all services present on the server and verify their legitimacy against the server's actual purpose.

## 🔎 Service Inventory

```bash
service --status-all
```

### 📸 Service Audit

![Service Audit](../assets/hardening_1.png)

The audit revealed the presence of **Apache2** (`[ + ]` status), which was unused and redundant alongside Nginx. Every unnecessary active service represents an additional attack vector.

---

## 🔎 Open Port Analysis

A check of open ports was performed to identify services exposed on the network, ensuring only strictly necessary ports are reachable from the outside.

> **Note:** The port scan output is intentionally omitted from this documentation for security reasons (production environment).

---

## 🗑️ Removal of Redundant Service

Apache2 was not functional and had no role in the current architecture. It was removed to eliminate unnecessary exposure.

```bash
sudo systemctl stop apache2
sudo apt remove apache2
```

---

## 🔐 SSH Access Hardening

The server is hosted on an OVH VPS. Root SSH access is not used for daily administration. All operations are performed via a dedicated user account with appropriate `sudo` privileges.

No changes were made to the existing SSH configuration to avoid any risk of production lockout.

---

# 🛡️ Section 2 – Intrusion Prevention (UFW & Fail2ban)

## 🔎 Fail2ban – Anti-Brute-Force Protection

Fail2ban was already present and active on the server. Its configuration was audited to confirm SSH protection was properly in place.

### Service Status Verification

```bash
sudo systemctl status fail2ban
```

### Jail Analysis

```bash
sudo fail2ban-client status
sudo fail2ban-client status sshd
```

The `sshd` jail was confirmed active with:

- Multiple failed authentication attempts tracked
- Several IPs already banned
- Real-time log monitoring operational

### Jail Status & Recent Bans

```bash
sudo fail2ban-client status sshd
```

### 📸 Fail2ban sshd Jail Status

![Fail2ban sshd Jail](../assets/hardening_2.png)

> **Note:** Fail2ban serves as the initial brute-force protection layer in this phase.
> It is fully replaced by CrowdSec in [Phase 3](../phase-3-hips-crowdsec/intrusion-prevention.md), which provides behavioral detection, GeoIP enrichment, and full observability integration.

---

## 🔥 UFW – Deny-by-Default Firewall Policy

A strict firewall policy was implemented following a **deny-all, allow-by-exception** model.

### Pre-Activation Checks

Before any configuration, UFW's presence and current state were verified:

```bash
sudo systemctl status ufw
sudo ufw --version
sudo ufw status verbose
```

UFW was confirmed present (v0.36.2) and **inactive** at this stage — allowing rules to be prepared without immediate impact on live connections.

---

### Rule Configuration

SSH was authorized **first** to guarantee administrative access would not be interrupted upon activation:

```bash
sudo ufw allow ssh
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp
```

Only essential ports are exposed:

| Port | Protocol | Service |
|------|----------|---------|
| 22   | TCP      | SSH     |
| 80   | TCP      | HTTP    |
| 443  | TCP      | HTTPS   |

### Pre-Activation Rule Verification

```bash
sudo ufw show added
```

Confirmed rules before activation:

```
ufw allow 22/tcp
ufw allow 80/tcp
ufw allow 443/tcp
```

### Firewall Activation

```bash
sudo ufw enable
```

> The standard SSH disruption warning was acknowledged and accepted after confirming SSH was properly allowed.

### Final UFW State Verification

```bash
sudo ufw status verbose
```

### 📸 UFW Configuration

![UFW Configuration](../assets/hardening_3.png)

Confirmed:

- Status: **active**
- Default policy: `deny (incoming)`, `allow (outgoing)`
- Rules: ports 22, 80, 443 — IPv4 and IPv6

---

# ⚙️ Section 3 – Linux Kernel Hardening (sysctl)

Network kernel parameters were optimized to mitigate low-level network attacks that operate below the application and firewall layers.

## 🔎 Pre-Hardening Audit

Before any modification, existing kernel security parameters were reviewed:

```bash
sysctl -a | grep syn
sysctl -a | grep rp
```

Only basic default protections were found to be active. The following hardening measures were then applied progressively.

---

## 🛡️ Parameters Applied

### SYN Flood Protection

```bash
sudo sysctl -w net.ipv4.tcp_syncookies=1
```

Enables SYN cookies — allows the server to handle abusive TCP connection attempts without impacting legitimate connections.

---

### Source Routing Disablement

```bash
sudo sysctl -w net.ipv4.conf.all.accept_source_route=0
sudo sysctl -w net.ipv4.conf.default.accept_source_route=0
```

Source routing is an obsolete and potentially dangerous feature. Disabling it reduces routing manipulation risk.

---

### ICMP Redirect Disablement

```bash
sudo sysctl -w net.ipv4.conf.all.accept_redirects=0
sudo sysctl -w net.ipv4.conf.default.accept_redirects=0
sudo sysctl -w net.ipv4.conf.all.send_redirects=0
```

Prevents unauthorized modifications to the server's routing table via ICMP redirects (Man-in-the-Middle mitigation).

---

### Reverse Path Filtering (IP Spoofing Prevention)

```bash
sudo sysctl -w net.ipv4.conf.all.rp_filter=1
sudo sysctl -w net.ipv4.conf.default.rp_filter=1
```

Strict mode — compatible with a standard VPS environment, no impact on production services.

---

### ICMP Noise Reduction

```bash
sudo sysctl -w net.ipv4.icmp_echo_ignore_broadcasts=1
sudo sysctl -w net.ipv4.icmp_ignore_bogus_error_responses=1
```

Reduces processing of unnecessary ICMP packets.

---

## ✅ Hot Validation (Before Persistence)

All parameters were first tested live — without a server reboot — to verify their impact on the production environment.

Result:

- Websites remained fully accessible
- Nginx and SSH continued operating normally
- No disruption observed

---

## 📁 Persistent Configuration

Once validated, parameters were made permanent via a dedicated configuration file:

```bash
/etc/sysctl.d/99-minimal-hardening.conf
```

### 📸 Sysctl Configuration File

![Sysctl Config](../assets/hardening_4.png)

Full parameter set written to the file:

```ini
net.ipv4.tcp_syncookies=1
net.ipv4.conf.all.accept_source_route=0
net.ipv4.conf.default.accept_source_route=0
net.ipv4.conf.all.accept_redirects=0
net.ipv4.conf.default.accept_redirects=0
net.ipv4.conf.all.send_redirects=0
net.ipv4.conf.all.rp_filter=1
net.ipv4.conf.default.rp_filter=1
net.ipv4.icmp_echo_ignore_broadcasts=1
net.ipv4.icmp_ignore_bogus_error_responses=1
```

Using a dedicated file under `/etc/sysctl.d/` ensures parameters survive reboots and are clearly separated from system defaults.

---

## 🚀 Apply Without Reboot

```bash
sudo sysctl --system
```

### 📸 Applying Kernel Parameters

![Sysctl Apply](../assets/hardening_5.png)

---

# ✅ Section 4 – Final Technical Validation

All applied parameters were verified to confirm active and persistent status:

```bash
sysctl net.ipv4.tcp_syncookies
sysctl net.ipv4.conf.all.rp_filter
```

Expected output:

```
net.ipv4.tcp_syncookies = 1
net.ipv4.conf.all.rp_filter = 1
```

### 📸 Final Verification

![Final Verification](../assets/hardening_6.png)

A returned value of `1` on all parameters confirms the hardening is active and persistent.

---

## 📊 Complete Hardening Summary

| Parameter | Value | Protection | Attack Mitigated |
|---|---|---|---|
| `tcp_syncookies` | 1 | SYN Cookies | SYN Flood |
| `accept_source_route` (all/default) | 0 | Source route block | Routing manipulation |
| `accept_redirects` (all/default) | 0 | ICMP redirect block | Man-in-the-Middle |
| `send_redirects` (all) | 0 | Outbound redirect block | Routing manipulation |
| `rp_filter` (all/default) | 1 | Reverse Path Filtering | IP Spoofing |
| `icmp_echo_ignore_broadcasts` | 1 | Broadcast ICMP block | Smurf / amplification |
| `icmp_ignore_bogus_error_responses` | 1 | Bogus ICMP block | ICMP noise / recon |

---

# 🚀 Conclusion

The VPS now benefits from:

- ✔️ Minimized attack surface (Apache2 removed, unused services audited)
- ✔️ Strict network filtering (UFW deny-by-default, ports 22/80/443 only)
- ✔️ Active brute-force protection (Fail2ban — sshd jail confirmed operational)
- ✔️ Hardened Linux kernel (7 parameters — SYN Flood, IP Spoofing, ICMP, Source Routing)
- ✔️ Zero service interruption throughout all actions
- ✔️ Persistent configuration (reboot-safe via `/etc/sysctl.d/`)

---

➡️ **Next: [Phase 2 – Observability Stack Deployment](../phase-2-observability/monitoring-setup.md)**
