## Benefits of SIEM for Analysts

### Centralisation

Tous les logs (réseau, cloud, endpoints, identity providers) centralisés dans une seule plateforme -> pas de navigation entre systèmes pendant une investigation.

|Sans SIEM|Avec SIEM|
|---|---|
|Connexion manuelle à chaque système (IPS, EDR...)|Tout accessible depuis une interface unique|
|Perte de temps à collecter les données|Investigation directe, vue globale immédiate|

### Correlation

Capacité à relier des événements séparés pour former une image complète.

Exemple : alerte IDS sur du network discovery (seulement une IP) -> corrélation avec Windows Event Logs / Sysmon -> identification de l'utilisateur, du device, de l'outil utilisé.

### Historical Events

Accès aux événements passés pour détecter des patterns ou des menaces antérieures non détectées.

Exemple : alerte login depuis un lieu inhabituel -> vérification des logs historiques pour déterminer si c'est un comportement récurrent ou une anomalie réelle.

---

## Log Sources Overview

### Host-Based Log Sources

Logs provenant des workstations et serveurs (web, SQL, DNS...). Quasi toutes les attaques impliquent des hosts -> source de données critique en investigation.

![[host-based_log_sources.png]]

### Network-Based Log Sources

Logs provenant des firewalls, routeurs, IDS/IPS. Donnent la visibilité sur les communications entre devices -> essentiels pour la corrélation.

![[network-based_log_sources.png]]

### Web-Based Log Sources

Logs des applications web. Les vulnérabilités web étant un vecteur d'entrée majeur -> monitoring quotidien indispensable en SOC.

Autres sources collectées par le SIEM : AWS, Azure, Entra ID, applications tierces.

![[web-based_log_sources.png]]

### Time Pitfalls

Les logs peuvent venir de systèmes en fuseaux horaires différents (UTC, heure locale, ou sans timezone). Le fuseau du SIEM peut différer de l'heure locale de l'analyste.

Exemple : analyste en UTC-2, SIEM normalisé en UTC+2 -> un event à 17h00 local apparaît à 21h00 dans le SIEM. Ce n'est pas un retard d'ingestion, c'est un décalage de configuration.

-> Toujours vérifier les fuseaux horaires avant d'analyser la chronologie d'un incident.

### Logs Normalisation

Les logs arrivent dans des formats variés (JSON, XML, plain text) avec des noms de champs différents selon la source. La normalisation convertit tout en une structure unique et cohérente dans le SIEM -> facilite la recherche, le filtrage et la corrélation.

---

## Windows Logs

Deux sources principales dans le SIEM : **WinEventLogs** et **Sysmon** (à installer/configurer séparément).

### Sysmon

Logs haute visibilité : exécution de processus, connexions réseau, injection de processus, modifications registre, création de fichiers.

**Malicious Process Execution** - EventCode 1

splunk

```splunk
index=winenv EventCode=1 *powershell* AND *EncodedCommand*
| table _time ComputerName ParentUser ParentImage ParentCommandLine Image CommandLine
```

**Suspicious Network Connection** - EventCode 3

splunk

```splunk
index=winenv EventCode=3 ComputerName=WINHOST05
| table _time ComputerName Image SourceIp SourcePort DestinationIp DestinationPort Protocol
```

-> Vérifier les IPs suspectes sur des plateformes de Threat Intelligence.

### WinEventLogs

Plus de 200 canaux de logs. Les plus utilisés en SOC :

#### Windows Security Logs

Détection : authentification, création/modification de comptes, accès fichiers/registre, exécution de processus, suppression de logs, changements de politiques de sécurité.

|EventCode|Signification|
|---|---|
|4720|Création de compte utilisateur|
|4722|Activation de compte utilisateur|

splunk

```splunk
index=winenv EventCode=4720 OR EventCode=4722
| table _time EventCode ComputerName Subject_Account_Name Target_Account_Name New_Account_Account_Name Keywords
```

#### Windows System Logs

Détection : services, activité système, erreurs. Utile pour identifier persistence ou privilege escalation via services.

|EventCode|Signification|
|---|---|
|7045|Création d'un service|
|7036|Démarrage/arrêt d'un service|

splunk

```splunk
index=winenv EventCode=7045 OR EventCode=7036 ComputerName=WINHOST05
| table _time EventCode ComputerName Service_Name Service_Account Service_File_Name Message
```

### Practice Scenario

Alerte : connexion réseau suspecte sur port 5678, host WIN-105.

splunk

```splunk
index=task4
```

#### Questions
##### Which IP address was the connection established with?

