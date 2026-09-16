# TP ITIL 5 — Amélioration du support infrastructure réseau (CHOR)

Dépôt : `tp-itil5-simon`
Auteur : Simon — TSRS — technicien réseaux & systèmes, outil NetBox
Cadre : ITIL 5 (PeopleCert, publié le 12 février 2026)

Ce dépôt applique ITIL 5 à un cas réel : le support / infrastructure réseau N2, qui reçoit les escalades du helpdesk N1 et de l'équipe câblage. Symptômes traités : lenteur, tickets perdus, rappels répétés.

> **Note de confidentialité.** Conformément à une approche *placeholder-first* et au contexte HDS/PGSSI-S/NIS2, ce dépôt ne contient **aucune configuration réelle** de l'établissement (pas de plages IP, de community SNMP, ni de noms/ID de VLAN internes). Les familles d'équipements et les récits de diagnostic sont conservés à titre illustratif.

## Livrables
| Fichier | Contenu |
|---------|---------|
| `p1-csi-register.md` | Diagnostic 4 dimensions + CSI Register priorisé + principe directeur |
| `p2-slm-events.md` | 2 SLA + SLO, classification des 5 logs (Event Management) |
| `p3-change-kb.md` | RFC (automatisation NetBox), article KB, positionnement Lifecycle |
| `logs.txt` | Ressource fournie (5 événements) |
| `README.md` | Demande de service GLPI + synthèse (ce fichier) |

---

## Partie 4 — Demande de service (Service Request Management)

> **Pourquoi une demande de service et non un Incident :** il s'agit d'une demande **planifiée** liée au changement (préparer l'environnement du déploiement), pas du rétablissement d'une panne. Elle relève de **Service Request Management**.

### Champs clés du ticket GLPI (copiés depuis l'outil, sans capture)

```
Titre        : [DEMANDE] Provisionnement VM dédiée Ansible/supervision (suite RFC-2026-021)
Type         : Demande
Catégorie    : Infrastructure / Virtualisation
Demandeur    : Simon 
Validé par   : Chef d'infrastructure réseau
Priorité     : P3 — Mineur (planifiée, non bloquante)
Statut       : Résolu → en attente de confirmation
Date ouverture   : 2026-09-16 09:00
Date d'échéance  : 2026-09-19 18:00
Date de résolution : 2026-09-18 16:00

Description :
Créer et livrer une VM dédiée (Linux + systemd) pour héberger le récepteur
de traps SNMP et le playbook Ansible du projet d'automatisation NetBox.
Accès SNMP v3 vers le VLAN de management à confirmer avec l'équipe sécurité.

Tâches :
- [x] Création VM + OS + durcissement de base
- [x] Ouverture de flux SNMP v3 vers le périmètre de management (règle pare-feu)
- [x] Installation dépendances (Ansible, collection huawei.ce, Python)
- [ ] Confirmation du demandeur → clôture définitive

Suivi (dernier) :
2026-09-18 16:00 — VM livrée et jointe au domaine, flux SNMP v3 validé en test.
Demande de confirmation envoyée avant clôture.
```

---

## Tableau récapitulatif — parties ↔ pratiques ITIL 5

| Partie | Pratique(s) ITIL 5 mobilisée(s) |
|--------|----------------------------------|
| Partie 1 | **Continual Improvement** (+ cadre des *Four Dimensions*) |
| Partie 2 | **Service Level Management** · **Monitoring and Event Management** |
| Partie 3 | **Change Enablement** · **Knowledge Management** (+ *Product and Service Lifecycle*) |
| Partie 4 | **Service Request Management** (+ synthèse) |

---

## Principe directeur le plus structurant sur l'ensemble du cas

**« Se concentrer sur la valeur » (Focus on value).**

**Exemple concret du TP :** chaque décision est ancrée sur la réduction du risque et de la douleur réelle, pas sur la technique pour elle-même. Le choix marquant est en Partie 3 : le passage d'une **mise à jour automatique** de la CMDB à un **workflow d'approbation humaine**. Techniquement, l'auto-update était plus « élégant » ; mais dans un hôpital, la valeur, c'est une CMDB **fiable et maîtrisée** — on a donc accepté une friction (validation manuelle) parce qu'elle protège la donnée de référence et l'équipe. La valeur pour le service prime sur la sophistication.

*(À distinguer de la Partie 1, où c'est la **priorisation** qui suit « Progresser itérativement avec du feedback » : le fil rouge d'ensemble reste « Focus on value ».)*

---

## Point critique — apport du module AI Governance et du modèle 6C

**Sur la solution livrée : pas d'IA, donc aucune obligation de gouvernance IA déclenchée — et c'est volontaire.** Le système d'automatisation repose sur des **règles déterministes** (trap SNMP → mapping → proposition de mise à jour), pas sur de l'IA. Imposer de l'IA ici serait contraire à « Rester simple et pratique ». Non-pertinence assumée, pas un oubli.

**Mais le cas est déjà, sans le savoir, une bonne illustration de gouvernance.** Le choix du chef — **ne pas** laisser la machine écrire seule dans la CMDB, et exiger une validation humaine par lien signé — est exactement une **decision boundary** avec **human-in-the-loop**. C'est le réflexe que le module AI Governance systématise.

**Où le modèle 6C (ITIL AI Capability Model) s'appliquerait concrètement (évolution future).** Si on ajoutait de l'IA à la supervision, on nommerait précisément ce qu'elle fait, chaque capacité appelant un niveau de contrôle différent :
- **Cognition** — prédire une défaillance à partir des tendances Zabbix (ex. dérive des compteurs d'erreurs annonçant un duplex mismatch) : faible risque décisionnel, l'humain agit.
- **Clarification** — résumer/router automatiquement un ticket d'incident réseau : suggestion à faible impact.
- **Coordination** — **écrire seule dans NetBox** ou **désactiver un port** (err-disable) sans humain : **fort impact décisionnel** → dans un hôpital (sécurité patient, HDS/PGSSI-S/NIS2), cette capacité **doit** rester derrière une validation humaine, exactement comme le workflow d'approbation actuel.

**Conclusion de gouvernance :** l'apport réel du 6C n'est pas « l'IA c'est important », c'est qu'il **gradue le contrôle selon la capacité** — une prédiction (Cognition) demande peu de garde-fous, une action autonome (Coordination) en exige beaucoup. Mon projet applique déjà ce principe pour de l'automatisation classique ; il serait directement réutilisable si une brique IA était introduite.

---

## Journal des commits (2 minimum par partie)

```
p1: constats 4 dimensions ancrés CHOR + ouverture CSI Register
p1: priorisation CSI + principe directeur
p2: 2 SLA + SLO par priorité (contexte hospitalier)
p2: classification Event Management des 5 logs + actions Zabbix/SNMP
p3: RFC-2026-021 automatisation NetBox (type, impact, rollback, CAB)
p3: article KB duplex mismatch + positionnement Lifecycle
p4: demande de service GLPI (provisionnement VM)
p4: README synthèse (tableau, principe, point AI Governance/6C)
```
