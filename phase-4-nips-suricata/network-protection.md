# 🔥 Phase 4 – Network IPS with Suricata (IDS ➜ Inline NIPS)

## 📝 Executive Summary

After implementing:

- ✔️ System hardening & surface reduction (Phase 1)
- ✔️ Full observability stack — LGMA (Phase 2)
- ✔️ Host-level behavioral prevention — CrowdSec (Phase 3)

This phase introduces a **Network Intrusion Prevention System (NIPS)** using Suricata in inline mode.

Deployment strategy:

1. Install Suricata and validate configuration
2. Deploy in IDS mode (detection only)
3. Validate logging & visibility
4. Benchmark traffic baseline
5. Switch to NFQUEUE inline mode
6. Enable controlled DROP rules
7. Implement zero-downtime rule updates

Objectives:

- Block real network scans (Nmap, NULL, XMAS)
- Avoid impacting SSH / HTTP / HTTPS / IPv6
- Maintain full observability
- Guarantee safe rollback at every step

---

# ⚙️ Section 0 — Suricata Installation & Prerequisites

## 📦 Install Suricata

```bash
sudo add-apt-repository ppa:oisf/suricata-stable
sudo apt-get update
sudo apt-get install suricata suricata-update
```

## 🔄 Update Rule Sets

```bash
sudo suricata-update
```

## 📋 Install Additional Rule Sources (Optional)

```bash
sudo suricata-update list-sources
sudo suricata-update enable-source et/open
sudo suricata-update
```

## 🔌 Required Kernel Module — NFQUEUE

NFQUEUE is a kernel module required for inline mode. Verify it is available:

```bash
lsmod | grep nfnetlink_queue
```

If not loaded:

```bash
sudo modprobe nfnetlink_queue
```

To persist across reboots:

```bash
echo "nfnetlink_queue" | sudo tee -a /etc/modules
```

---

# 🔎 Section 1 — Suricata IDS (Detection Mode)

## Configuration Validation

Before starting the service, configuration integrity was validated:

```bash
sudo suricata -T
```

![Suricata Config Test](../assets/network_1.png)

✔ Configuration successfully loaded
✔ No fatal errors
✔ Production-safe validation

This ensures Suricata starts with a clean configuration.

---

## Service Startup

```bash
sudo systemctl start suricata
sudo systemctl status suricata
```

![Suricata Service Running](../assets/network_2.png)

✔ Suricata running under systemd
✔ No crash loops
✔ Proper PID management

IDS mode is now active.

---

# 📡 Section 2 — Traffic Inspection & Log Integration

Suricata inspects traffic directly on the public interface.

Example TLS event captured from `eve.json`:

![TLS Event Captured](../assets/network_3.png)

Observed fields:

- Source IP
- Destination IP
- TLS version
- SNI hostname
- Direct capture from interface

✔ Deep Packet Inspection confirmed
✔ Full visibility of encrypted session metadata

Logs are forwarded to Loki via Alloy for centralized analysis.

---

# 📊 Section 3 — IDS Baseline (Before Inline)

Initial Grafana dashboard (IDS mode):

![IDS Dashboard Baseline](../assets/network_4.png)

Observations:

- High alert volume
- Noise from non-exposed ports
- Recon attempts fully visible

✔ Detection working correctly
✔ Useful for baseline benchmarking

---

# 🔥 Section 4 — Transition to NIPS (Inline Mode)

## Architecture Principle

```
Internet Traffic
      │
      ▼
   nftables
   ├── SSH → ACCEPT (bypass NFQUEUE)
   ├── Established connections → ACCEPT
   ├── IPv6 → ACCEPT (excluded from inspection)
   └── HTTP/HTTPS (NEW, IPv4) → NFQUEUE
                                    │
                                    ▼
                                Suricata
                           (behavioral engine)
                           ├── ALLOW → packet forwarded
                           └── DROP  → packet blocked
```

- **UFW** → Controls which ports are exposed
- **nftables** → Selects which traffic enters NFQUEUE
- **Suricata** → Makes DROP/ACCEPT decisions on queued packets

Suricata does **not** inspect everything — only targeted traffic via NFQUEUE.

---

## nftables Traffic Redirection

```bash
sudo nft list table inet suricata_nips
```

![nftables NFQUEUE Table](../assets/network_5.png)

Key elements:

