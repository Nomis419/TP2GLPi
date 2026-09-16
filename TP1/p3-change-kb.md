# Partie 3 — Traitement du changement (Change Enablement + Knowledge Management + Product and Service Lifecycle)

**Amélioration retenue :** Amélioration A de la Partie 1 — *automatiser la détection des événements physiques de câblage (connexion/déconnexion de port) et fiabiliser la CMDB NetBox.*

---

## 1. RFC — Request For Change

| Champ | Valeur |
|-------|--------|
| **ID** | RFC-2026-021 |
| **Titre** | Déploiement d'un système d'automatisation « détection d'événements câblage → mise à jour NetBox » avec workflow d'approbation |
| **Demandeur** | Simon — technicien réseaux & systèmes |
| **Approbateur** | Chef d'infrastructure réseau |
| **Date de dépôt** | 2026-09-15 |
| **Service impacté** | CMDB (NetBox) + supervision réseau |
| **Type de changement** | **Normal** (voir justification) |

### 1.1 Description du changement
Déployer, sur une **VM dédiée** (fournie par le chef d'infra), une chaîne d'automatisation : un récepteur de **traps SNMP** capte les événements de connexion/déconnexion de ports sur les switches, un module d'alerte notifie l'équipe infra, et un module de mise à jour synchronise **NetBox**. Point clé : plutôt qu'une mise à jour automatique, l'équipe **valide ou refuse** chaque modification via un lien e-mail **signé HMAC** (Accept / Refuse), avec délai d'expiration et journalisation complète.

Composants : `trap_receiver.py`, `alerting.py`, `netbox_updater.py`, `test_simulation.py` ; playbook Ansible `deploy_trap_receiver.yml` ; service systemd ; configuration par `.env`.

### 1.2 Classification du type — justification
- **Pas standard** : nouvelle capacité, non pré-approuvée, qui écrit dans la CMDB de référence.
- **Pas urgent** : aucune panne active ; déploiement planifiable.
- **➜ Normal** : touche un système de production (NetBox) et nécessite une **évaluation** (impact, tests, rollback) avant mise en service.

### 1.3 Analyse d'impact
**Qui est affecté :**
- L'équipe infra réseau (reçoit et arbitre les demandes d'approbation).
- La CMDB NetBox et tous ceux qui s'appuient dessus (documentation, IPAM, futurs projets NAC/Wi-Fi).
- Indirectement la production hospitalière : la VM et le récepteur de traps consomment des ressources et écoutent le réseau de management.

**Risques de régression :**
- Un mapping trap → objet NetBox erroné pourrait **écrire une mauvaise information** dans la CMDB (port/appareil incorrect).
- Un volume de traps élevé (tempête d'événements lors d'une maintenance) pourrait **inonder** l'équipe de mails d'approbation.
- Un secret HMAC ou un jeton mal géré exposerait les liens d'approbation (**enjeu sécurité**, contexte HDS/PGSSI-S/NIS2).

### 1.4 Plan de rollback (concret)
1. **Avant mise en service** : le système ayant été **développé et validé en simulation** (`test_simulation.py`) sans matériel réel, on conserve cet environnement de test comme référence.
2. Le service tournant sous **systemd** sur une VM dédiée : rollback immédiat = `systemctl stop` + `disable` du service → plus aucune écriture NetBox, l'équipe revient à la saisie manuelle (modes opératoires MO existants).
3. Le **workflow d'approbation est lui-même un garde-fou** : aucune écriture NetBox n'a lieu sans validation humaine, donc une anomalie ne corrompt pas la CMDB automatiquement.
4. **Sauvegarde NetBox (dump PostgreSQL)** horodatée avant activation ; en cas d'écriture erronée validée par erreur, restauration ciblée depuis le dump.
5. **Critère de déclenchement** : toute écriture erronée en CMDB non rattrapable par l'historique NetBox, ou saturation d'alertes ingérable.

### 1.5 Simulation CAB (Change Advisory Board)

**Demandeur (Simon) — pour :**
> « Le changement supprime une source de tickets liés à des ports/mouvements non documentés et fiabilise la CMDB, socle des projets à venir (NAC, refonte Wi-Fi). Le risque est maîtrisé : développement testé en simulation, service désactivable en une commande, et surtout **aucune écriture automatique** — l'humain valide chaque mise à jour. »

