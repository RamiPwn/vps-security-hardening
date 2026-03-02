# 🛡️ Phase 1 – System Hardening & Surface Reduction

## 📝 Executive Summary

The objective of this phase is to secure a production **Nginx web server** using a progressive and structured hardening approach.

This strategy is built on four main pillars:

- **Service Audit & Cleanup** to reduce the attack surface  
- **Restrictive Network Filtering** (UFW)  
- **Anti-Brute-Force Protection** (Fail2ban)  
- **Linux Kernel Hardening** via `sysctl`  

> [!IMPORTANT]  
> All actions were performed **without service interruption**, ensuring full availability of hosted websites.

---

# 🔍 Section 1 – Service Analysis & Attack Surface Reduction

An audit revealed the presence of **Apache2**, which was unused and redundant alongside Nginx.  
It was removed to eliminate an unnecessary attack vector.

## 🔎 Service Inventory

```bash
service --status-all
```

## 🗑 Removal of Redundant Service

```bash
sudo systemctl stop apache2
sudo apt-get purge apache2
```

## 🔐 SSH Access Hardening

- Disabled direct root login  
- Enforced usage of a dedicated user with `sudo` privileges  

### 📸 Evidence

![Service Audit](../assets/hardening_1.png)

---

# 🛡️ Section 2 – Intrusion Prevention (UFW & Fail2ban)

## 🔎 Fail2ban – Anti-Brute-Force Protection

Verification of active SSH protection through log monitoring:

```bash
sudo tail -n 50 /var/log/fail2ban.log
```

### 📸 Fail2ban Logs

![Fail2ban Logs](../assets/hardening_2.png)

---

## 🔥 UFW – Deny-by-Default Firewall Policy

A strict firewall policy was implemented:

Only essential ports are allowed:

- 22 → SSH  
- 80 → HTTP  
- 443 → HTTPS  

### Firewall Rule Configuration

```bash
sudo ufw allow ssh
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp
```

### Firewall Activation

```bash
sudo ufw enable
```

### 📸 UFW Configuration

![UFW Configuration](../assets/hardening_3.png)

---

# ⚙️ Section 3 – Linux Kernel Hardening (sysctl)

Network kernel parameters were optimized to mitigate low-level attacks.

## 🛡️ Implemented Protections

- **SYN Cookies** → Protection against SYN Flood attacks  
- **Reverse Path Filtering** → IP Spoofing prevention  
- **ICMP Redirect Disablement** → Mitigation against Man-in-the-Middle attacks  

---

## 📁 Persistent Configuration

Configuration file used:

```bash
/etc/sysctl.d/99-minimal-hardening.conf
```

### 📸 Sysctl Configuration

![Sysctl Config](../assets/hardening_4.png)

---

## 🚀 Apply Changes Without Reboot

```bash
sudo sysctl --system
```

### 📸 Applying Kernel Parameters

![Sysctl Apply](../assets/hardening_5.png)

---

# ✅ Section 4 – Final Technical Validation

Validation of critical security flags:

```bash
sysctl net.ipv4.tcp_syncookies
sysctl net.ipv4.conf.all.rp_filter
```

Expected output:

```
1
```

If the value returned is `1`, the hardening is successfully active.

### 📸 Final Verification

![Final Verification](../assets/hardening_6.png)

---

# 🚀 Conclusion

The VPS now benefits from:

- A minimized attack surface  
- Strict network filtering  
- Active brute-force protection  
- Hardened Linux kernel parameters  

The system is now secure and ready for **Phase 2: Observability Stack Deployment**.
