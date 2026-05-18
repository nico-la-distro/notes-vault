- **SIEM (Security Information and Event Management)** : solution **centrale** utilisée par un analyste SOC pour la **surveillance sécurité**.

- Objectif clé : comprendre comment les équipements d’un réseau **génèrent des logs** et pourquoi il faut une solution **centralisée** pour :
    - **collecter** les logs,
    - les **normaliser** (format commun),
    - les **corréler** (lier plusieurs événements pour détecter une attaque / anomalie).

## **Logs everywhere, answer nowhere**

- Dans un réseau, **plein d’équipements** (endpoints Windows/Linux, serveurs, routeur, etc.) communiquent entre eux et avec Internet.

- Tous ces équipements génèrent en continu des **logs** (= traces d’activités) → utiles pour 
    - détecter du **malveillant**
    - faire du **troubleshooting**

### **Types de log sources**

|Catégorie|Ça observe…|Exemples d’équipements|Exemples de logs|
|---|---|---|---|
|**Host-centric** (côté machine)|Événements **sur le host**|Windows, Linux, serveurs|accès fichier, tentative d’auth, exécution de processus, modif registre, exécution PowerShell|
|**Network-centric** (côté réseau)|Événements liés aux **communications**|firewall, IDS/IPS, routeur, VPN…|connexions SSH, transferts FTP, trafic web, accès via VPN, partage de fichiers réseau|

### **“Answers Nowhere” : pourquoi analyser des logs isolés est compliqué**

| Problème                     | Ce que ça implique concrètement                                                                                               |
| ---------------------------- | ----------------------------------------------------------------------------------------------------------------------------- |
| **Trop de sources**          | Des centaines d’événements/sec → impossible de tout vérifier device par device                                                |
| **Pas de centralisation**    | Il faut se connecter à chaque machine (SSH/RDP…) → **lent** et inefficace en incident                                         |
| **Contexte limité**          | Un log seul peut paraître normal, mais **corrélé** avec d’autres il révèle l’attaque (ex : accès fichier + mouvement latéral) |
| **Analyse limitée (humain)** | Volume énorme → on rate forcément des signaux importants                                                                      |
| **Formats différents**       | Logs hétérogènes → il faut connaître plein de formats → friction + erreurs                                                    |

## **Fonctionnalités SIEM**

|Fonction|Ce que ça fait|Pourquoi c’est utile|
|---|---|---|
|**Centralized Log Collection**|Centralise les logs (endpoints, serveurs, firewalls…) via **agents** légers ou **APIs**|Évite de se connecter machine par machine (SSH/RDP)|
|**Normalization of Logs**|**Parsing** = découper un log en champs ; **Normalization** = convertir tous les logs en un format cohérent|Facilite lecture, recherche, comparaison entre sources|
|**Correlation of Logs**|Relie des événements de sources différentes pour détecter des **patterns**|Un comportement “normal” isolé peut devenir suspect une fois corrélé|
|**Real-time Alerting**|Déclenche des **alertes** quand une règle match ; règles par défaut + règles custom par les analystes|Détection + notification rapide → investigation dans le SIEM|
|**Dashboards & Reporting**|Visualise des **insights actionnables** (dashboards par défaut + custom)|Suivi SOC : tendances, santé, alertes, métriques|

**Corrélation : exemple à retenir (ultra synthèse)**
**Sur 5 min :**
- login VPN depuis une IP jamais vue
- accès à des docs sur partage
- exécution PowerShell
- connexion réseau sortante
➡️ Pris séparément : “ok”.  
➡️ Corrélés : possible **compromission VPN + exfiltration de données**.

## Dashboards : exemples de contenus

- Alert highlights
- Notifications système / health alerts
- tentatives de login échouées
- volume d’événements ingérés
- règles déclenchées
- top domaines visités

## **Log source**

