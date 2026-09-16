# TP ITIL / GLPI — « Une journée chez NORTEK SI »

Traitement complet d'une journée type de support IT chez **NORTEK SI** (200 salariés, DSI de 4 personnes), appliqué dans **GLPI** en respectant les bonnes pratiques **ITIL** : gestion des incidents, des problèmes, des changements, des configurations (CMDB) et des demandes de service.

> Cas d'école pédagogique — l'entreprise NORTEK SI est fictive.

## Contenu du dépôt

| Fichier | Rôle |
|---------|------|
| [`procedure-glpi-nortek.md`](./procedure-glpi-nortek.md) | Mode opératoire pas à pas : ce qu'il faut créer et lier dans GLPI, phase par phase, avec tous les champs et les justifications. |
| [`recapitulatif-nortek.md`](./recapitulatif-nortek.md) | Compte rendu à rendre : un paragraphe justifié par phase, **avec les captures d'écran** des manipulations. |
| [`images/`](./images) | Captures d'écran GLPI référencées par le compte rendu (`etape0-sla.png`, `phase1-tickets.png`, …). |

## Les 5 phases → pratiques ITIL

| Phase | Sujet | Pratique ITIL | Objet GLPI |
|-------|-------|---------------|------------|
| 1 | 5 tickets à prioriser | Gestion des incidents | Tickets (Incident) |
| 2 | 3e panne mail récurrente | Gestion des problèmes | Problème + KEDB |
| 3 | Correctif du serveur mail | Gestion des changements | Changement (RFC + CAB) |
| 4 | Vision d'impact avant intervention | Gestion des configurations | CI / CMDB |
| 5 | Onboarding IT (Julie) | Gestion des demandes | Ticket (Demande) |

## Fil rouge (traçabilité GLPI)

```
3 incidents "serveur mail HS"
        |  (liés au)
        v
   Problème #2001 --(KEDB)--> Base de connaissances
        |  (résolu par)
        v
  Changement #3001 (RFC + CAB)
        |  (porte sur le CI)
        v
   CI SRV-MAIL01 --> CI dépendants (AD, sauvegarde, réseau, 200 postes)

Demande de service #4001 (onboarding Julie) — fil parallèle indépendant
```

## Grille d'évaluation visée (/20)

- Phase 1 — priorisation justifiée … /4
- Phase 2 — lien incidents / cause racine / KEDB … /4
- Phase 3 — RFC complète + traçabilité … /4
- Phase 4 — relations CI + analyse d'impact … /4
- Phase 5 — distinction incident/demande + SLA … /3
- Cohérence globale du fil rouge … /1

## Comment utiliser ce dépôt

1. Lire `procedure-glpi-nortek.md` et l'exécuter dans GLPI dans l'ordre (Étape 0 → Phase 5).
2. Vérifier les 6 liens du tableau final (fil rouge).
3. Prendre les captures d'écran des manipulations, les déposer dans `images/` avec les noms attendus, puis rendre `recapitulatif-nortek.md`.

## Captures d'écran attendues (dossier `images/`)

| Fichier | Ce qu'il doit montrer |
|---------|------------------------|
| `etape0-sla.png` | Les 4 SLA (TTO/TTR) + le calendrier heures ouvrées |
| `phase1-tickets.png` | Les 5 tickets triés par priorité (n°3 en « Sécurité / Hameçonnage ») |
| `phase2-probleme-liens.png` | Le Problème + ses 3 incidents liés (onglet « Tickets ») |
| `phase2-kedb.png` | L'article KEDB |
| `phase3-changement-liens.png` | Le Changement lié au Problème et aux incidents |
| `phase4-impact.png` | Le graphe d'analyse d'impact de SRV-MAIL01 |
| `phase5-demande.png` | La fiche Demande (Type, SLA demande, date cible, tâches) |

> Crée le dossier `images/` à la racine du dépôt. Les emplacements dans le compte rendu sont déjà positionnés (balises `![...](images/...)`) — il suffit de déposer les PNG aux bons noms.
