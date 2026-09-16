# Procédure GLPI — « Une journée chez NORTEK SI »

Mode opératoire à dérouler **dans l'ordre**. Chaque phase indique le chemin GLPI, les champs à recopier, les liens à créer et la justification.

> Les libellés de menus peuvent varier légèrement selon la version de GLPI ; adapte si besoin. Les numéros de tickets (#10xx, etc.) sont donnés à titre de repère — utilise ceux que GLPI génère.

---

## Étape 0 — Préparation (5 min, une seule fois)

Avant de créer les tickets, mets en place le référentiel pour que les champs existent.

### 0.1 Catégories ITIL
*Configuration > Intitulés > Catégories ITIL* — créer : `Messagerie`, `Matériel / Périphériques`, `Sécurité / Hameçonnage`, `Impression`, `Applications métier`, `Onboarding`.

### 0.2 Configuration des SLA

Le modèle GLPI est : un **SLM** (conteneur « Niveau de service ») contient des **SLA**, chaque SLA étant soit un **TTO** (temps de prise en compte) soit un **TTR** (temps de résolution), rattaché à un **calendrier**.

**a) Calendrier (heures ouvrées)** — *Configuration > Intitulés > Calendriers > Ajouter*
- Nom : `Heures ouvrées NORTEK`
- Onglet **Plages horaires** : une plage par jour, **lundi → vendredi, 08:00–18:00**.
- Onglet **Périodes de fermeture** : ajouter les jours fériés.

> Choix assumé : DSI de 4 personnes **sans astreinte** → SLA décomptés en **heures ouvrées**, pas en 24/7. Sinon un P4 ouvert vendredi 17h « consommerait » son délai le week-end et serait faussement dépassé le lundi.

**b) Conteneur SLM** — *Configuration > Niveaux de services > Ajouter*
- Nom : `SLM NORTEK` · Calendrier : `Heures ouvrées NORTEK`
- Les SLA se créent **dans** ce SLM (onglet SLA/OLA).

**c) Les 8 SLA (TTO + TTR par priorité)** — pour chacun : Nom, Type, Durée, Calendrier `Heures ouvrées NORTEK`.

| Nom | Type | Durée à saisir | équivaut à |
|-----|------|----------------|-----------|
| `P1 - TTO` | Prise en compte | **15 minutes** | 15 min |
| `P1 - TTR` | Résolution | **4 heures** | 4 h |
| `P2 - TTO` | Prise en compte | **30 minutes** | 30 min |
| `P2 - TTR` | Résolution | **8 heures** | 1 j ouvré |
| `P3 - TTO` | Prise en compte | **2 heures** | 2 h |
| `P3 - TTR` | Résolution | **16 heures** | 2 j ouvrés |
| `P4 - TTO` | Prise en compte | **4 heures** | 4 h |
| `P4 - TTR` | Résolution | **40 heures** | 5 j ouvrés |

> **Astuce :** saisir les délais longs (P3/P4) **en heures** (16 h, 40 h), pas en « jours » — avec un calendrier ouvré, l'unité « jour » est ambiguë selon les versions ; en heures, GLPI ne décompte que les plages ouvertes (16 h = 2 journées de 8 h).

**d) Escalade (optionnel, valorisé)** — sur `P1 - TTR`, onglet **Niveaux d'escalade** :
- à **-30 min** de l'échéance → action *Envoyer une notification* au responsable DSI.
- à **l'échéance (0 min)** → action *Notifier le groupe support + escalader*.

**e) Affectation priorité → SLA** — *Administration > Règles > Règles métier pour les tickets* : une règle par priorité.
- Critère `Priorité` = `Majeure/Très haute` → Actions : SLA (prise en compte) = `P1 - TTO` **et** SLA (résolution) = `P1 - TTR`.
- Répéter : Haute → P2, Moyenne → P3, Basse/Très basse → P4.
- Alternative pour le TP : sélection **manuelle** du TTO/TTR sur chaque ticket (bloc SLA).