```splunk
index=task4 EventCode=3 ComputerName=WIN-105 DestinationPort=5678
```

filed DestinationIp -> 1 ip

**Answer** : 10.10.114.80

##### Which process initiated this suspicious connection?

same filter. in the filed list add the ProcessId 1460 to the tilter, then check for the image in the events

```splunk
index=task4 EventCode=3 ComputerName=WIN-105 DestinationPort=5678
```

![[log_analysis_siem_t4q2.png]]

**Answer** : SharePoInt.exe

##### What is the MD5 hash of the malicious process from the previous question?

clic on the image and add to the filter. then remove DestinationPort and change the EventCode to 1

```splunk
index=task4 EventCode=1 ComputerName=WIN-105 ProcessGuid="{c5d2b969-c41e-689d-dc02-000000002101}" "C:\\Windows\\Temp\\SharePoInt"
```

![[log_analysis_siem_t4q3.png]]

we can compare this hash to the others of the same executable and notice that's the only one with this hash

**Answer** : 770D14FFA142F09730B415506249E7D1

##### What is the name of the scheduled task that was created on the system?

first i checked for the processes launch by the malware along with its processid (1460) :

```splunk
index=task4 ParentProcessId=1460
```

then i find two cmd.exe launched by the malware, so i checked what was doing those commands along with their processids 

```splunk
index=task4 ParentProcessId=5844 OR ParentProcessId=700
```

then i saw that a schtasks was created

![[log_analysis_siem_t4q4.png]]

**Answer** : Office365 Install

---

## Linux Logs

Deux sources principales sur Linux dans le SIEM : `auth.log` et `syslog`.

|Source|Contenu|
|---|---|
|`auth.log`|Logins, sudo, SSH, escalade de privilèges|
|`syslog`|Services, crons, processus en arrière-plan|

### Authentication Logs

#### Unusual login activities

Recherche de tentatives SSH (succès + échecs) pour un utilisateur :

splunk

```splunk
index=linux source="auth.log" *ubuntu* process=sshd
| search "Accepted password" OR "Failed password"
```

-> Nombreux échecs suivis d'un succès = brute-force réussi -> escalader à L2.

#### Privilege Escalation behaviours

splunk

```splunk
index=linux source="auth.log" *su*
| sort + _time
```

-> Permet de détecter un passage vers root. `auth.log` seul ne suffit pas à déterminer le vecteur exact -> logs supplémentaires nécessaires.

### System Logs

#### Persistence Mechanisms

Détection de persistence via cron jobs ou scripts malveillants :

splunk

```splunk
index=linux sourcetype=syslog ("CRON" OR "cron")
| search ("python" OR "perl" OR "ruby" OR ".sh" OR "bash" OR "nc")
```

-> Exemples détectés : script `.sh` depuis `/tmp` exécuté toutes les 5 minutes, reverse shell Perl vers `10.10.101.12:9999`.

Autres outils courants en environnement réel : `auditd`, `osquery`.

### Practice Scenario

Alerte : création d'un utilisateur `remote-ssh` suspect sur un serveur Ubuntu.

splunk

```splunk
index=task5
```

#### Questions
##### What was the timestamp of the remote-ssh account creation? Answer Format Example: 2025-01-15 12:30:45

add this filter :

```splunk
index=task5 
|  search "useradd"
```

then we can see the useradd command

![[log_analsyis_siem_t5q1.png]]

**Answer** : 2025-08-12 09:52:57

##### Which user successfully escalated their privileges to root prior to the action from the first question?

add this filter :

```splunk
index=task5 
|  search "su"
```

check for the user who launched the su command

![[log_analysis_siem_t5q2.png]]

**Answer** : jack-brown

##### From which IP address did the user from the previous question successfully log in to the system?

add this filter :

```splunk
index=task5 source="auth.log" process=sshd user_name=jack-brown
```

go to the src_ip field, there is one ip : 

![[log_analysis_siem_t5q3.png]]

**Answer** : 10.14.94.82

##### How many failed login attempts occurred prior to this successful login?

add this filter :

```splunk
index=task5 source="auth.log" process=sshd user_name=jack-brown src_ip="10.14.94.82"
| search "Failed password"
```

then check for the failed attempts

![[log_analysis_siem_t5q4.png]]

**Answer** : 4

##### Which port is the persistence mechanism configured to connect to?

add this filter :

```splunk
index=task5 sourcetype=syslog ("CRON" OR "cron")
```

check for an ip:port pattern

![[log_analysis_siem_t5q5.png]]