- Hook: input
- Priority: -150
- Established connections accepted
- SSH explicitly allowed
- IPv6 excluded
- HTTP/HTTPS (IPv4, NEW state) sent to NFQUEUE

✔ Minimal attack surface
✔ Performance optimized
✔ Explicit whitelist before inspection

---

## Local DROP Rules (Production-Safe)

Only high-confidence signatures enabled:

- TCP NULL scan
- TCP XMAS scan

Goal:

- Zero false positives
- Immediate scan blocking
- No impact on legitimate traffic

---

# 🚫 Section 5 — Effective DROP Verification

Real-time log monitoring confirms DROP execution:

![DROP Event Confirmation](../assets/network_6.png)

✔ DROP action visible in logs
✔ Not just alert — actual packet blocking
✔ Inline enforcement active

Suricata now operates as a **true Network IPS**.

---

# 📉 Section 6 — Telemetry Impact (After NIPS)

Grafana dashboard after inline activation:

![NIPS Dashboard Clean](../assets/network_7.png)

Observed improvements:

- Noise dramatically reduced
- Non-exposed port scans dropped before logging
- Cleaner telemetry signal
- Reduced log ingestion load

✔ Better readability
✔ Lower system overhead
✔ Security signal becomes actionable

This behavior is expected and desired in production.

---

# ⚙️ Section 7 — Production Hardening (Service Mode)

Suricata permanently configured in NFQUEUE mode.

Verification:

```bash
sudo systemctl status suricata
```

![Suricata NFQUEUE Mode](../assets/network_8.png)

Confirmed:

- Service: **active (running)** — managed by systemd
- Loaded with `override.conf` drop-in (NFQUEUE mode configuration)
- ExecStart confirms `-q 0 -D` flags — NFQUEUE queue 0, daemon mode
- ExecReload confirms hot rule reload via `suricatasc`
- Automatic startup on boot enabled

✔ Automatic startup
✔ Inline enforcement persistent
✔ No manual debug dependency

---

# 🔄 Section 8 — Zero-Downtime Rule Updates

Production-safe update strategy:

1. Download new rules
2. Validate with `suricata -T`
3. Apply `systemctl reload suricata`
4. Preserve old rules on failure

```bash
sudo suricata-update
sudo suricata -T -c /etc/suricata/suricata.yaml
sudo systemctl reload suricata
```

**Why `reload` and not `restart`?**

In inline (Fail-Closed) mode:

- A `restart` immediately drops live traffic during the gap between stop and start.
- A `reload` performs a hot rule swap while keeping NFQUEUE active.

✔ Pre-validation before deployment
✔ Hot reload
✔ No traffic interruption

---

# 🛟 Section 9 — Rollback Procedure

In the event of a misconfiguration or unexpected DROP behavior, a safe rollback procedure was defined before going inline.

## Step 1 — Stop Suricata

```bash
sudo systemctl stop suricata
```

With Fail-Closed (NFQUEUE), stopping Suricata without removing the nftables rules will block all queued traffic. **Always flush nftables rules first.**

## Step 2 — Remove NFQUEUE nftables Rules

```bash
sudo nft delete table inet suricata_nips
```

Traffic now bypasses Suricata entirely and flows normally through UFW.

## Step 3 — Verify Service Continuity

```bash
curl -I http://localhost
sudo systemctl status nginx
```

✔ Services remain reachable
✔ Zero downtime during rollback

## Step 4 — Diagnose Before Re-enabling

```bash
sudo suricata -T
sudo journalctl -u suricata -n 50
```

Fix the offending rule or configuration, re-validate, then re-apply the nftables table and restart Suricata.

> [!IMPORTANT]
> This rollback procedure was validated **before** activating inline mode on production.
> Never switch to NFQUEUE without a tested rollback path.

---

# ✅ Final Architecture State

The VPS now includes:

- ✔️ Host-level IPS (CrowdSec)
- ✔️ Network-level IPS (Suricata inline NFQUEUE)
- ✔️ Kernel-level enforcement (nftables)
- ✔️ Zero-downtime rule updates
- ✔️ Full observability via Loki & Grafana
- ✔️ Controlled minimal DROP rules
- ✔️ Documented production-safe rollback procedure

Suricata is used as a **targeted decision engine**, not as a generic firewall.

---

➡️ **Next: [Phase 5 – Threat Hunting & MITRE ATT&CK Validation](../phase-5-threat-hunting/hunting-scenarios.md)**