**f) Vérification** — ouvrir un ticket de test priorité `Majeure` : `P1 - TTO 15 min` / `P1 - TTR 4 h` doivent s'appliquer, avec une **date d'échéance** calculée sur le calendrier ouvré. Un P4 ouvert vendredi 17h doit échoir le **vendredi suivant**, pas pendant le week-end.

> Rappel : la **matrice Impact × Urgence → Priorité** se règle ailleurs (*Configuration > Générale > onglet Assistance*). Les SLA ne font que réagir à la priorité obtenue.

---

## Phase 1 — Gestion des incidents

### La matrice de priorité (rappel)
Dans GLPI, la **Priorité** se calcule automatiquement à partir de **Impact × Urgence**. Impact = « combien de monde / quelle criticité », Urgence = « à quelle vitesse ça doit être traité ».

### Tableau de synthèse (à respecter pour l'ordre de traitement)

| # | Ticket | Type | Impact | Urgence | **Priorité** | SLA | Ordre |
|---|--------|------|--------|---------|--------------|-----|:-----:|
| 1 | Serveur messagerie HS (toute l'entreprise) | Incident | Très haute | Haute | **Majeure** | P1 | **1er** |
| 3 | Email suspect / demande d'identifiants | Incident **sécurité** | Haute | Haute | **Haute** | P2 | **2e** |
| 5 | DAF (VIP) — tableau de bord financier KO | Incident | Moyenne | Haute | **Haute** | P2 | **3e** |
| 4 | Imprimante 2e étage HS | Incident | Moyenne | Moyenne | **Moyenne** | P3 | **4e** |
| 2 | Souris comptabilité HS | Incident | Très basse | Basse | **Très basse** | P4 | **5e** |

> **Justification de l'ordre :** on traite par priorité décroissante, pas par ordre d'arrivée. Les tickets 3 et 5 sont tous deux « Haute » : on passe le **3 avant le 5** parce qu'un incident de sécurité a un impact qui **croît avec le temps** (propagation, vol d'identifiants), alors que le tableau du DAF est une dégradation stable limitée à un utilisateur.

### Chemin GLPI
*Assistance > Tickets > (+) Ajouter*, pour chacun des 5 tickets.

---

#### Ticket #1001 — Serveur de messagerie ne répond plus
```
Type         : Incident
Catégorie    : Messagerie
Titre        : Serveur de messagerie injoignable - toute l'entreprise
Demandeur    : (utilisateur ayant signalé) / Source : plusieurs signalements
Impact       : Très haute        Urgence : Haute
Priorité     : Majeure  (auto)   SLA : P1 (TTO 15 min / TTR 4 h)
Statut       : En cours (attribué)
Description  : Plus aucun envoi/réception d'email pour l'ensemble des 200
               utilisateurs. Webmail inaccessible. Confirmé sur SRV-MAIL01.
```
**Justification priorité :** service critique + périmètre = toute l'entreprise → impact maximal, traité en premier.
**Traitement :** vérifier l'état du service et de SRV-MAIL01 (CPU/RAM/disque, files SMTP), rétablir le service. ⚠️ Ce ticket sera **rattaché au Problème** en Phase 2 (3e occurrence).

#### Ticket #1003 — Email suspect demandant des identifiants
```
Type         : Incident (incident de sécurité)
Catégorie    : Sécurité / Hameçonnage
Titre        : Tentative d'hameçonnage - demande d'identifiants
Demandeur    : (collaborateur ayant signalé)
Impact       : Haute             Urgence : Haute
Priorité     : Haute   (auto)    SLA : P2 (TTO 30 min / TTR 8 h)
Statut       : En cours
Description  : Un collaborateur signale un email demandant ses identifiants.
               Suspicion de phishing. Vérifier étendue (campagne ?).
```
**Qualification (le piège) :** ce **n'est pas** un incident technique classique ni une demande — c'est un **incident de sécurité (hameçonnage)**. On ne « répare » rien sur un poste : on **contient**.
**Traitement (procédure sécurité) :**
1. Répondre à l'utilisateur : **ne pas cliquer, ne pas répondre, ne pas saisir d'identifiants** ; transférer l'email en pièce jointe pour analyse (en-têtes).
2. Vérifier si **d'autres utilisateurs** ont reçu le même message → si oui, campagne en cours.
3. **Bloquer** l'expéditeur / l'URL sur la passerelle de messagerie ; **alerter l'ensemble du personnel** (message de sensibilisation).
4. Si des identifiants **ont été saisis** → réinitialiser le mot de passe immédiatement, forcer la MFA → l'urgence passe à **Très haute**.
5. Tracer comme incident de sécurité.