|Source|Où voir / où sont stockés les logs|Ce que ça apporte au SOC|
|---|---|---|
|**Windows**|**Event Viewer** (Observateur d’événements) avec des **Event ID** uniques par type d’événement|Suivi facile (ID = repère rapide), logs envoyés au SIEM pour visibilité globale|
|**Linux**|Fichiers dans **/var/log/**|Logs systèmes + services (auth, cron, kernel, web…) ingérés dans le SIEM|
|**Web server (Apache)**|Souvent **/var/log/apache/** ou **/var/log/httpd/**|Monitoring des **requêtes/réponses** pour détecter tentatives d’attaques web|

### **Linux : emplacement important**

| Chemin                                  | Contenu typique                    |
| --------------------------------------- | ---------------------------------- |
| `/var/log/httpd`                        | requêtes HTTP + réponses + erreurs |
| `/var/log/cron`                         | événements liés aux **cron jobs**  |
| `/var/log/auth.log` / `/var/log/secure` | logs d’**authentification**        |
| `/var/log/kern`                         | événements **kernel**              |
**cron** = command run on notice = (**commande exécutée à date**)

---
### Logs Web (Apache) : ce qu’on voit souvent

- IP source
- timestamp
- méthode + URL (ex: `GET /...`)
- code HTTP (200, 404, 500…)
- user-agent (navigateur / curl…)
➡️ Très utile pour repérer scans, exploitation (`/cgi-bin/...`), bots, etc.

---
### **Log ingestion (comment ça arrive dans SIEM)**

|Méthode|Principe|Quand c’est utilisé|
|---|---|---|
|**Agent / Forwarder**|petit agent sur l’endpoint qui collecte + envoie au SIEM (Splunk = “forwarder”)|le plus courant pour endpoints/serveurs|
|**Syslog**|protocole standard pour envoyer des logs en temps réel vers un collecteur central|très utilisé pour Linux, web servers, équipements réseau|
|**Manual Upload**|import de données “offline” pour analyse rapide (Splunk, ELK…)|investigations ponctuelles, datasets historiques|
|**Port Forwarding**|le SIEM écoute un port, les endpoints envoient dessus|quand on veut un flux direct vers une instance SIEM|

## **Alerting process and Analysis**

- Un SIEM **détecte** en appliquant des **detection rules** sur des logs **corrélés**.

- Une **règle de détection** = une **expression logique** (conditions) qui, si vraie, **déclenche une alerte**.

- Ces règles s’appuient sur des **paires champ/valeur** (field-value pairs) → d’où l’importance des logs **normalisés**.

**exemples**

|Condition|Alerte|
|---|---|
|5 échecs de login en 10 secondes|Multiple Failed Login Attempts|
|Login réussi après plusieurs échecs|Successful Login After Multiple Login Attempts|
|USB branché (si interdit par la politique)|USB Plugged In|
|Trafic sortant > 25 MB (selon policy)|Potential Data Exfiltration Attempt|

---

### **Construction d'un règle : 2 use cases**

|Use case|Signal (source/champs)|Règle (logique)|Alerte|
|---|---|---|---|
|**1. Effacement des logs**|Windows **Event ID 104** (clear/remove event logs)|`LogSource = WinEventLog` **AND** `EventID = 104`|**Event Log Cleared**|
|**2. Commande whoami**|Process execution : **Event ID 4688** + `NewProcessName` contient `whoami`|`LogSource = WinEventLog` **AND** `EventCode = 4688` **AND** `NewProcessName CONTAINS "whoami"`|**WHOAMI Command Execution Detected**|

➡️ Point clé : sans **normalisation**, ces champs ne seraient pas fiables/consistants → règles moins efficaces.

**normalisation / normalized** = **convertir des logs de sources différentes en un format et des champs standardisés**, pour pouvoir les **rechercher, corréler et détecter** de façon fiable dans un SIEM. **(windows synthaxe != Linux synthaxe)**

---

## Investigation d’alerte (workflow)

1. Surveille via **dashboards** (vue synthèse)
2. Alerte déclenchée → analyser les **events/flows liés**.
3. Vérifier la **règle** : quelles conditions ont match.
4. Décider : **True Positive** vs **False Positive**.

---

### **Actions possibles après analyse**

|Résultat|Actions typiques|
|---|---|
|**False Positive**|**tuner** la règle pour réduire les faux positifs|
|**True Positive**|investigation approfondie|
|Besoin de contexte|contacter le **propriétaire de l’asset**|
|Suspicion confirmée|**isoler** l’hôte infecté|
|Menace externe identifiée|**bloquer** l’IP suspecte|

---

## **Suite : - [Junior Security Analyst Intro](https://tryhackme.com/room/jrsecanalystintrouxo), [Splunk: The Basics](https://tryhackme.com/room/splunk101), [Incident Handling with Splunk](https://tryhackme.com/room/splunk201), [Benign](https://tryhackme.com/room/benign), [Investigating with Splunk](https://tryhackme.com/room/investigatingwithsplunk), [Investgating with ELK](https://tryhackme.com/room/investigatingwithelk101), [ItsyBitsy](https://tryhackme.com/room/itsybitsy)
