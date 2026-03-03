# 🔥 Phase 4 – IPS Réseau avec Suricata (IDS ➜ NIPS Inline)

## 📝 Résumé Exécutif

Après la mise en place :

- D’un IPS hôte comportemental (CrowdSec)
- D’une stack complète d’observabilité (LGMA)

Cette phase introduit un **Network Intrusion Prevention System (NIPS)** basé sur Suricata en mode inline.

### Stratégie de déploiement :

1. Déploiement initial en mode IDS (détection seule)
2. Validation des logs et de la visibilité
3. Établissement d’une baseline du trafic
4. Passage en mode inline via NFQUEUE
5. Activation de règles DROP contrôlées
6. Mise en place d’un système de mise à jour sans interruption

### Objectifs :

- Bloquer les scans réseau réels (Nmap, NULL, XMAS)
- Ne pas impacter SSH / HTTP / HTTPS / IPv6
- Conserver une observabilité complète
- Garantir un rollback sécurisé

---

# 🔎 Section 1 — Suricata en mode IDS (Détection)

## Validation de la configuration

Avant le démarrage du service, l’intégrité de la configuration est validée :

```bash
sudo suricata -T
```

![Test Configuration Suricata](../assets/network_1.png)

✔ Configuration chargée avec succès  
✔ Aucune erreur critique  
✔ Validation adaptée à un environnement de production  

Cela garantit un démarrage propre de Suricata.

---

## Démarrage du service

```bash
sudo systemctl start suricata
sudo systemctl status suricata
```

![Service Suricata Actif](../assets/network_2.png)

✔ Service actif via systemd  
✔ Aucun redémarrage en boucle  
✔ Gestion correcte du PID  

Le mode IDS est désormais opérationnel.

---

# 📡 Section 2 — Inspection du trafic & Intégration des logs

Suricata inspecte directement le trafic sur l’interface publique.

Exemple d’événement TLS capturé dans `eve.json` :

![Événement TLS Capturé](../assets/network_3.png)

Champs observés :

- IP source
- IP destination
- Version TLS
- Nom SNI
- Capture directe sur l’interface réseau

✔ Deep Packet Inspection confirmé  
✔ Visibilité complète des métadonnées des sessions chiffrées  

Les logs sont ensuite envoyés vers Loki via Alloy pour analyse centralisée.

---

# 📊 Section 3 — Baseline IDS (Avant le mode Inline)

Dashboard Grafana en mode IDS :

![Dashboard IDS Initial](../assets/network_4.png)

Observations :

- Volume d’alertes élevé
- Bruit provenant des ports non exposés
- Tentatives de reconnaissance pleinement visibles

✔ Détection fonctionnelle  
✔ Base de référence avant activation du NIPS  

---

# 🔥 Section 4 — Transition vers le NIPS (Mode Inline)

## Principe d’architecture

- UFW → Exposition des ports
- nftables → Sélection du trafic à inspecter
- Suricata → Moteur décisionnel (DROP)

Suricata n’inspecte pas tout le trafic, uniquement celui redirigé via NFQUEUE.

---

## Redirection du trafic via nftables

```bash
sudo nft list table inet suricata_nips
```

![Table NFQUEUE nftables](../assets/network_5.png)

Éléments clés :

- Hook : input
- Priority : -150
- Connexions établies acceptées
- SSH explicitement autorisé
- IPv6 exclu du NIPS
- HTTP/HTTPS (IPv4, état NEW) envoyés vers NFQUEUE

✔ Surface d’inspection minimale  
✔ Optimisation des performances  
✔ Whitelist explicite avant inspection  

---

## Règles DROP locales (Production Safe)

Seules des signatures à très forte confiance sont activées :

- TCP NULL scan
- TCP XMAS scan

Objectif :

- Aucun faux positif
- Blocage immédiat des scans automatisés
- Aucun impact sur le trafic légitime

---

# 🚫 Section 5 — Vérification du DROP effectif

Observation en temps réel confirmant l’exécution des DROP :

![Confirmation DROP](../assets/network_6.png)

✔ Action DROP visible dans les logs  
✔ Blocage réel des paquets  
✔ Enforcement inline actif  

Suricata fonctionne désormais comme un véritable IPS réseau.

---

# 📉 Section 6 — Impact sur la télémétrie (Après NIPS)

Dashboard Grafana après activation inline :

![Dashboard NIPS Nettoyé](../assets/network_7.png)

Améliorations constatées :

- Réduction drastique du bruit
- Scans sur ports non exposés bloqués avant journalisation
- Signal de sécurité plus lisible
- Charge d’ingestion réduite

✔ Meilleure lisibilité  
✔ Moins de surcharge système  
✔ Données plus exploitables  

Ce comportement est attendu et recherché en production.

---

# ⚙️ Section 7 — Pérennisation en mode service

Suricata est configuré définitivement en mode NFQUEUE.

Vérification :

```bash
ps aux | grep suricata
```

![Suricata Mode NFQUEUE](../assets/network_8.png)

Confirmation :

- Processus unique
- Mode nfqueue actif
- Gestion via systemd
- Intégration complète avec nftables

✔ Démarrage automatique  
✔ Enforcement inline persistant  
✔ Aucun lancement manuel requis  

---

# 🔄 Section 8 — Mise à jour des règles sans interruption

Stratégie de mise à jour sécurisée :

1. Téléchargement des nouvelles signatures
2. Validation via `suricata -T`
3. Application via `systemctl reload suricata`
4. Conservation des anciennes règles en cas d’échec

Pourquoi `reload` et non `restart` ?

En mode inline (Fail-Closed) :

- Un restart coupe immédiatement le trafic.
- Un reload maintient la NFQUEUE active.

✔ Validation préalable  
✔ Rechargement à chaud  
✔ Aucune interruption de service  

---

# ✅ État final de l’architecture

Le VPS dispose désormais :

- ✔ D’un IPS hôte (CrowdSec)
- ✔ D’un IPS réseau inline (Suricata + NFQUEUE)
- ✔ D’un enforcement kernel-level via nftables
- ✔ De mises à jour sans interruption
- ✔ D’une observabilité complète via Loki & Grafana
- ✔ De règles DROP minimales et contrôlées
- ✔ D’une capacité de rollback sécurisé

Suricata est utilisé comme un moteur décisionnel ciblé, et non comme un simple pare-feu générique.

---

# 🚀 Prêt pour la Phase 5 — Threat Hunting & Mapping MITRE ATT&CK
