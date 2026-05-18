## Assets & Identities

Pour trier une alerte (ex : **G.Baker se connecte sur HQ-FINFS-02**, télécharge un fichier financier et le partage à **R.Lund**), il faut du **contexte** :

- **Qui** est l’utilisateur ? (rôle, horaires, privilèges, localisation)
    
- **Quel** est l’actif ? (serveur/PC, purpose, propriétaire, localisation)
    
- **Pourquoi** l’action serait légitime ? (besoin métier, accès attendu)

---

### Identity Inventory (inventaire des identités)

Catalogue des **comptes** :

- employés (user accounts)
    
- services (machine/service accounts)
    

- infos utiles : **rôle, privilèges, contacts, localisation, accès**
    

➡️ Sert à comprendre rapidement _qui est G.Baker / R.Lund_ et si leur activité est cohérente.

#### Exemple d’identités (à retenir)

|Full Name / Compte|Username|Role|Location|Access|
|---|---|---|---|---|
|Gregory Baker|G.Baker|Chief Financial Officer (CFO)|Europe, UK|VPN, HQ, FINANCE|
|Raymond Lund|R.Lund|US Financial Adviser|US, Texas|VPN, FINANCE|
|Kate Danner|K.Danner|Chief Technology Officer (CTO)|Europe, UK|VPN, DA, HQ, AWS|
|Backup Service Account|svc-veeam-06|Service account|N/A|VEEAM, DMZ, HQ|
|Web App Service Account|svc-nginx-pp|Service account|N/A|DMZ|

**Point SOC important :**

- Les **comptes de service** (svc-*) ne sont pas des humains → comportement attendu = automatisé / spécifique à un service.
    

#### Sources possibles d’identités

|Solution|Exemples|Ce que ça apporte|
|---|---|---|
|Active Directory|On-prem AD, Entra ID|Base d’identité la plus utilisée en SOC|
|SSO Providers|Okta, Google Workspace|Alternative cloud à AD, recherche utilisateurs facile|
|HR Systems|BambooHR, SAP, HiBob|Données RH riches mais seulement employés|
|Custom|CSV / Excel|Très courant en IT/Sécu (moins fiable/à jour)|

---

### Asset Inventory (inventaire des actifs)

Liste des **ressources informatiques** (dans cette room : **serveurs + workstations** uniquement).  
➡️ Sert à comprendre _c’est quoi HQ-FINFS-02_ (où, à quoi ça sert, qui le gère).

#### Exemple d’actifs

|Hostname|Location|IP|OS|Owner|Purpose|
|---|---|---|---|---|---|
|HQ-FINFS-02|UK Datacenter|172.16.15.89|Windows Server 2022|Central IT|File server for financial records|
|HQ-ADDC-01|UK Datacenter|172.16.15.10|Windows Server 2019|Central IT|Primary AD domain controller|
|PC-891D|London Office|192.168.5.13|Windows 11 Pro|Tech Support|PC fixe pour comptables|
|L007694|Remote|N/A|MacOS 13|A.Kelly (DevOps)|Laptop corporate|
|L005325|Remote|N/A|MacOS 13|J.Eldridge (HR)|Laptop corporate|

**Point SOC important :**

- Un serveur “FINFS” + “financial records” → accès et exfiltration potentiels = **sensibles** (données financières).
    

#### Sources possibles d’actifs

|Solution|Exemples|Ce que ça apporte|
|---|---|---|
|Active Directory|On-prem AD, Entra ID|AD peut aussi servir de base d’inventaire machines|
|SIEM / EDR|Elastic, CrowdStrike|Agents remontent infos hôtes monitorés|
|MDM|Intune, Jamf|Inventaire + gestion des endpoints (surtout laptops)|
|Custom|CSV / Excel|Courant, mais dépend de la qualité de maintenance|

---

### Ce qu’il faut retenir (ultra synthèse)

- **Identity inventory = “qui fait quoi”** (personnes + comptes services + rôles + accès).
    
- **Asset inventory = “c’est quoi cette machine”** (serveur/PC + localisation + owner + purpose).
    
- En triage SOC, ces deux lookups permettent de juger si une action est **attendue** ou **suspecte** (rôle cohérent ? accès normal ? machine sensible ?).

---

## Network Diagrams