#### Ticket #1005 — DAF (VIP) : tableau de bord financier inaccessible
```
Type         : Incident
Catégorie    : Applications métier
Titre        : DAF (VIP) - tableau de bord financier ne s'ouvre plus
Demandeur    : DAF   → cocher le flag VIP
Impact       : Moyenne           Urgence : Haute
Priorité     : Haute   (auto)    SLA : P2 (TTO 30 min / TTR 8 h)
Statut       : En cours
Description  : Le DAF n'accède plus à son tableau de bord financier. Un seul
               utilisateur mais fonction clé + statut VIP.
```
**Justification priorité :** un seul utilisateur (impact modéré) mais **VIP** + fonction sensible → GLPI remonte la priorité via le flag VIP. Traité 3e.
**Traitement :** vérifier droits d'accès, session, connectivité à l'application/base financière, cache navigateur.

#### Ticket #1004 — Imprimante 2e étage HS
```
Type         : Incident
Catégorie    : Impression
Titre        : Imprimante 2e etage hors service
Impact       : Moyenne           Urgence : Moyenne
Priorité     : Moyenne (auto)    SLA : P3 (TTO 2 h / TTR 2 j)
Statut       : En cours
Description  : Imprimante du 2e étage HS. Plusieurs utilisateurs concernés
               mais contournement possible (imprimer sur un autre étage).
```
**Justification priorité :** un étage impacté mais **contournement** disponible → priorité moyenne.

#### Ticket #1002 — Souris comptabilité HS
```
Type         : Incident
Catégorie    : Matériel / Périphériques
Titre        : Souris HS - service comptabilite
Impact       : Très basse        Urgence : Basse
Priorité     : Très basse (auto) SLA : P4 (TTO 4 h / TTR 5 j)
Statut       : En cours
Description  : Souris non fonctionnelle, 1 utilisateur, contournement immédiat
               (souris de rechange).
```
**Justification priorité :** 1 utilisateur, remplacement trivial → priorité la plus basse, traité en dernier.

**Livrable Phase 1 :** les 5 tickets créés + la colonne « justification » du tableau de synthèse.

---

## Phase 2 — Gestion des problèmes

### Chemin GLPI
*Assistance > Problèmes > (+) Ajouter*.

