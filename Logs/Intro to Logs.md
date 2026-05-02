## Introduction

Identifier activités malveillantes via **logs** → preuves d’événements passés (intrusion, actions).

Logs = archive d’activités → améliore sécurité et protection des assets.

### Various types of digital logs

Comprendre les logs → détecter patterns + menaces.

Analyse manuelle difficile (volume élevé) → nécessite outils + techniques.

Analyse de logs :

- interprétation d’événements passés
- source fiable de preuve
- détection rapide d’incidents

---

## Expanding Perspectives: Logs as Evidence of Historical Activity

Logs = traces d’activités système (auth, accès, erreurs, réseau).

Analogie : anneaux d’arbre → historique.

Chaque interaction → log.

### In the Heart of Data: Logs

Logs = historique système (événements).

Exemples :

- login
- accès fichiers
- erreurs
- connexions réseau
- modifications système

### Components of a digital log

Contenu typique d’une entrée de log :

|Élément|Description|
|---|---|
|Timestamp|date/heure|
|Source|système/app|
|Event type|type d’événement|
|Details|user, IP, etc.|

Stockage :

- fichiers de logs (agrégés)
- taille élevée (croissance rapide)

### The True Power of Logs: Contextual Correlation

Un log seul → faible valeur  
Corrélation de logs → analyse complète

Questions clés :

|Question|Objectif|
|---|---|
|What|événement|
|When|moment|
|Where|origine|
|Who|acteur|
|Success|succès/échec|
|Result|impact|

#### Exemple

|Question|Réponse|
|---|---|
|What|accès GitLab|
|When|08/09/2023 22:10|
|Where|IP 10.10.133.168|
|Who|User-Agent identifié|
|Success|accès + web shell|
|Result|RCE + post-exploitation|

Logs → reconstruction complète d’un incident.

---

## Types, Formats, and Standards

### Log Types

Logs couvrent ~80% des cas usuels.

|Type|Contenu|
|---|---|
|Application Logs|status, erreurs|
|Audit Logs|conformité, actions|
|Security Logs|login, permissions, firewall|
|Server Logs|system, error, access|
|System Logs|kernel, boot, hardware|
|Network Logs|trafic, connexions|
|Database Logs|requêtes, updates|
|Web Server Logs|requêtes HTTP, codes|

### Log Formats

Format = structure + organisation des logs.

#### Semi-structured Logs

Structure partielle + texte libre.

**Syslog** : A widely adopted logging protocol for system and network logs.

```bash
damianhall@WEBSRV-02:~/logs$ cat syslog.txt 

May 31 12:34:56 WEBSRV-02 CRON[2342593]: (root) CMD ([ -x /etc/init.d/anacron ] && if [ ! -d /run/systemd/system ]; then /usr/sbin/invoke-rc.d anacron start >/dev/null; fi)
```

**Windows EVTX** : Proprietary Microsoft log for Windows systems.

```powershell
PS C:\WINDOWS\system32> Get-WinEvent -Path "C:\Windows\System32\winevt\Logs\Application.evtx"

ProviderName: Microsoft-Windows-Security-SPP

 TimeCreated Id LevelDisplayName Message 
 ----------- -- ---------------- ------- 
 31/05/2023 17:18:24 16384 Information Successfully scheduled Software Protection service for re-start 
 31/05/2023 17:17:53 16394 Information Offline downlevel migration succeeded.
```

#### Structured Logs

Format strict → parsing facile.

**CSV** : Comma-Separated Values (CSV) and Tab-Separated Values (TSV) are formats often used for tabular data.

```bash
damianhall@WEBSRV-02:~/logs$ cat log.csv
 
"time","user","action","status","ip","uri" 
"2023-05-31T12:34:56Z","adversary","GET",200,"34.253.159.159","http://gitlab.swiftspend.finance:80/"
```

**JSON** : Known for its readability and compatibility with modern programming languages.

