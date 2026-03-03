# 🛡️ Phase 3 – HIPS with CrowdSec (Behavioral Detection & Active Remediation)

## 📝 Résumé Global

Après :

- ✔️ Réduction de surface d’attaque (Phase 1)  
- ✔️ Observabilité complète LGMA (Phase 2)  

Cette phase introduit une **Host Intrusion Prevention System (HIPS)** basée sur CrowdSec.

Objectifs :

- Détection comportementale en temps réel (SSH & NGINX)
- Enrichissement GeoIP (Pays / ASN)
- Alerting instantané (SMTP)
- Centralisation métrique via VictoriaMetrics
- Remédiation automatique via nftables (Kernel-level)
- Zéro interruption de service

La transition a été effectuée progressivement :
Observation ➜ Validation ➜ Activation remédiation ➜ Suppression Fail2ban

---

# 🔎 Section 1 — Détection comportementale (Mode Observation)

## 🎯 Objectif

Valider la capacité du moteur à :

- Parser correctement les logs
- Déclencher des scénarios (ssh-bf, ssh-slow-bf, http-probing…)
- Générer des alertes enrichies
- Sans bannissement initial

---

## 📥 Validation ingestion logs

Commande de vérification :

```bash
sudo cscli metrics
```

![CrowdSec Metrics](../assets/crowdsec_1.png)

### 🔎 Analyse

Cette vue confirme :

- Lecture active des logs SSH & NGINX
- Parsers fonctionnels
- Buckets alimentés
- Aucun parsing error critique

👉 Le moteur détecte correctement les flux de production.

---

# 🚨 Section 2 — Alertes & Enrichissement GeoIP

## 📊 Consultation des alertes détectées

```bash
sudo cscli alerts list
```

![CrowdSec Alerts](../assets/crowdsec_2.png)

### Données enrichies :

- IP source
- Pays d'origine
- ASN (Autonomous System Number)
- Scénario déclenché (ex: ssh-bf)

👉 Cette enrichissement permet une analyse SOC exploitable immédiatement.

---

## 📧 Alerting SMTP (Surveillance mobile)

Notification configurée via SMTP Gmail (mot de passe d’application).

Test :

```bash
sudo cscli notifications test email_default
```

![CrowdSec Email Alert](../assets/crowdsec_3.png)

Chaque alerte contient :

- IP bannie
- Scénario
- Localisation géographique
- Timestamp précis

🎯 Permet une surveillance hors accès SSH.

---

# 📊 Section 3 — Visualisation SOC (Grafana + VictoriaMetrics)

Les alertes CrowdSec sont :

1. Écrites en NDJSON
2. Collectées via Alloy dédié
3. Ingestées dans VictoriaMetrics
4. Visualisées dans Grafana

---

## 🌍 Dashboard Global

![CrowdSec Overview](../assets/crowdsec_4.png)

Ce dashboard permet :

- Vue consolidée des menaces
- Cartographie mondiale (Geomap)
- Volume d’attaques 24h
- ASN les plus actifs

---

## 🔐 Focus SSH Brute Force

![SSH Detection Panel](../assets/crowdsec_5.png)

Analyse :

- Pics temporels sur port 22
- Identification brute-force rapide vs lent
- Corrélation avec périodes horaires

👉 Outil clé pour décision de remédiation.

---

# 🛡️ Section 4 — Activation Remédiation (nftables Bouncer)

Après validation en mode observation, activation du bouncer firewall.

---

## 🧾 Création Allowlist Stratégique

Avant activation :

- IP administrateurs
- IP fournisseur VPS
- IP monitoring externe

```bash
sudo cscli allowlists create admin_ips
sudo cscli allowlists add admin_ips "X.X.X.X"
```

---

## 🔑 Création du Bouncer

```bash
sudo cscli bouncers add nftables-prod
```

---

## ✅ Vérification Enregistrement

```bash
sudo cscli bouncers list
```

![Bouncer Valid](../assets/crowdsec_6.png)

Statut attendu :

```
Valid
```

👉 Le moteur peut désormais appliquer les décisions.

---

## 🚫 Décisions Actives

```bash
sudo cscli decisions list
```

![Active Decisions](../assets/crowdsec_7.png)

On observe :

- IP bannies
- Scénario déclencheur
- Durée restante
- Type : ban

---

## ⚙️ Injection Kernel-Level (nftables)

```bash
sudo nft list table ip crowdsec
```

![nftables Injection](../assets/crowdsec_8.png)

Les IP apparaissent directement dans les sets nftables.

🎯 Blocage effectué :

- Au niveau noyau Linux
- Sans surcharge CPU
- Sans service intermédiaire

Performance optimale.

---

# ⚠️ Incident Réel — Gestion du Volume de Règles

Un volume excessif de décisions dynamiques a temporairement :

- augmenté la complexité des règles nftables
- généré une surcharge firewall

### Remédiation appliquée :

```bash
sudo cscli decisions delete --all
```

Effets :

- Réduction drastique des règles dynamiques
- Stabilisation firewall
- Maintien des détections futures

👉 Décision pragmatique orientée production.

---

# 🔄 Transition Finale

Une fois la remédiation validée :

```bash
sudo systemctl stop fail2ban
sudo systemctl disable fail2ban
sudo apt remove --purge fail2ban
```

CrowdSec remplace entièrement Fail2ban avec :

- Détection comportementale avancée
- Enrichissement réseau
- Observabilité complète
- Blocage haute performance

---

# ✅ Conclusion

Le VPS dispose désormais :

- ✔️ D’un HIPS comportemental
- ✔️ D’alertes temps réel enrichies
- ✔️ D’une visualisation SOC
- ✔️ D’un blocage kernel-level automatisé
- ✔️ D’une architecture scalable et maintenable

La couche Host IPS est opérationnelle.

Le système est prêt pour :

➡️ Phase 4 – NIPS avec Suricata (Inspection Deep Packet Inline)