#### Fiche Problème #2001
```
Titre        : Instabilite recurrente du serveur de messagerie (SRV-MAIL01)
Catégorie    : Messagerie
Impact       : Très haute     Priorité : Haute
Statut       : En cours (analyse)
Description  : 3e incident similaire en 2 semaines (lenteur puis coupure du
               serveur mail). Recherche de cause racine.
```
**Lier les incidents :** onglet **« Tickets »** du Problème → rattacher les **3 incidents** de coupure mail (dont #1001 de la Phase 1 + les 2 précédents de l'historique).

### Analyse de cause racine — méthode des 5 Pourquoi
1. **Pourquoi** le serveur mail tombe-t-il ? → Le service se fige après une montée en charge disque.
2. **Pourquoi** la charge disque monte-t-elle ? → La partition des journaux/bases de la messagerie se remplit à 100 %.
3. **Pourquoi** se remplit-elle ? → Les journaux de transactions ne sont plus purgés automatiquement.
4. **Pourquoi** ne sont-ils plus purgés ? → La tâche de maintenance de purge échoue silencieusement depuis la dernière montée de version.
5. **Pourquoi** échoue-t-elle ? → **Cause racine :** un défaut connu de la version installée du logiciel de messagerie (gestion des journaux) → **correctif fournisseur requis**.

### Workaround immédiat (en attendant le correctif)
- Purge manuelle des journaux + extension temporaire de la partition.
- Redémarrage planifié du service hors heures de pointe.
- **Supervision de l'espace disque** avec alerte à 80 % pour intervenir avant la coupure.
→ Objectif : **plus de coupure** le temps de planifier le changement (Phase 3).

### Entrée KEDB (base de connaissances GLPI)
*Outils > Base de connaissances > Ajouter* (puis lier au Problème via l'onglet « Base de connaissances ») :
```
Titre    : [Erreur connue] SRV-MAIL01 - saturation disque par non-purge des journaux
Symptôme : Lenteur puis coupure du serveur de messagerie ; partition journaux à 100 %.
Cause connue : Défaut de la version logicielle -> tâche de purge des journaux
               inopérante depuis la mise à jour.
Contournement : Purge manuelle + extension partition + supervision disque à 80 %.
Solution définitive : Application du correctif fournisseur (voir Changement #3001).
Mots-clés : messagerie, SRV-MAIL01, saturation disque, journaux, purge, erreur connue
```

**Livrable Phase 2 :** Problème #2001 + 3 incidents liés + 5 pourquoi + workaround + entrée KEDB.

---

## Phase 3 — Gestion des changements

### Chemin GLPI
*Assistance > Changements > (+) Ajouter*.

### RFC — Request For Change #3001
```
Titre       : Application du correctif fournisseur sur SRV-MAIL01
Catégorie   : Messagerie
Type de changement : Normal (significatif) -> nécessite validation CAB
Impact / Priorité : Haute
Fenêtre de maintenance proposée : mercredi 20h00-22h00 (hors heures ouvrées)
```
**Description :** appliquer le correctif livré par le fournisseur pour rétablir la purge automatique des journaux et supprimer la cause racine du Problème #2001.

**Risques :**
- Échec ou régression du correctif → indisponibilité prolongée de la messagerie.
- Incompatibilité avec la configuration en place.

**Impact :** interruption **planifiée** de la messagerie pendant la fenêtre → les 200 utilisateurs sont prévenus ; les emails entrants sont mis en file (pas de perte, remise différée).

**Plan de rollback :**
1. **Snapshot** de la VM SRV-MAIL01 + **sauvegarde** des bases avant intervention.
2. Point de non-retour fixé à 21h30 : si le service n'est pas nominal, **restauration du snapshot**.
3. Test post-correctif : envoi/réception d'un email de bout en bout + vérification que la purge des journaux fonctionne.

### Simulation CAB (Change Advisory Board)
- **Avis favorable (Responsable infra) :** « Correctif éditeur qui traite une cause racine avérée (3 incidents). Fenêtre hors production, rollback par snapshot testé. **Favorable.** »
- **Avis réservé (Référent sécurité) :** « Favorable **sous conditions** : valider l'intégrité/l'origine du correctif, et communiquer aux utilisateurs 48 h à l'avance. »
- **Décision :** **approuvé** pour la fenêtre proposée, sous les conditions ci-dessus.

### Traçabilité GLPI (le fil rouge)
Dans le Changement #3001, créer les liens :
- Onglet **« Problèmes »** → lier au **Problème #2001**.
- Onglet **« Tickets »** → lier aux **incidents** de coupure mail (Phases 1 et 2).

**Livrable Phase 3 :** RFC #3001 rédigée + liens Changement ↔ Problème ↔ Incidents visibles dans GLPI.

---

## Phase 4 — Gestion des configurations (CMDB)

### Chemin GLPI
*Parc > (Serveurs / Ordinateurs)* pour la fiche CI, puis onglets **« Liens »/« Connexions »** et vue **« Analyse d'impact »**.

### Fiche CI — SRV-MAIL01
```
Nom          : SRV-MAIL01
Type         : Serveur (CI)
Statut       : En production
Rôle         : Serveur de messagerie
Localisation : Salle serveur - siège NORTEK SI
Criticité    : Haute
```

### CI liés (dépendances à documenter)
Arbre de dépendances de SRV-MAIL01 :

```
SRV-MAIL01 (serveur mail)
├── dépend de → SRV-AD01 (Active Directory / annuaire)      [authentification]
├── dépend de → SRV-BACKUP01 (sauvegarde)                    [restauration]
├── dépend de → Lien réseau / Switch cœur + Pare-feu         [relais SMTP entrant/sortant]
└── est utilisé par → 200 postes clients / boîtes utilisateurs [messagerie]
                    → GLPI et applis métier                   [notifications par email]
```
Dans GLPI : créer ces relations (« dépend de » / « est utilisé par ») entre les CI ; la vue **Analyse d'impact** génère le graphe automatiquement.

### Note d'impact — simulation de panne de SRV-MAIL01
Si SRV-MAIL01 tombe (ou pendant la fenêtre de changement) :
- **Utilisateurs impactés :** l'ensemble des 200 collaborateurs (plus d'email entrant/sortant, webmail KO).
- **Services impactés :** notifications automatiques de GLPI et des applications métier (envoyées par email), remise différée des mails entrants (file d'attente), workflows dépendant du mail.
- **Non impactés directement :** l'annuaire AD et les fichiers restent accessibles (le mail dépend d'eux, pas l'inverse) → utile pour cadrer le périmètre réel de l'interruption.

**Livrable Phase 4 :** fiche CI SRV-MAIL01 + liste des CI dépendants + note d'impact.

---

## Phase 5 — Gestion des demandes de service

### Chemin GLPI
*Assistance > Tickets > Ajouter*, **Type = Demande** (et non Incident).

### Service Request #4001 — Onboarding IT de Julie
```
Type         : Demande (Service Request)
Catégorie    : Onboarding
Titre        : Onboarding IT - Julie (chargée de communication)
Demandeur    : Manager de Julie
Bénéficiaire : Julie
Priorité     : Moyenne
SLA          : SLA "Demande" (distinct du SLA incident) -> délai de traitement 5 j ouvrés
Date cible   : vendredi (veille de l'arrivée), mise à disposition avant lundi
Statut       : En cours
```
**Distinction incident/demande :** ce n'est **pas une panne** mais une **demande planifiée** issue du catalogue de services → Service Request Management, avec un SLA propre (plus long, orienté délai de mise à disposition, pas rétablissement).

### Catalogue de services — éléments à traiter (checklist / tâches GLPI)
- [ ] **Création compte utilisateur + boîte mail** (AD + messagerie)
- [ ] **Attribution poste + périphériques** (PC, écran, clavier/souris)
- [ ] **Accès VPN**
- [ ] **Accès au dossier partagé « Communication »**

Créer une **tâche GLPI par élément** pour suivre l'avancement, avec la date cible du vendredi.

**Livrable Phase 5 :** Service Request #4001 complet, SLA demande + échéance renseignés, 4 tâches du catalogue.

---

## Vérification finale du fil rouge (liens à contrôler dans GLPI)

| Lien à vérifier | Où |
|-----------------|-----|
| 3 incidents mail ↔ Problème #2001 | Onglet « Tickets » du Problème |
| Problème #2001 ↔ Changement #3001 | Onglet « Problèmes » du Changement |
| Incidents ↔ Changement #3001 | Onglet « Tickets » du Changement |
| KEDB ↔ Problème #2001 | Onglet « Base de connaissances » du Problème |
| SRV-MAIL01 ↔ CI dépendants | Vue « Analyse d'impact » du CI |
| Changement #3001 ↔ SRV-MAIL01 | Onglet « Éléments » du Changement |

Si ces six liens sont visibles, le point « cohérence globale du fil rouge » de la grille est acquis.