```bash
damianhall@WEBSRV-02:~/logs$ cat log.json 

{"time": "2023-05-31T12:34:56Z", "user": "adversary", "action": "GET", "status": 200, "ip": "34.253.159.159", "uri": "http://gitlab.swiftspend.finance:80/"}
```

**W3C ELF** : Defined by the World Wide Web Consortium (W3C), customizable for web server logging. It is typically used by Microsoft Internet Information Services (IIS) Web Server.

```bash
damianhall@WEBSRV-02:~/logs$ cat elf.log 

#Version: 1.0 

#Fields: date time c-ip c-username s-ip s-port cs-method cs-uri-stem sc-status 

31-May-2023 13:55:36 34.253.159.159 adversary 34.253.127.157 80 GET /explore 200
```

**XML** : Flexible and customizable for creating standardized logging formats.

```bash
damianhall@WEBSRV-02:~/logs$ cat log.xml 

<log><time>2023-05-31T12:34:56Z</time><user>adversary</user><action>GET</action><status>200</status><ip>34.253.159.159</ip><url>https://gitlab.swiftspend.finance/</url></log>
```

#### Unstructured Logs

Texte libre → parsing difficile.

**CLF (Apache)** :  A standardized web server log format for client requests. It is typically used by the Apache HTTP Server by default.

```bash
damianhall@WEBSRV-02:~/logs$ cat clf.log 

34.253.159.159 - adversary [31/May/2023:13:55:36 +0000] "GET /explore HTTP/1.1" 200 4886
```

**Combined (Nginx)** : An extension of CLF, adding fields like referrer and user agent. It is typically used by Nginx HTTP Server by default.

```bash
damianhall@WEBSRV-02:~/logs$ cat combined.log 

34.253.159.159 - adversary [31/May/2023:13:55:36 +0000] "GET /explore HTTP/1.1" 200 4886 "http://gitlab.swiftspend.finance/" "Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:109.0) Gecko/20100101 Firefox/115.0"
```

Note :

- formats custom possibles → parsing spécifique

### Log Standards

Standard = règles (génération, transport, stockage, rétention).

| Standard                       | Objectif                          |
| ------------------------------ | --------------------------------- |
| CEE                            | structure commune logs            |
| OWASP Logging Cheat Sheet      | bonnes pratiques logging sécurité |
| Syslog                         | standard transmission logs        |
| NIST SP 800-92                 | gestion logs                      |
| Azure Monitor Logs             | logs Azure                        |
| Google Cloud Logging           | logs GCP                          |
| Oracle Cloud Logging           | logs OCI                          |
| Virginia Tech Logging Standard | guidelines audit/log review       |


