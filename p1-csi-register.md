# Partie 1 — Diagnostic (4 dimensions + Continual Improvement)

**Établissement :** centre hospitalier (CHOR) — environnement de production de santé.
**Service analysé :** support / infrastructure réseau de niveau 2 (N2), qui reçoit les escalades du helpdesk N1 et de l'équipe câblage.
**Symptômes rapportés :** lenteur de traitement · tickets perdus · utilisateurs qui rappellent plusieurs fois pour le même problème.

> Contexte réel : dans les chambres et services, le brassage câble et la configuration de port sont réalisés par une **équipe câblage** qui escalade ensuite vers l'infra pour diagnostic. La CMDB de l'établissement est **NetBox** ; la supervision repose sur **Zabbix**, **Netdisco** et **SNMP**.
>
> Vocabulaire ITIL 5 : ces quatre dimensions sont les « ITIL Four Dimensions of Product and Service Management ».

---

## 1. Constats par dimension

### 1.1 Organisations & personnes
**Constat :** la chaîne de traitement traverse trois acteurs — utilisateur → helpdesk N1 → équipe câblage → infra réseau (N2) — mais le passage de relais entre câblage et infra se fait souvent **à l'oral**, sans propriétaire unique du ticket de bout en bout. Un problème escaladé informellement n'a plus de porteur identifié : il stagne (**ticket perdu**) et l'utilisateur relance faute d'interlocuteur (**rappels**).
**Symptôme(s) :** tickets perdus, rappels répétés.

### 1.2 Information & technologie
**Constat :** les outils existent mais ne se parlent pas toujours. Certains équipements ne remontent pas dans la supervision (ex. un switch d'accès visible seulement en CDP/LLDP dans **Netdisco** mais non managé faute de chemin SNMP fonctionnel), ce qui crée des **angles morts** : le problème n'est pas détecté proactivement et surgit sous forme de plainte utilisateur. Par ailleurs, l'absence de base de connaissances impose de **re-diagnostiquer** à chaque fois des pannes déjà rencontrées (duplex, VLAN ToIP…), ce qui allonge les délais.
**Symptôme(s) :** lenteur, rappels répétés.

### 1.3 Partenaires & fournisseurs
**Constat :** certains diagnostics dépendent d'un tiers et l'attente n'est pas tracée comme telle. Exemple vécu : un équipement médical connecté (**Dräger**) présentant des erreurs de lien relève d'une escalade vers l'**équipe biomédicale** (fournisseur/partenaire interne) ; sans statut « en attente tiers » ni relance cadrée, le ticket reste immobilisé et l'utilisateur rappelle pour connaître l'avancement. Idem pour les équipements en attente de livraison ou de configuration par un prestataire.
**Symptôme(s) :** lenteur, rappels répétés.

### 1.4 Value streams & processus
**Constat :** le flux de traitement n'est pas formalisé et surtout **il n'y a pas de critère de clôture sur confirmation**. Sur les pannes **intermittentes** (typiquement le lien médical en 10 Mbps half-duplex avec compteurs d'erreurs : CRC, late collisions, OutDiscards), le ticket peut être clôturé après une accalmie apparente alors que la cause racine n'est pas traitée. Le problème réapparaît → l'utilisateur **rappelle** → doublon pour le même incident.
**Symptôme(s) :** rappels répétés pour un même problème, tickets perdus.

---

## 2. CSI Register (registre d'amélioration continue)

> Note : la pratique ITIL 4/5 s'appelle **Continual Improvement** ; PeopleCert parle du **Continual Improvement Register (CIR)**. On conserve l'intitulé « CSI Register » de l'énoncé — c'est le même registre.

| # | Amélioration | Effort | Impact | Symptômes visés |
|---|--------------|:------:|:------:|-----------------|
| A | **Fiabiliser la CMDB et automatiser la détection des événements physiques de câblage** : détection des connexions/déconnexions de ports (trap SNMP) → mise à jour NetBox → alerte de l'équipe infra. | Moyen/Fort | **Fort** | Tickets perdus (ports/mouvements non documentés), lenteur |
| B | **Supprimer les angles morts de supervision** : garantir que chaque équipement d'accès est réellement managé (SNMP joignable, Netdisco/Zabbix), et formaliser un statut « En attente tiers » + clôture sur confirmation. | Faible/Moyen | **Fort** | Lenteur (détection proactive), rappels, tickets bloqués tiers |
| C | **Base de connaissances** des diagnostics récurrents (duplex/CRC, VLAN ToIP, SNMP bloqué par le pare-feu…). | Fort | Moyen | Lenteur sur récurrents, rappels pour un même sujet |

### Priorisation justifiée (P1 > P2 > P3)

1. **P1 — Amélioration B.** Meilleur ratio impact/effort et pré-requis des autres : sans supervision fiable, on ne détecte rien et on ne mesure rien. Le volet « statut en attente tiers + clôture sur confirmation » est de la configuration réversible et attaque directement rappels et tickets bloqués.
2. **P2 — Amélioration A.** Impact fort sur la fiabilité de la CMDB et les tickets liés à des ports non documentés, mais effort de mise en œuvre plus élevé (automatisation, VM dédiée, tests). C'est un projet en tant que tel (cf. Partie 3).
3. **P3 — Amélioration C.** Effort le plus élevé et bénéfice **différé** : on documente d'autant mieux que le flux amont (B) et la CMDB (A) sont déjà fiables.

---

## 3. Principe directeur mobilisé

**Principe retenu : « Progresser de manière itérative avec du feedback ».**

**Pourquoi :** on démarre par l'incrément le plus sûr et mesurable (B : rendre la supervision fiable et poser les statuts), on **mesure** l'effet (équipements réellement managés, tickets sans statut, taux de rappels) *avant* d'engager l'automatisation (A), plus lourde, puis la KB (C). C'est aussi la logique de mon projet d'automatisation, développé et **testé en simulation** avant l'arrivée du matériel réel.

*Principes secondaires :* « Commencer là où vous êtes » (réutiliser NetBox/Zabbix/Netdisco déjà en place) et « Rester simple et pratique ».