**Answer** : 7654

---

## Web Application Logs

### Web Log Sources

Sources : Apache, Nginx, autres serveurs web.

|Type de log|Utilité|
|---|---|
|Access logs|Détection de scanning, DDoS, web shells, attaques web|
|Error logs|Failures et erreurs serveur|

### Brute Force Activity

Filtrer les POST vers la page de login, grouper par IP, seuil > 25 requêtes sur 5 min.

splunk

```splunk
index=* method=POST uri_path="/wp-login.php"
| bin _time span=5m
| stats values(referer_domain) as referer_domain values(status) as status values(useragent) as UserAgent values(uri_path) as uri_path count by clientip _time
| where count > 25
| table referer_domain clientip UserAgent uri_path count status
```

-> Vérifier le `UserAgent` : la présence de `Hydra` confirme un brute-force.

### Possible Web Shell

Rechercher des requêtes POST/GET avec status 200 vers des extensions exécutables, seuil > 2 requêtes.

splunk

```splunk
index=*
| search status=200 AND uri_path IN(*.php, *.phtm, *.asp, *.aspx, *.jsp, *.exe) AND (method=POST AND method=GET)
| stats values(status) as status values(useragent) as UserAgent values(method) as method
  values(uri) as uri values(clientip) as clientip count by referer_domain
| where count > 2
| table referer_domain count method status clientip UserAgent uri
```

-> Inspecter les URIs suspects (ex: `505.php`) de plus près.

### DDoS Activity

Status 503 (serveur surchargé), seuil > 100 000 requêtes sur 10 min.

splunk

```splunk
index=* status=503
| bin _time span=10m
| stats values(referer_domain) as referer_domain values(status) as status values(useragent) as UserAgent values(uri_path) as uri_path count by clientip _time
| where count > 100000
| table _time referer_domain clientip UserAgent uri_path count status
```

### Practice Scenario

Alerte : pic d'activité sur le serveur web.

splunk

```splunk
index=task6
```

#### Questions
##### Which URI path had the highest number of requests?

filter :

```splunk
index=task6
| stats count by uri_path
| sort - count
```

![[log_analysis_siem_t6q1.png]]

**Answer** : /wp-login.php

##### Which IP address was the source of the activity?

clic on the forst uri_path to get this filter :

```splunk
index=task6  uri_path="/wp-login.php"
```

then check de clientip in the fields

![[log_analysis_siem_t6q2.png]]

**Answer** : 10.10.243.134

##### How can this activity be classified?

according to the number of POST request :

**Answer** : Brute Force

##### Which tool did the threat actor use?

we can see the user agent that the threat actor used in the http request of the previous filter with client ip added

```splunk
index=task6  uri_path="/wp-login.php" clientip="10.10.243.134"
```

**Answer** : wpscan

---

## Log Analysis with SIEM - Résumé

### Pourquoi le SIEM

- Centralise tous les logs en une seule plateforme
- Corrèle des événements séparés pour former une image complète
- Permet d'accéder aux événements historiques pour détecter des patterns
- Attention aux décalages de fuseaux horaires entre systèmes et SIEM
- Normalisation automatique des formats (JSON, XML, plain text) -> structure unique

### Sources de logs

|Source|Outil/Fichier|Ce qu'on détecte|
|---|---|---|
|Windows|Sysmon (EC 1, 3)|Exécution de processus, connexions réseau|
|Windows|Security logs (EC 4720, 4722)|Création/activation de comptes|
|Windows|System logs (EC 7045, 7036)|Création/démarrage de services|
|Linux|`auth.log`|Brute-force SSH, escalade de privilèges|
|Linux|`syslog`|Cron jobs malveillants, reverse shells|
|Web|Access logs|Brute-force, web shells, DDoS|

### Patterns d'attaque couverts

- Brute-force SSH -> nombreux `Failed password` puis `Accepted password`
- Persistence Windows -> service malveillant sous SYSTEM depuis `Temp/`
- Persistence Linux -> script `.sh` en cron depuis `/tmp/`
- Web shell -> requêtes POST/GET status 200 vers extensions exécutables
- DDoS -> status 503, volume massif de requêtes sur courte durée
- Privilege escalation -> `su` vers root, service sous SYSTEM

### Réflexes clés

- Toujours vérifier le `UserAgent` (révèle l'outil utilisé : Hydra, curl...)
- Checker les IPs suspectes sur des plateformes de Threat Intelligence
- Un log seul ne suffit pas -> toujours croiser les sources
- Brute-force réussi + escalade -> escalader à L2

