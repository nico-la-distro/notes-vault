
- L’**EDR (Endpoint Detection and Response)** est une solution de sécurité qui **surveille**, **détecte** et **répond** aux menaces avancées **au niveau des endpoints** (postes, serveurs). 

- Pour un **analyste SOC**, il est important de comprendre son fonctionnement car c’est une solution **très répandue** pour protéger les machines.

### Solutions in the market

- [**CrowdStrike Falcon**](https://www.crowdstrike.com/wp-content/uploads/2022/03/crowdstrike-falcon-insight-data-sheet.pdf)
- [**SentinelOne ActiveEDR**](https://sentinelone.com/resources/datasheets/assets/usecase/sentinel-one-active-#page=1)
- [**Microsoft Defender for Endpoint**](https://learn.microsoft.com/en-us/defender-endpoint/microsoft-defender-endpoint)
- [**OpenEDR**](https://www.openedr.com/)
- [**Symantec EDR**](https://docs.broadcom.com/doc/endpoint-detection-and-response-atp-endpoint-en)

---

## What is an EDR

- **Explosion des devices** + **augmentation des menaces**.
    
- Les protections “périmètre réseau” suffisent moins, surtout avec le **Remote Work** (endpoints **hors du réseau** de l’entreprise).  
    ➡️ Besoin d’une solution qui protège les **endpoints partout** et contre des **menaces avancées** : **EDR**.
    

**Définition** :

- **EDR (Endpoint Detection & Response)** = solution de sécurité **centrée hôte** qui assure **monitoring continu**, **détection** et **réponse** sur les endpoints, où qu’ils soient.

---

### Core features

|Pilier|Ce que ça apporte|À retenir|
|---|---|---|
|**Visibility**|Collecte **très détaillée** sur l’endpoint : **process**, **registry**, **fichiers/dossiers**, **actions utilisateur**, etc. + présentation structurée|Permet de voir **process tree** + **timeline** complète. Detections = **contexte riche** + accès à **l’historique** (threat hunting).|
|**Detection**|Combine **signature-based** + **behavior-based** + **ML** (déviation du baseline)|Peut détecter **fileless malware** (en mémoire). Supporte **IOCs custom**. Detections souvent **mappées MITRE** (tactic/technique).|
|**Response**|Actions à distance depuis la console centrale|Ex : **isoler un endpoint**, **tuer un process**, **quarantaine de fichiers**, **connexion remote** (RTR / Real Time Response) pour exécuter des actions.|

**Visibility**

![[EDR (visibility -> process tree).png]]

_The following screenshot shows graphical representation of a process tree. We can see which processes were spawned on the endpoint. Each node represents a process. The lines connecting them represents their relationship. If we click on the `+` icon given with each process, we will be able to see all the network connections, registry changes, file changes etc. associated with that process._

**Detection**

![[EDR (detection).png]]

_The following screenshot shows a dashboard of all the detections happening on the different endpoints. Each detection is represented by a row with different fields including the severity of the detection, time, triggering file, hostname, username, and more. The Tactic via Technique field maps the detection with MITRE. Any detection when clicked will show us rich details which helps a SOC analyst during the analysis._

**Response**

![[EDR (response).png]]

The following screenshot shows the actions available that can be taken on the host after connecting to it.

**Limite importante**

- **EDR = host-only** : très fort sur l’endpoint, mais **ne détecte pas** les menaces **au niveau réseau** (network-level threats).

---

## Beyond the Antivirus

Les deux protègent l’endpoint, mais **pas au même niveau** :

- **AV** : surtout **signature-based** (match avec une base de menaces connues).  
    ➜ efficace contre le “connu”, faible contre le **nouveau / evasive**.
    
- **EDR** : **surveillance continue + enregistrement des comportements** + **corrélation** + **réponse**.  
    ➜ détecte mieux les **menaces avancées** qui contournent les signatures.

### AV vs EDR

**scénario d’attaque (phishing → macro → PowerShell → payload → injection → C2)**

![[EDR Beyond antivirus (scenario breakdown).png]]

| Étape | Attaque                                     | Réponse AV (typique)                       | Réponse EDR                                                                     |
| ----- | ------------------------------------------- | ------------------------------------------ | ------------------------------------------------------------------------------- |
| 1     | Phishing + doc Word avec macro malveillante | Rien si pas de signature                   | **Log** du téléchargement + surveillance                                        |
| 2     | Ouverture du doc (winword.exe légitime)     | Rien                                       | **Trace** l’exécution de winword.exe + monitoring                               |
| 3     | Macro exécute PowerShell                    | Rien si macro inconnue                     | **Détecte** relation **parent-enfant anormale** (winword.exe → powershell.exe)  |
| 4     | PowerShell obfusqué télécharge payload      | Souvent non détecté                        | **Flag** exécution obfusquée                                                    |
| 5     | Injection dans svchost.exe                  | Souvent ignoré (pas de monitoring mémoire) | **Détecte Process Injection** dans svchost.exe                                  |
| 6     | Accès distant / connexion sortante          | AV souvent limité (peu de visibilité)      | **Détecte comportement anormal** + connexion sortante                           |
| Final | Résultat global                             | Peut être “clean”                          | **Alerte** avec **attack chain complète** + actions possibles depuis la console |


---

### A retenir

- **AV = connu / signatures** ; **EDR = comportement + contexte + réponse**
    
- L’EDR est “au-dessus” pour les **attaques modernes** (macro, PowerShell obfusqué, injection, fileless, etc.).
    
- Note : certains AV modernes s’améliorent, mais **EDR reste plus complet** en **détection + réponse** côté endpoint.

---

## How an EDR works ?

**Idée clé :** _Agents (capteurs) sur les endpoints → envoient la télémétrie → Console centrale corrèle/analyse → Alertes → Analyste enquête + répond._

![[EDR (agents & console).png]]

---

### Architecture EDR

|Composant|Rôle|Points clés|
|---|---|---|
|**EDR Agent / Sensor**|“Yeux & oreilles” sur l’endpoint|Monitor **toutes activités** et envoie en **temps réel** vers la console. Peut faire un peu de **détection locale** (signature + comportement) et remonter des alertes.|
|**EDR Console**|“Cerveau” central|**Corrèle** les données, applique **logique + ML**, matche avec **threat intel**, et produit des **detections/alerts**. Vue globale multi-endpoints.|

![[EDR (console).png]]

---

### Après une détection (workflow SOC)

1. **Acknowledge** l’alerte
    
2. **Prioriser** via la **sévérité** (Critical/High/Medium/Low/Info)
    
3. **Investiguer** : contexte complet (process, fichiers, réseau, registre, etc.)
    
4. **Qualifier** : **false positive** vs **true positive**
    
5. Si **TP** : **actions** directement depuis la console EDR (remédiation)
    

---

### EDR dans l’écosystème sécu

- L’EDR n’est pas seul : il coexiste avec **Firewall, DLP, Email Security Gateway, IAM**, etc.
    
- Pour centraliser et corréler côté analystes, tout est souvent intégré à un **SIEM** (point central d’investigation).

- _**DLP (Data Loss Prevention)** : solutions qui **détectent et empêchent la fuite de données** sensibles (exfiltration, envoi email, upload cloud, copie sur USB, impression, etc.). Basé sur des règles/politiques (types de données, labels, mots-clés, regex, classification)._
    
- _**IAM (Identity and Access Management)** : gestion des **identités** et des **droits d’accès**. Ça couvre l’**authentification** (SSO, MFA), l’**autorisation** (rôles/permissions), le cycle de vie des comptes (création/suppression), et souvent **PAM** pour les comptes privilégiés._

---

## EDR Telemetry

**Telemetry** = les **données collectées par l’agent EDR** et envoyées à la console.  
➡️ Vu comme la **“boîte noire”** d’un endpoint : tout ce qu’il faut pour **détecter** et **investiguer**.

---

### Types de telemetry

| Télémétrie                                 | À quoi ça sert (détection / enquête)                                        |
| ------------------------------------------ | --------------------------------------------------------------------------- |
| **Process executions / terminations**      | Repérer **parent-child** suspects, exe douteux, payloads                    |
| **Network connections**                    | Détecter **C2**, ports inhabituels, **exfiltration**, **lateral movement**  |
| **Command line activity** (CMD/PowerShell) | Voir commandes malveillantes, **PowerShell obfusqué** (souvent raté par AV) |
| **Files & folders modifications**          | Identifier staging, ransomware, drop de fichiers, changements anormaux      |
| **Registry modifications**                 | Traces de config/persistance Windows ; nombreuses modifs liées aux attaques |

---

### À retenir

- Plus il y a de **télémétrie**, plus l’EDR peut **corréler** et juger ce qui est malveillant.
    
- Les menaces avancées utilisent souvent des outils légitimes : **pris isolément** ça semble normal, **corrélé** ça révèle l’attaque.
    
- Pour l’analyste : permet de reconstruire **attack chain**, trouver la **root cause**, et bâtir une **timeline** complète.


---

## Detection & Response  Capabilities

#### Détection — techniques avancées

|Technique|Principe|Exemple|
|---|---|---|
|**Behavioral detection**|Analyse le **comportement** (pas juste signature) + abuse d’outils légitimes|`winword.exe` → `powershell.exe` (parent-child **anormal**)|
|**Anomaly detection**|Compare au **baseline** de l’endpoint ; écart = flag (possible **FP**)|Modif d’une **clé auto-start** inhabituelle|
|**IOC matching**|Match contre **Threat Intel** (hash, IP, domaines, etc.)|Hash d’un exe téléchargé = IOC connu → alerte|
|**MITRE ATT&CK mapping**|Map l’alerte à une **tactique/technique** (stade d’attaque)|Scheduled task → **Persistence / Scheduled Task/Job**|
|**Machine Learning**|Détecte des **patterns complexes** (chaîne d’actions)|Fileless + intrusions multi-stages détectées “en chaîne”|

---

#### Réponse — mécanismes

|Action|Objectif|Note/risque|
|---|---|---|
|**Isolate host**|**Containment** : couper l’endpoint du réseau pour stopper le lateral movement|Très efficace mais impact business possible|
|**Terminate process**|Neutraliser une action malveillante **sans isoler** la machine|Risque si process légitime → disruption|
|**Quarantine file**|Mettre un fichier en zone isolée **non exécutable**|Permet review puis restore/suppression|
|**Remote access / RTR**|Shell distant pour actions custom, collecte, scripts|Utile si actions “one-click” insuffisantes|
|**Artefacts collection**|Forensic / reporting légal sans accès physique|**Memory dump**, **event logs**, contenu de dossier, **registry hives**|

![[EDR (réponse - mécanisme).png]]

---

**À retenir :** EDR = détection plus moderne que l’AV (comportement, anomalie, ML) + vraie capacité de **réponse** (auto + manuel).

---