- **[Common Event Expression (CEE):(opens in new tab)](https://cee.mitre.org/)** This standard, developed by MITRE, provides a common structure for log data, making it easier to generate, transmit, store, and analyse logs.
- **[OWASP Logging Cheat Sheet:(opens in new tab)](https://cheatsheetseries.owasp.org/cheatsheets/Logging_Cheat_Sheet.html)** A guide for developers on building application logging mechanisms, especially related to security logging.
- **[Syslog Protocol:(opens in new tab)](https://datatracker.ietf.org/doc/html/rfc5424)** Syslog is a standard for message logging, allowing separation of the software that generates messages from the system that stores them and the software that reports and analyses them.
- **[NIST Special Publication 800-92:(opens in new tab)](https://nvlpubs.nist.gov/nistpubs/Legacy/SP/nistspecialpublication800-92.pdf)** This publication guides computer security log management.
- **[Azure Monitor Logs:(opens in new tab)](https://learn.microsoft.com/en-us/azure/azure-monitor/logs/data-platform-logs)** Guidelines for log monitoring on Microsoft Azure.
- **[Google Cloud Logging:(opens in new tab)](https://cloud.google.com/logging/docs)** Guidelines for logging on the Google Cloud Platform (GCP).
- **[Oracle Cloud Infrastructure Logging:(opens in new tab)](https://docs.oracle.com/en-us/iaas/Content/Logging/Concepts/loggingoverview.htm)** Guidelines for logging on the Oracle Cloud Infrastructure (OCI).
- **[Virginia Tech - Standard for Information Technology Logging:(opens in new tab)](https://it.vt.edu/content/dam/it_vt_edu/policies/Standard_for_Information_Technology_Logging.pdf)** Sample log review and compliance guideline.

---

## Collection, Management, and Centralisation

### Log Collection

Collecte = agrégation de logs (serveurs, réseau, apps, DB).

Synchronisation temps essentielle → cohérence chronologique :

```bash
root@WEBSRV-02:~ ntpdate pool.ntp.org 
12 Aug 21:03:44 ntpdate[2399365]: adjust time server 85.91.1.180 offset 0.000060 sec 
root@WEBSRV-02:~ date 
Saturday, 12 August, 2023 09:04:55 PM UTC
```

Processus :

|Étape|Action|
|---|---|
|Identify Sources|sources logs|
|Choose Collector|outil collecte|
|Configure|NTP + événements|
|Test|validation collecte|

_NTP :  **Network Time Protocol**_

### Log Management

Objectif : stockage sécurisé + accès rapide.

|Étape|Action|
|---|---|
|Storage|stockage + rétention|
|Organisation|classification|
|Backup|sauvegarde|
|Review|vérification|

Approche hybride → conserver + filtrer.

### Log Centralisation

Centralisation → accès + analyse + réponse rapide.

|Étape|Action|
|---|---|
|System|ex: Elastic, Splunk|
|Integrate|connecter sources|
|Monitoring|alertes temps réel|
|Incident Integration|intégration IR|

### Practical Activity: Log Collection with rsyslog

Objectif : rediriger logs `sshd` → fichier dédié.

#### Vérification

```bash
sudo systemctl status rsyslog
```

#### Configuration

```bash
nano /etc/rsyslog.d/98-websrv-02-sshd.conf
```

```bash
$FileCreateMode 0644
:programname, isequal, "sshd" /var/log/websrv-02/rsyslog_sshd.log
```

#### Redémarrage

```bash
sudo systemctl restart rsyslog
```

#### Test

```bash
ssh localhost
```

Vérifier :

```bash
cat /var/log/websrv-02/rsyslog_sshd.log
```

Note :

If remote forwarding of logs is not configured, tools such as `scp` / `rsync`, among others, can be utilised for the manual collection of logs.

![[intro to logs configure log file.png]]

---

## Storage, Retention, and Deletion

### Log Storage

Stockage :

- local
- centralisé
- cloud

Facteurs :

|Facteur|Impact|
|---|---|
|Security|conformité|
|Accessibility|rapidité accès|
|Capacity|volume|
|Cost|budget|
|Compliance|obligations légales|
|Retention|durée stockage|
|Disaster Recovery|disponibilité|

### Log Retention

Stockage limité → compromis coût / utilité.

|Type|Durée|Accès|
|---|---|---|
|Hot|3–6 mois|rapide|
|Warm|6 mois–2 ans|modéré|
|Cold|2–5 ans|lent|

### Log Deletion

Suppression contrôlée :

- éviter perte de données utiles
- backup avant suppression

Objectifs :

- réduire taille logs
- conformité (ex: GDPR)
- contrôler coûts

### Best Practices: Log Storage, Retention and Deletion

- définir politiques (business + légal)
- réviser régulièrement
- automatiser processus
- chiffrer logs sensibles
- backups réguliers

### Practical Activity: Log Management with logrotate

`logrotate`, a tool that automates log file rotation, compression, and management, ensuring that log files are handled systematically. It allows automatic rotation, compression, and removal of log files. As an example, here's how we can set it up for `/var/log/websrv-02/rsyslog_sshd.log`

Objectif : rotation + compression + suppression automatique.

#### Configuration

```bash
sudo nano /etc/logrotate.d/98-websrv-02_sshd.conf
```

```bash
/var/log/websrv-02/rsyslog_sshd.log {
    daily
    rotate 30
    compress
    lastaction
        DATE=$(date +"%Y-%m-%d")
        echo "$(date)" >> "/var/log/websrv-02/hashes_"$DATE"_rsyslog_sshd.txt"
        for i in $(seq 1 30); do
            FILE="/var/log/websrv-02/rsyslog_sshd.log.$i.gz"
            if [ -f "$FILE" ]; then
                HASH=$(/usr/bin/sha256sum "$FILE" | awk '{ print $1 }')
                echo "rsyslog_sshd.log.$i.gz "$HASH"" >> "/var/log/websrv-02/hashes_"$DATE"_rsyslog_sshd.txt"
            fi
        done
        systemctl restart rsyslog
    endscript
}
```

#### Exécution

```bash
sudo logrotate -f /etc/logrotate.d/98-websrv-02_sshd.conf
```

![[intro to log logrotate.png]]

---

## Hands-on Exercise: Log analysis process, tools, and techniques

### Log Analysis Process

Pipeline :

| Étape          | Description                                                                          |
| -------------- | ------------------------------------------------------------------------------------ |
| Data Sources   | systèmes/applications générant des logs (origine des événements)                     |
| Parsing        | extraction des données depuis logs bruts → champs exploitables malgré formats variés |
| Normalisation  | standardisation des logs (format unique) → comparaison multi-sources                 |
| Sorting        | tri (temps, source, type, sévérité) → identification patterns / anomalies            |
| Classification | catégorisation (type, sévérité, source) → filtrage et focus                          |
| Enrichment     | ajout de contexte (geo, user, threat intel, autres sources) → meilleure analyse      |
| Correlation    | liaison entre logs → détection relations, patterns, attaques                         |
| Visualisation  | représentation graphique (graph, timeline) → lecture rapide des tendances            |
| Reporting      | synthèse structurée → support décision, conformité, suivi                            |
### Log Analysis Tools

- Security Information and Event Management (SIEM) tools such as Splunk or Elastic Search can be used for complex log analysis tasks.

- However, in scenarios where immediate data analysis is needed, such as during incident response, Linux-based systems can employ default tools like `cat`, `grep`, `sed`, `sort`, `uniq`, and `awk`, along with `sha256sum` for hashing log files. 

- Windows-based systems can utilise [EZ-Tools(opens in new tab)](https://ericzimmerman.github.io/#!index.md) and the default cmdlet `Get-FileHash` for similar purposes. These tools enable rapid parsing and analysis, which suits these situations.

Important :

- hasher logs → intégrité (preuve légale)

### Log Analysis Techniques

|Technique|Description|
|---|---|
|Pattern Recognition|identification de séquences / comportements récurrents → détection activité normale ou suspecte|
|Anomaly Detection|détection d’écarts par rapport au comportement attendu → identification précoce incidents / attaques|
|Correlation Analysis|lien entre logs → compréhension relations, dépendances, causes|
|Timeline Analysis|analyse temporelle → tendances, périodicité, évolution|
|Machine Learning and AI|automatisation (classification, enrichment) + détection avancée + prédiction|
|Visualisation|représentation graphique → identification rapide patterns / anomalies|
|Statistical Analysis|analyse quantitative (régression, tests) → validation hypothèses / décisions data-driven|

### Working with Logs: Practical Application

Deux approches d’analyse des logs :

|Approche|Description|Objectif|
|---|---|---|
|Raw logs (non parsés)|accès direct aux fichiers via Log Viewer|analyse rapide sans traitement préalable|
|Parsed & consolidated logs|logs transformés et fusionnés via outils Unix|analyse structurée et exploitable|

Conclusion :  
→ flexibilité selon besoin (inspection rapide vs analyse approfondie)

#### Unparsed Raw Log Files

Accès direct via Log Viewer (URL avec chemins encodés) :

```yaml
http://10.128.138.86:8111/log?log=%2Fvar%2Flog%2Fgitlab%2Fnginx%2Faccess.log&log=%2Fvar%2Flog%2Fwebsrv-02%2Frsyslog_cron.log&log=%2Fvar%2Flog%2Fwebsrv-02%2Frsyslog_sshd.log&log=%2Fvar%2Flog%2Fgitlab%2Fgitlab-rails%2Fapi_json.log
```

Usage :

- lecture directe des logs bruts
- pas de transformation
- utile pour inspection rapide

#### Parsed and Consolidated Log File

To create a parsed and consolidated log file, you can use a combination of Unix tools like `cat`, `grep`, `sed`, `sort`, `uniq`, and `awk`. Here's a step-by-step guide:

1. Use `awk` and `sed` to normalize the log entries to the desired format. For this example, we will sort by date and time:
    
    ```yaml
    # Process nginx access log
    awk -F'[][]' '{print "[" $2 "]", "--- /var/log/gitlab/nginx/access.log ---", "\"" $0 "\""}' /var/log/gitlab/nginx/access.log  | sed "s/ +0000//g" > /tmp/parsed_consolidated.log
    
    # Process rsyslog_cron.log
    awk '{ original_line = $0; gsub(/ /, "/", $1); printf "[%s/%s/2023:%s] --- /var/log/websrv-02/rsyslog_cron.log --- \"%s\"\n", $2, $1, $3, original_line }' /var/log/websrv-02/rsyslog_cron.log >> /tmp/parsed_consolidated.log
    
    # Process rsyslog_sshd.log
    awk '{ original_line = $0; gsub(/ /, "/", $1); printf "[%s/%s/2023:%s] --- /var/log/websrv-02/rsyslog_sshd.log --- \"%s\"\n", $2, $1, $3, original_line }' /var/log/websrv-02/rsyslog_sshd.log >> /tmp/parsed_consolidated.log
    
    # Process gitlab-rails/api_json.log
    awk -F'"' '{timestamp = $4; converted = strftime("[%d/%b/%Y:%H:%M:%S]", mktime(substr(timestamp, 1, 4) " " substr(timestamp, 6, 2) " " substr(timestamp, 9, 2) " " substr(timestamp, 12, 2) " " substr(timestamp, 15, 2) " " substr(timestamp, 18, 2) " 0 0")); print converted, "--- /var/log/gitlab/gitlab-rails/api_json.log ---", "\""$0"\""}' /var/log/gitlab/gitlab-rails/api_json.log >> /tmp/parsed_consolidated.log
    ```
    
2. **Optional:** Use `grep` to filter specific entries:
    
    ```yaml
    grep "34.253.159.159" /tmp/parsed_consolidated.log > /tmp/filtered_consolidated.log
    ```
    
3. Use `sort` to sort all the log entries by date and time:
    
    ```yaml
    sort /tmp/parsed_consolidated.log > /tmp/sort_parsed_consolidated.log
    ```
    
4. Use `uniq` to remove duplicate entries:
    
    ```yaml
    uniq /tmp/sort_parsed_consolidated.log > /tmp/uniq_sort_parsed_consolidated.log
    ```
    

You can now access the parsed and consolidated log file through the [Log Viewer(opens in new tab)](https://github.com/sevdokimov/log-viewer) tool using the following URL:

```yaml
http://10.128.138.86:8111/log?path=%2Ftmp%2Funiq_sort_parsed_consolidated.log
```


![[intro to log Log Viewer.png]]
  

**NOTE:** You can access the URL using the AttackBox or VM browser. However, please be aware that Firefox on the VM may take a few minutes to boot up.