- quel **service** est exposé sur un port (ex: 10443)
    
- à quel **subnet** appartient une IP interne (ex: 10.10.0.53)
    
- quels **flux sont possibles/bloqués** entre subnets (règles firewall)
    
- reconstruire un **chemin d’attaque** (kill chain réseau)

![[SOC Workbooks & Lookups (Network Diagrams).png]]

---

### Logs observés

|Heure|Observation|Traduction SOC (hypothèse)|
|---|---|---|
|08:00|103.61.240.174 se connecte au firewall sur **TCP/10443** en boucle|Cible probable : **VPN** (souvent sur ports “non standards” type 10443) → brute force / tentative d’accès|
|08:23|103.61.240.174 est **NAT** vers **10.10.0.53**|Connexion acceptée / tunnel établi → l’attaquant obtient une IP interne (VPN pool)|
|08:25|10.10.0.53 scanne **172.16.15.0/24** sans ports ouverts|Reconnaissance / scan, mais **bloqué** (filtrage) ou pas de services accessibles|
|08:32|10.10.0.53 scanne **172.16.23.0/24**, attaque continue|Pivot vers un autre subnet plus accessible (souvent postes users)|

---

### Ce que montre le diagramme réseau

|Élément|Détail|
|---|---|
|Firewall expose|**VPN** sur **10443** (et web sur ports HTTP classiques)|
|Subnets derrière firewall|**Office** (172.16.23.0/24), **Database** (172.16.15.0/24), **VPN** (10.10.0.0/16)|
|Point clé|Le diagramme te dit **où peut aller** une IP VPN et ce que le firewall **bloque/autorise**|

---

### Reconstruction du chemin d’attaque

1. 103.61.240.174 → **brute force VPN** (cible : `vpn.tryhatme.thm` sur 10443)
    
2. Succès → attribution d’une IP **VPN subnet** : **10.10.0.53**
    
3. Scan **Database subnet** (172.16.15.0/24) → échec probable car **règles firewall restrictives**
    
4. Scan **Office subnet** (172.16.23.0/24) → tentative de trouver une machine pivot / prochaine cible

---

### À retenir

- **Port + NAT + subnet = contexte** : sans diagramme, les IPs sont “muettes”.
    
- Les scans successifs entre subnets révèlent souvent : **accès initial (VPN) → reconnaissance → pivot**.

---

## Workbooks Theory

Les inventaires (identités/actifs) + diagrammes réseau donnent le **contexte**.  
Mais pour rendre un **verdict** (safe / suspect), certaines alertes demandent **beaucoup d’étapes** : sans cadre, tu risques d’oublier des vérifs ou d’interpréter des preuves de travers.

---

### SOC Workbooks (synonymes)

Un **SOC workbook** (aussi appelé **playbook / runbook / workflow**) est un document structuré qui décrit :

- les **étapes d’investigation**
    
- les **actions de remédiation**  
    pour traiter une menace **de façon efficace et cohérente**.
    

**Objectif SOC :**

- aider les **L1 (junior)** à trier correctement même sans expérience complète
    
- standardiser la qualité, éviter les erreurs, accélérer la prise de décision
    
- imposer un process “evidence-based” (pas de verdict sans preuves)
    

---

### Exemple : “Unusual Login Location”

Workbook typique pour login atypique (email/web/VPN) : schéma + guide texte + liens vers outils.

![[SOC Workbooks & Lookups (Unusual Login Loc).png]]

#### Structure standard (3 blocs logiques)

| Bloc              | But                      | Exemples d’actions                                                                                    |
| ----------------- | ------------------------ | ----------------------------------------------------------------------------------------------------- |
| **Enrichment**    | Obtenir le contexte      | Threat Intel sur IP/domain, identité via HR/identity inventory (ex: BambooHR)                         |
| **Investigation** | Décider si c’est attendu | Corréler avec logs SIEM (ex: Splunk), vérifier comportement avant/après login, comparer aux habitudes |
| **Escalation**    | Agir si doute/incident   | Escalader **L2**, ou contacter l’utilisateur si nécessaire, ou clôturer si bénin                      |

---

### À retenir

- Workbook = **checklist + workflow** pour ne rien oublier.
    
- Conçu par **seniors** pour guider les **L1**.
    
- Sépare clairement **contexte → preuve → décision → action**.

---


