# Partie 2 — Pilotage du service (Service Level Management + Event Management)

## 0. Échelle de priorité de référence (contexte hospitalier)

| Priorité | Définition | Exemple CHOR |
|----------|-----------|--------------|
| **P1 — Critique** | Impact sur un soin ou un service clinique, sans contournement. | Équipement médical connecté isolé du réseau, téléphonie SIP d'un service de soins HS. |
| **P2 — Majeur** | Fonction dégradée ou un poste bloqué avec contournement. | Poste de travail lent (mismatch duplex), imprimante d'un bureau HS. |
| **P3 — Mineur** | Gêne ou demande non bloquante. | Question d'usage, déclaration d'un nouvel équipement. |

**Base de mesure :** heures ouvrées, temps décompté aux statuts « En attente utilisateur » et « En attente tiers » (ex. escalade biomédicale). Les événements sont issus de **Zabbix / SNMP / Netdisco**.

---

## 1. SLA + SLO

> **SLA** = engagement formel sur le niveau de service. **SLO** = objectif mesurable qui compose le SLA.

### SLA 1 — Délai de première réponse (Time to First Response)

| Priorité | Engagement (SLA) |
|----------|------------------|
| P1 | ≤ **1 h** |
| P2 | ≤ **4 h** |
| P3 | ≤ **1 jour ouvré** |

**SLO associé :** *≥ 95 % des tickets P1 pris en charge en moins d'1 h, mesuré sur 30 jours glissants.*

### SLA 2 — Délai de résolution (Time to Resolution)

| Priorité | Engagement (SLA) |
|----------|------------------|
| P1 | ≤ **4 h ouvrées** |
| P2 | ≤ **1 jour ouvré** |
| P3 | ≤ **3 jours ouvrés** |

**SLO associé :** *≥ 90 % des P1 résolus < 4 h ouvrées et ≥ 95 % des P2 < 1 jour ouvré, sur 30 jours.*

> Lien Partie 1 : ces SLA ne sont fiables que si la supervision couvre réellement le parc (amélioration B) — un équipement absent de Zabbix/Netdisco fausse le décompte.

---

## 2. Classification des logs (pratique Event Management)

> **Informational** = normal, aucune action · **Warning** = seuil approché, action préventive · **Exception** = anomalie avérée, action immédiate (souvent un Incident).

| # | Log (résumé) | Type | Justification | Action (ancrée outillage CHOR) |
|---|--------------|------|---------------|--------------------------------|
| 1 | `AUTH jdupont login success WKS-042` | **Informational** | Connexion réussie = nominal. | Aucune. Journalisation conservée (audit, contexte HDS/PGSSI-S). |
| 2 | `DISK SRV-FILE01 usage=82% threshold=80%` | **Warning** | Seuil d'alerte (80 %) franchi (82 %), serveur encore opérationnel. | **Préventif :** créer une tâche Zabbix de suivi de tendance, planifier nettoyage/extension de volume, définir un seuil critique (ex. 90 %) déclenchant un Incident. |
| 3 | `SVC helpdesk-portal unreachable duration=00:04:12` | **Exception** | Service central indisponible 4 min 12 s = anomalie à fort impact. | **Immédiat :** Incident **P1**, vérifier l'item Zabbix du service, isoler la cause (applicatif, dépendance, réseau), restaurer, puis analyse post-incident. |
| 4 | `BACKUP nightly-backup SRV-DB01 completed 45GB` | **Informational** | Sauvegarde réussie = attendu. | Aucune. Contrôle de cohérence de taille possible en routine. |
| 5 | `NET switch-3F-port12 down flapping=true count=6/10min` | **Exception** | Port instable (6 bascules / 10 min) = défaut actif, coupures intermittentes, risque de recalcul STP. | **Immédiat :** Incident, corréler avec Zabbix/Netdisco, identifier le port et l'équipement en bout, vérifier câble/SFP et duplex, désactiver le port (err-disable) pour stabiliser, remplacer l'élément défectueux, réactiver après contrôle, **mettre à jour NetBox**. |

**Point d'attention supervision (vécu) :** si un événement attendu **n'apparaît pas** dans Netdisco/Zabbix, vérifier d'abord que l'équipement est bien managé — SNMP joignable et **aucune règle de pare-feu ne bloque le serveur de supervision vers le VLAN de management**. Un angle mort de supervision est lui-même un risque (un vrai incident qui ne génère aucun événement).

**Actions concrètes exigées (Warning + Exception) :** logs **2, 3, 5** — voir colonne « Action ». Les logs 3 et 5 ouvrent un Incident ; le log 2 déclenche une action préventive avant bascule en critique.