**Approbateur (chef d'infra réseau) — évaluation :**
> « Approbation **conditionnelle**. J'ai fourni la VM et je valide l'approche, mais j'impose : (1) SNMP en **v2** (auth + chiffrement) sur le réseau de management, pas de v2c ; (2) gestion propre du secret HMAC et des délais d'expiration des liens ; (3) mécanisme anti-tempête (regroupement/seuil) pour ne pas noyer l'équipe pendant les maintenances ; (4) documentation avec **placeholders** (aucune config réelle de l'établissement dans les livrables partagés). Sous ces conditions : validé. »

*(Cette interaction reflète le déroulé réel : projet accueilli positivement, VM dédiée proposée, et workflow d'approbation par mail suggéré par le chef en remplacement de la mise à jour automatique.)*

---

## 2. Article de base de connaissance (Knowledge Management)

**Titre : Équipement médical connecté injoignable par intermittence — désadaptation vitesse/duplex du port**

- **Symptôme :** un équipement médical connecté (ex. moniteur **Dräger**) perd la communication de façon intermittente ; le service concerné rappelle plusieurs fois pour « l'appareil qui se déconnecte ». Côté switch : compteurs d'erreurs élevés sur le port (**CRC**, **late collisions**, **OutDiscards** massifs).
- **Cause :** port forcé en **10 Mbps half-duplex** (souvent en dur côté équipement) alors que l'autre extrémité est en full/auto → **duplex mismatch**. Résultat : collisions tardives et trames rejetées, d'où des pertes intermittentes difficiles à reproduire.
- **Résolution :**
  1. Relever les compteurs sur le port (`show interface`) et établir une **baseline** fraîche.
  2. Confirmer le mismatch : vitesse/duplex des deux extrémités.
  3. **Aligner** la configuration (idéalement auto/auto des deux côtés, ou full/full explicite si l'équipement l'impose) — attention : un équipement médical n'est pas toujours modifiable côté appareil.
  4. **Corréler avec Zabbix** (événements, historique des erreurs) pour valider la disparition des compteurs après correction.
  5. Si l'équipement biomédical impose une contrainte non modifiable, **escalader vers l'équipe biomédicale** (statut « En attente tiers ») et documenter la contrainte.
  6. Vérifier la stabilité, **mettre à jour NetBox**, puis clôturer **après confirmation** du service.
- **Mots-clés :** duplex mismatch, half-duplex, CRC, late collisions, OutDiscards, Dräger, biomédical, port switch, intermittent.

> Cet article traite une cause classique de « rappels pour un même problème » (Partie 1) : une panne intermittente clôturée trop tôt faute de traitement de la cause racine.

---

## 3. Positionnement dans le Product and Service Lifecycle

> Le **Product and Service Lifecycle** (8 étapes : Discover, Design, Acquire, Build, Transition, Operate, Deliver, Support) remplace la **Service Value Chain** d'ITIL 4 (6 activités : Plan, Improve, Engage, Design & Transition, Obtain/Build, Deliver & Support). À ne pas confondre.

### Étapes mobilisées par la RFC-2026-021
- **Discover / Design (amont)** : le chef a fait évoluer le besoin — passer d'une mise à jour automatique à un **workflow d'approbation** relève de la (re)conception de la solution.
- **Acquire** : mise à disposition de la **VM dédiée** et de la collection Ansible `huawei.ce`.
- **Build (principale)** : développement et **test en simulation** des scripts et du playbook avant matériel réel.
- **Transition (principale)** : déploiement maîtrisé sur la VM, avec rollback prévu (service systemd désactivable).
- *En aval* : **Operate / Support** (le système vit en production) et **Deliver** (la CMDB fiable délivre sa valeur aux projets NAC/Wi-Fi).

### Pourquoi ce n'est pas un enchaînement strictement linéaire
ITIL 5 présente ces activités comme des « stepping stones » combinées en *value streams*, avec boucles et sauts. Ici, le **feedback du chef** (Transition/exploitation anticipée) a renvoyé vers **Design** (ajout de l'approbation), et les tests en simulation (**Build**) ont eu lieu **avant** l'acquisition du matériel réel (**Acquire** partiellement en attente) — preuve concrète que les étapes se chevauchent au lieu de se suivre en ligne droite.
