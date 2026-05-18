## Alert Funnel

|Niveau|Rôle principal|Issue typique|
|---|---|---|
|**L1**|Reçoit les alertes (SIEM / EDR / ticketing), **triage + première analyse**|La majorité **close** en _False Positive_ ou traitées au niveau L1|
|**L2**|Analyse plus poussée + **remédiation**|Gère les alertes **complexes / menaçantes** (True Positives escaladées)|
|**DFIR**|Investigation forensique + réponse à incident|Seulement les cas les plus graves (incident avéré)|

**Exemple de funnel :** 100 alertes en L1 → **10 True Positives** escaladées en L2 → **1 incident** finit en **DFIR**.

---

### 3 notions à connaître pour “envoyer plus loin” une alerte

#### 1) Alert Reporting (rapport)

- Avant de **clore** ou **transférer** à L2, il peut être demandé de **documenter l’investigation**.
    
- Selon les standards + la sévérité :
    
    - soit **commentaire court**,
        
    - soit **rapport détaillé** avec **toutes les preuves** (surtout pour les **True Positives**).
        

#### 2) Alert Escalation (escalade)

- Si l’alerte **TP** nécessite actions supplémentaires / analyse plus profonde → **escalade vers L2** via les procédures définies.
    
- Le **report** sert de **contexte initial** à L2 → évite de repartir de zéro, **gain de temps**.
    

#### 3) Communication (inter-services)

- Pendant / après l’analyse, besoin de **valider du contexte** avec d’autres équipes :
    
    - **IT** : confirmer un changement légitime (ex : droits admin accordés)
        
    - **HR** : infos sur un employé (ex : nouvel arrivant)
        

---

🧠 **À retenir :**  
**L1 = triage + preuve**, **report = trace exploitable**, **escalade = passage à L2**, **communication = contexte métier/légitime**.

---

## Reporting Guide

|Objectif du report|Explication (essentiel)|
|---|---|
|**Donner du contexte à l’escalade**|Un bon report **fait gagner du temps** à L2 et leur permet de **comprendre rapidement** ce qui s’est passé.|
|**Garder une trace durable**|Les **logs SIEM** sont souvent conservés **3–12 mois**, alors que les **alertes** restent **indéfiniment** → mieux vaut **capturer le contexte dans l’alerte**.|
|**Améliorer les compétences d’investigation**|Si tu ne peux pas l’expliquer simplement, tu ne le maîtrises pas → écrire un report **force à structurer** et renforce les skills L1.|

---

### Format recommandé : **5W (Five Ws)**

> Rédige comme si tu étais **L2 / DFIR / IT** : ils doivent comprendre l’alerte _sans te recontacter_.

|W|À inclure dans le report|
|---|---|
|**Who**|Quel **utilisateur** (login), qui exécute une commande / télécharge un fichier|
|**What**|Quelle **action exacte** / **séquence d’événements**|
|**When**|**Début / fin** de l’activité suspecte (timestamps)|
|**Where**|Quel **device**, **IP**, **site/web** impliqué|
|**Why** _(le + important)_|Ton **raisonnement** menant au verdict final (**TP/FP**) + justification|

**Mémo rapide :** _Who/What/When/Where = faits_, **Why = conclusion argumentée**.

---

## Escalation Guide

|Escalader si…|Exemple / idée|
|---|---|
|**Signe d’attaque majeure** → besoin d’investigation +/ou **DFIR**|Compromission, mouvement latéral, exfiltration, etc.|
|**Remédiation nécessaire**|Suppression malware, **isolation host**, reset mdp, blocage compte|
|**Communication externe / management** requise|Clients, partenaires, direction, forces de l’ordre|
|**Tu ne comprends pas assez** → besoin d’un senior|Mieux vaut demander que **clore à l’aveugle**|

---

### Étapes d’escalade (process “classique”)

- **Réassigner le ticket** au **L2 on-shift**
    
- **Ping** L2 (chat interne / en personne)  
    _(Parfois : formulaire d’escalade très détaillé selon l’équipe.)_
    

![[SOC L1 Alert Reporting (Escalation steps).png]]

---

### Après escalade : ce que fait L2

- Lit ton **report**
    
- Te recontacte si questions
    
- **Valide TP/FP**, creuse l’analyse
    
- Coordonne avec d’autres équipes si besoin
    
- Si incident majeur → déclenche **Incident Response** formelle  
    **Exemple** : phishing escaladé → L2 **rotate/reset credentials**.
    

---

### Demander du support L2 (normal)

- OK de demander de l’aide si c’est flou, surtout au début.
    
- Flux type : **L1 demande → L2 accepte → partage/mentorat** (knowledge sharing).

![[SOC L1 Alert Reporting (Requesting L2 support).png]]

---

### SOC Dashboard Escalation (checklist)

1. Mettre l’alerte en **In Progress** + analyser
    
2. Écrire le **report** + poser le **verdict** (ex: TP)
    
3. Si besoin : **assign à L2 on-shift**
    
4. L2 est notifié et démarre **depuis ton report** ✅

---

## SOC Communication

- Idéal : procédures **Crisis Communication** déjà définies.
    
- Sinon : connaître les **cas critiques** + quoi faire.

---

### Cas de communication

|Situation|Action recommandée|
|---|---|
|**Alerte critique urgente** et **L2 ne répond pas (30 min)**|Avoir les **contacts d’urgence**. **Appeler** : L2 → **L3** → **manager**.|
|Suspected **compromission Slack/Teams** et besoin de valider avec l’utilisateur|**Ne jamais** contacter via le canal potentiellement compromis. Utiliser **téléphone** / autre moyen.|
|Pic d’alertes massif, dont des critiques|**Prioriser** selon le workflow, et **prévenir L2** de la surcharge.|
|Tu réalises plus tard une **mauvaise classification** (possible action malveillante manquée)|**Prévenir immédiatement L2** + expliquer. Un attaquant peut rester **silencieux des semaines**.|
|Triage impossible : logs SIEM mal parsés / non recherchables|**Ne pas skip**. Investiguer ce que tu peux + **remonter le problème** à L2 ou au **SOC engineer**.|

---

### Communication côté L2 (exemple)

- L1 escalade une alerte **data leak** → L2 peut lancer **DFIR** + contacter **Legal** et **PR**.

![[SOC L1 Alert Reporting (Communication by L2).png]]