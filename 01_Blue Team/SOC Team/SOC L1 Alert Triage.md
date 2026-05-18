## Events and Alerts

Un **event** (activité) est **loggé**, envoyé à une solution (SIEM/EDR), puis transformé en **alert** quand un **pattern/condition suspecte** est détecté. Le **SOC L1** fait le **tri** (vrai/faux, priorité) et **escalade** à L2 si menace réelle.

---

### De l’Event à l’Alert

1. **Event** se produit (login, lancement de process, téléchargement…)
    
2. **Log** généré par un système (OS, firewall, cloud…)
    
3. **Logs centralisés** dans une solution sécurité (SIEM/EDR…)
    
4. **Alert** générée si un **événement ou une séquence** correspond à une règle/anomalie  
    ➡️ But : éviter de lire **des millions de logs** → triage de **quelques dizaines d’alertes/jour**.
    

---

### Plateformes de gestion d’alertes

|Type|Exemples|Rôle / idée clé|
|---|---|---|
|**SIEM**|Splunk ES, Elastic|Très bon pour **centraliser** et **gérer** les alertes → choix “standard” SOC|
|**EDR / NDR**|MS Defender, CrowdStrike|Ont leurs dashboards, mais souvent **on centralise plutôt dans SIEM/SOAR**|
|**SOAR**|Splunk SOAR, Cortex SOAR|Pour SOC plus “gros” : **agrège** plusieurs sources + **automatisation**|
|**ITSM / Ticketing**|Jira, TheHive|Gestion **tickets/workflow** (parfois custom)|

---

### Rôle du SOC L1 dans l’alert triage

- **SOC L1 (toi)** :
    
    - **review** des alertes
        
    - distingue **benin vs malveillant** (false positive vs true positive)
        
    - **escalade** à L2 si menace
        
- **SOC L2** : analyse approfondie + **remédiation**
    
- **SOC engineers** : font en sorte que les alertes aient **assez de contexte** pour trier vite
    
- **SOC manager** : suit **vitesse** et **qualité** du triage (éviter de rater une vraie attaque)
    

---

### Exemples d’alertes vues (interprétation rapide)

- **“Email Marked as Phishing”** : probable signalement utilisateur / filtre mail → souvent volumineux
    
- **“Unusual Gmail Login Location”** : connexion depuis lieu atypique → à valider (voyage, VPN, compromission)
    
- **“Unapproved Mimikatz Usage”** : outil associé au vol d’identifiants → **alerte critique** à prioriser/escalader
    

Si tu m’envoies la suite du cours (ou les sections suivantes), je continue dans le même format “prise de notes révision”.

---

## Alert properties

![[SIEM Dashboard (fake).png]]

|#|Propriété|À quoi ça sert|Notes / exemples clés|
|---|---|---|---|
|1|**Alert Time**|Heure de **création** de l’alerte|Souvent **quelques minutes après** l’event (ex : _Alert 15:35_ vs _Event 15:32_)|
|2|**Alert Name**|Résumé basé sur le **nom de la règle**|_Unusual Login Location_, _Email Marked as Phishing_, _RDP Bruteforce_, _Data Exfiltration_|
|3|**Severity**|**Urgence** (priorité)|Défini par les détection engineers, **modifiable** par analystes : 🟢 Low, 🟡 Medium, 🟠 High, 🔴 Critical|
|4|**Status**|Indique si l’alerte est **prise en charge**|🆕 New → 🔄 In Progress → ✅ Closed (+ statuts custom)|
|5|**Verdict / Classification**|“C’est une menace ?”|🔴 True Positive vs 🟢 False Positive (+ verdicts custom)|
|6|**Assignee / Owner**|Qui est **responsable** de l’alerte|L’assigné “porte” l’alerte jusqu’à clôture|
|7|**Description**|Donne le **contexte**|Souvent : 1) logique de la règle 2) pourquoi c’est suspect 3) parfois procédure de triage|
|8|**Fields**|Données déclencheuses + commentaires|Ex : **hostname affecté**, **command line**, etc. (varie selon alerte)|

### À retenir

- **Ne confonds pas** _Alert Time_ (création) et _Event Time_ (moment réel).
    
- **Severity** = urgence ; **Verdict** = conclusion (vrai/faux).
    
- **Status + Assignee** = workflow/ownership ; **Fields + Description** = matière pour trier vite.


---

## Alert Priorisation

**Définition :** décider **quelle alerte traiter en premier** pour détecter une menace à temps, surtout quand la file est chargée.

---

**Approche “simple & standard” (souvent automatisée dans SIEM/EDR)**

|Étape|Règle|Objectif|Exemple concret|
|---|---|---|---|
|1|**Filtrer**|Éviter doublons / travail en parallèle inutile|Ne prendre que **New / Unassigned / unresolved** (pas “In Progress”, pas “Closed”)|
|2|**Trier par sévérité**|Traiter d’abord le **plus risqué / impactant**|🔴 Critical → 🟠 High → 🟡 Medium → 🟢 Low|
|3|**Trier par temps**|Réduire le “dwell time” des attaques déjà en cours|À sévérité égale : **plus ancien d’abord** (l’attaquant a potentiellement déjà avancé)|

---

### À retenir

- **On ne prend que ce qui est “à nous prendre”** (non traité, non assigné).
    
- **Severity d’abord**, car calibrée pour refléter **probabilité + impact**.

---

## Alert Triage

**Synonymes** : alert handling / processing / investigation / analysis.  
Dans le module : **Alert Triage**.

![[SOC Alert Triage Workflow.png]]

---

### Workflow standard (3 phases)

|Phase|But|Actions clés|Output attendu|
|---|---|---|---|
|**1) Initial Actions**|**Prendre ownership** sans gêner les autres|Assigner l’alerte à toi → passer en **In Progress** → lire **Name / Description / Indicateurs**|Tu es responsable + prêt à enquêter|
|**2) Investigation**|Déterminer si l’activité est légitime ou malveillante|Analyse technique dans SIEM/EDR + logs|Hypothèse solide (TP/FP) + preuves|
|**3) Final Actions**|**Décider + documenter + clôturer**|Choisir **True Positive** ou **False Positive** → commenter (étapes + raisons) → mettre **Closed**|Alerte classifiée, traçable, fermée|

---

### Investigation — check-list L1 (si pas de workbook/playbook)

- **Identifier “qui” est touché** : user, hostname, cloud asset, réseau, site web…
    
- **Identifier “quoi”** : login suspect, malware, phishing, etc.
    
- **Corréler les événements autour** : avant/après l’alerte (signaux faibles, enchaînements suspects)
    
- **Valider avec des ressources** : threat intel / outils internes / contexte connu
    

> Note : certaines équipes ont des **Workbooks** (= playbooks/runbooks) : procédures par type d’alerte pour accélérer et standardiser.


