- **Secrets** = données critiques/confidentielles
- **Threat actors** = attaquants
- **Vulnerability** = vulnérabilité exploitable
- **SOC** = surveillance + détection + réponse en continu

## **Detect & Respond**

|Axe|But|Comment|
|---|---|---|
|**Detection**|Identifier rapidement ce qui est anormal / risqué|Monitoring continu + indices (logs, localisation, comportements, etc.)|
|**Response**|Réduire l’impact + comprendre la cause|Containment/mitigation + root cause analysis + support à l’équipe IR|

---

**ce que le SOC Detecte**

|Type|Définition|Exemple|
|---|---|---|
|**Vulnérabilité**|Faiblesse exploitable (OS / logiciel / système)|PCs Windows à **patcher** contre une vuln publiée|
|**Activité non autorisée**|Usage illégitime d’un compte / accès|Connexion avec identifiants volés, **géoloc** suspecte|
|**Violation de politique**|Non-respect des règles de sécu internes|Téléchargement piraté, envoi de docs confidentiels non sécurisé|
|**Intrusion**|Accès non autorisé à un système/réseau|Exploit d’une web app, infection après site malveillant|

--- 

**Réponse --> rôle du SOC**

- **Minimiser l’impact**
- Faire l’**analyse de cause racine** (RCA)

---

**3 pilliers d'un SOC**

|Pilier|Idée clé|
|---|---|
|**People**|équipe qualifiée (analystes, expertise)|
|**Process**|procédures/workflows (escalade, gestion d’incident)|
|**Technology**|outils de sécurité modernes + centralisation|

## **People**

**Pourquoi People rest indispensable en SOC (même avec automatisation)**

L’automatisation génère souvent beaucoup de **“bruit”** (alertes/red flags). Sans humains, on risque de traiter des **faux positifs** et de gaspiller temps/ressources (analogie : alarmes incendie déclenchées par la fumée de cuisine).  

➡️ Les analystes **filtrent, qualifient** et déclenchent une **réponse rapide** sur les vrais signaux.

### **Hiérarchie / rôle SOC**

![[SOC hiérarchie.png]]

|Rôle|Mission principale|Ce qu’il fait concrètement|
|---|---|---|
|**SOC Analyst L1** (Tier 1)|**Premier tri** / premiers répondants|Reçoit toutes les alertes → **triage** basique (harmful ou non), escalade + reporting via les bons canaux|
|**SOC Analyst L2** (Tier 2)|**Investigation approfondie**|Analyse plus poussée, **corrèle** plusieurs sources (logs/outils) pour confirmer/infirmer|
|**SOC Analyst L3** (Tier 3)|**Chasse & réponse incident avancée**|Recherche proactive d’**indicateurs de menace**, gère les cas critiques : **containment / eradication / recovery**|
|**Security Engineer**|**Déploiement & config des outils**|Installe, configure, maintient les solutions de sécurité pour qu’elles tournent correctement|
|**Detection Engineer**|**Création des règles de détection**|Conçoit/maintient les **règles** (logique) qui déclenchent les alertes (souvent fait aussi par L2/L3)|
|**SOC Manager**|**Pilotage opérationnel**|Supervise les **process**, supporte l’équipe, communique avec le **CISO** sur posture/efforts sécu|

## **Process**

**Report SOC exemples --> 5W (à mettre dans le ticket)**

|W|Contenu|
|---|---|
|**What**|Fichier malveillant détecté sur un hôte interne|
|**When**|13:20 — **5 juin 2024**|
|**Where**|Répertoire de l’hôte **“GEORGE PC”**|
|**Who**|Utilisateur **George**|
|**Why**|Téléchargé depuis un site de vente de logiciels piratés (volonté d’obtenir un logiciel gratuitement)|

---

**Reporting (escalade / ticketing)**

- Les alertes **harmful** doivent être **escaladées** vers des analystes plus seniors pour une réponse rapide.
- L’escalade se fait via des **tickets** assignés aux personnes/équipes concernées.
- Un bon report = **5W + analyse** (ce qui a été vérifié, conclusions).
- Ajouter des **captures d’écran** comme **preuves** (evidence).

---

**Incident Response & Forensics (quand ça devient critique)**

| Domaine                    | Quand                                                              | Objectif                                                                 |
| -------------------------- | ------------------------------------------------------------------ | ------------------------------------------------------------------------ |
| **Incident Response (IR)** | Détections indiquant une activité **critique** / très malveillante | **Gérer l’incident** (contenir, éradiquer, récupérer)                    |
| **Forensics**              | Quand il faut aller plus loin dans les détails                     | Trouver la **root cause** via l’analyse d’**artefacts** (système/réseau) |

(Idée clé : une détection peut déclencher un simple ticket… ou basculer en IR/forensics si gravité élevée.)

## **Technology**

- **People** + **Process** ne suffisent pas sans **solutions de sécurité**.
- **Objectif** : **réduire le travail manuel** via **centralisation** (infos/logs de tout le réseau) + **automatisation** (détection/réponse).

**pourquoi centraliser ?**

- Un réseau = beaucoup de **devices + applis** → surveiller “un par un” coûte trop cher.
- Les outils SOC **agrègent** les données et **orchestrent** la détection / réponse.

---

**Outils SOC principaux**

|Solution|À quoi ça sert|Points clés|
|---|---|---|
|**SIEM**|**Collecte & corrélation de logs** + alertes|Log sources → règles de détection → corrélation multi-sources → alertes. SIEM modernes : **UBA/UEBA**, **threat intel**, **ML**. ⚠️ Ici : **détection seulement**.|
|**EDR**|**Visibilité endpoint** + **réponse**|Vue temps réel + historique sur postes/serveurs, investigation détaillée, **actions de réponse** automatisées / “en quelques clics”.|
|**Firewall**|**Filtrage réseau** (barrière interne/externe)|Surveille trafic entrant/sortant, **bloque** l’illégitime. Peut aussi avoir des **règles de détection** pour stopper du trafic suspect avant qu’il atteigne l’interne.|
**SIEM** (Security Information and Event Management)
**EDR** (Endpoint Detection and Response)

---

**Autres solutions (à connaître)**

- **Antivirus**, **EPP**, **IDS/IPS**, **XDR**, **SOAR**, etc.
- Choix des technos = selon **surface d’attaque (threat surface)** + **ressources** dispo dans l’organisation.

