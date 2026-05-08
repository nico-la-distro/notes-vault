## Log Analysis Basics

Logs = flux de messages horodatés enregistrant des événements.  
Log analysis = interprétation des logs pour comprendre l’activité d’une infrastructure.

### What Are Logs?

Logs = événements/transactions (applications, systèmes, réseau…).  
Contiennent :

- timestamp
- source
- détails de l’événement

```
Jul 28 17:45:02 10.10.0.4 FW-1: %WARNING% general: Unusual network activity detected from IP 10.10.0.15 to IP 203.0.113.25. Source Zone: Internal, Destination Zone: External, Application: web-browsing, Action: Alert.
```

Champs clés :

- `Jul 28 17:45:02` → timestamp
- `10.10.0.4` → source
- `%WARNING%` → sévérité

|Sévérité (ordre croissant)|
|---|
|Informational|
|Warning|
|Error|
|Critical|

- `Action: Alert` → notification déclenchée

Autres infos :

- src IP: `10.10.0.15`
- dst IP: `203.0.113.25`
- zone: Internal → External
- application: web-browsing

### Why Are Logs Important?

- Troubleshooting → identifier erreurs, réduire downtime
- Cyber Security → détecter intrusions, malware, accès non autorisés
- Threat Hunting → chercher anomalies, patterns, IOCs
- Compliance → conformité (GDPR, HIPAA, PCI DSS)

### Types of Logs

- Application Logs → état + erreurs app
- Audit Logs → actions utilisateurs / changements
- Security Logs → événements sécurité (login, firewall…)
- Server Logs → opérations serveur (error, access…)
- System Logs → kernel, boot, hardware
- Network Logs → connexions, trafic
- Database Logs → requêtes, modifications
- Web Server Logs → requêtes HTTP (IP, URL, code réponse)

Analyse croisée des logs = essentielle pour détection et investigation.

---

## Investigation Theory

Méthodologies + techniques pour construire une timeline cohérente et analyser les logs.

### Timeline

Timeline = représentation chronologique des événements.  
Permet :

- contextualisation
- compréhension de la séquence d’événements

Incident response :

- reconstruction de l’incident
- identification point initial de compromission
- analyse des TTPs

### Timestamp

Logs contiennent timestamps → gestion critique :

- fuseaux horaires
- formats différents

→ convertir en timezone unique

Ex :

- Splunk → conversion en temps UNIX (`_time`)
- affichage ensuite en timezone locale

→ corrélation fiable entre sources

### Super Timelines

Super timeline = timeline consolidée multi-sources

Inclut :

- system logs
- application logs
- network logs
- firewall logs

→ corrélation + détection patterns impossibles en analyse isolée

Outil :

- Plaso (Log2Timeline) → création automatique timeline unifiée
 [Plaso (Python Log2Timeline)](https://github.com/log2timeline/plaso)

### Data Visualization

Outils :

- Kibana
- Splunk

Fonction :

- transformer logs → visualisations
- détecter patterns / anomalies
- dashboards ("single pane of glass")

Ex :

- objectif → détecter failed logins
- source → logs d’authentification
- visuel → line chart
- filtre → période (ex: 7 jours)

![[intro to log analysis data vizualisation.png]]

### Log Monitoring and Alerting

Monitoring + alerting → détection proactive

SIEM :

- Splunk
- Elastic Stack

Alertes sur :

- failed logins multiples
- privilege escalation
- accès fichiers sensibles

→ notification immédiate

Process :

- définir rôles
- définir escalade selon sévérité

 [Splunk: Dashboards and Reports](https://tryhackme.com/jr/splunkdashboardsandreports)

### External Research and Threat Intel

Threat intel = indicateurs liés à attaquants :

- IP
- hash
- domain

Utilisation :

- rechercher IoC dans logs

Ex log :

```
54.36.149.64 - - [25/Aug/2023:00:05:36 +0000] "GET /admin HTTP/1.1"
```

Recherche :

```
grep "54.36.149.64" logfile.txt
```

→ corrélation avec feeds (ex: ThreatFox) pour identifier acteurs malveillants
 [ThreatFox](https://threatfox.abuse.ch/)

![[intro to log analysis threatfox.png]]

---

## Detection Engineering

Comprendre où sont les logs + reconnaître patterns = détection efficace.

### Common Log File Locations

Paths communs (variables selon config) :

|Composant|Type|Path|
|---|---|---|
|Nginx|access|`/var/log/nginx/access.log`|
|Nginx|error|`/var/log/nginx/error.log`|
|Apache|access|`/var/log/apache2/access.log`|
|Apache|error|`/var/log/apache2/error.log`|
|MySQL|error|`/var/log/mysql/error.log`|
|PostgreSQL|logs|`/var/log/postgresql/postgresql-{version}-main.log`|
|PHP|error|`/var/log/php/error.log`|
|Linux|system|`/var/log/syslog`|
|Linux|auth|`/var/log/auth.log`|
|iptables|firewall|`/var/log/iptables.log`|
|Snort|IDS|`/var/log/snort/`|

→ vérifier via documentation/config

### Common Patterns

Patterns = artefacts laissés dans les logs par activités malveillantes.

#### Abnormal User Behavior

Déviation du comportement normal utilisateur.

Détection :

- baseline + ML → anomalies

detection engines and machine learning algorithms to establish normal behavior patterns. Deviations from these patterns or baselines can then be alerted as potential security incidents. Some examples of these solutions include [_Splunk User Behavior Analytics (UBA)_(opens in new tab)](https://www.splunk.com/en_us/products/user-behavior-analytics.html), _[IBM QRadar UBA(opens in new tab)](https://www.ibm.com/docs/en/qradar-common?topic=app-qradar-user-behavior-analytics)_, and _[Azure AD Identity Protection(opens in new tab)](https://learn.microsoft.com/en-us/azure/active-directory/identity-protection/overview-identity-protection)_.

Indicateurs :

- multiples failed logins → brute force
- horaires inhabituels → accès suspect
- anomalies géographiques → compromission
- changements fréquents de mot de passe → prise de contrôle
- user-agent inhabituel → automatisation/malware

Ex :

- `Nmap Scripting Engine`
- `(Hydra)`

→ ajuster détection pour limiter faux positifs

### Common Attack Signatures

Signatures = patterns spécifiques d’attaques.

#### SQL Injection

Indicateurs :

- `'` `--` `#` `UNION`
- `WAITFOR DELAY` `SLEEP()`

```
10.10.61.21 - - [2023-08-02 15:27:42] "GET /products.php?q=books' UNION SELECT null, null, username, password, null FROM users-- HTTP/1.1" 200 3122
```

 A useful SQLi payload list to reference can be found [here(opens in new tab)](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/SQL%20Injection).

#### Cross-Site Scripting (XSS)

allow attackers to inject malicious scripts into web pages

Indicateurs :

- `<script>`
- `onerror`, `onclick`, `onerror`

```
10.10.19.31 - - [2023-08-04 16:12:11] "GET /products.php?search=<script>alert(1);</script> HTTP/1.1" 200 5153
```

A useful XSS payload list to reference can be found [here(opens in new tab)](https://github.com/payloadbox/xss-payload-list).

#### Path Traversal

Indicateurs :

- `../` `../../`
- accès fichiers sensibles (`/etc/passwd`)

Encodage :

 avoid detection by firewalls or monitoring tools

- `%2E` → `.`
- `%2F` → `/`

```
10.10.113.45 - - [2023-08-05 18:17:25] "GET /../../../../../etc/passwd HTTP/1.1" 200 505
```

A useful directory traversal payload list to reference can be found [here(opens in new tab)](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Directory%20Traversal/README.md)

---

## Automated vs. Manual Analysis

### Automated Analysis

Analyse via outils (ex: XPLG, SolarWinds Loggly) + IA/ML → détection patterns/tendances.

|Avantages|Désavantages|
|---|---|
|Gain de temps|Outils souvent coûteux|
|Détection automatique patterns|Faux positifs|
|Scalabilité|Manque détection inconnus|

### Manual Analysis

Analyse sans outils automatisés (ex: lecture logs, commandes Linux).

|Avantages|Désavantages|
|---|---|
|Faible coût|Très chronophage|
|Analyse approfondie|Risque de manquer événements|
|Réduction faux positifs||
|Analyse contextuelle||

---

## Log Analysis Tools: Command Line

(cf. /Documents/thm/intro_to_log_analysis/apache-1691435735822.log)

Command line = analyse rapide sans SIEM.  
Permet : lecture, filtrage, extraction, transformation.

### cat

Affiche tout le fichier.

```bash
cat apache.log
```

→ pas adapté aux gros fichiers

### less

Lecture paginée (navigation).

```bash
less apache.log
```

- scroll (flèches / PgUp / PgDn)
- quitter : `q`

### tail

Affiche fin du fichier.

```bash
tail apache.log
```

Options :

- `-n X` → X dernières lignes
- `-f` → suivi temps réel

```bash
tail -f -n 5 apache.log
```

→ monitoring live

### wc

Stats fichier : lignes / mots / caractères

```bash
wc apache.log
70 1562 14305 apache.log
```

- 70 lines
- 1562 words
- 14305 characters

### cut

Extraction champs (délimiteur).

```bash
cut -d ' ' -f 1 apache.log
```

options :
- -d = use DELIM instead of TAB for field delimiter
- -f -> we can change the field number to `-f 7` to extract the URLs and `-f 9` to extract the HTTP status codes.

|Champ|Contenu|
|---|---|
|1|IP|
|7|URL|
|9|status HTTP|

### sort

Tri données.
For example, to sort the list of returned IP addresses from the above `cut` command, we can run

```bash
cut -d ' ' -f 1 apache.log | sort -n
```

Options :

- `-n` → numérique (ordre croissant par défaut)
- `-r` → inverse (ajouter pour ordre décroissant)

### uniq

Supprime doublons (input trié).

```bash
cut -d ' ' -f 1 apache.log | sort -n | uniq
```

Compter occurrencesavec -c :

```bash
cut -d ' ' -f 1 apache.log | sort -n | uniq -c
1 221.90.64.76 
1 211.87.186.35 
1 203.78.122.88 
6 203.64.78.90 
...
```

→ détection IPs fréquentes

### sed

Transformation texte (substitution).
exemple : to replace all occurrences of "**31/Jul/2023**" with "**July 31, 2023**"

```bash
sed 's/31\/Jul\/2023/July 31, 2023/g' apache.log
203.0.113.42 - - [July 31, 2023:12:34:56 +0000] "GET /index.php HTTP/1.1"...
120.54.86.23 - - [July 31, 2023:12:34:57 +0000] "GET /contact.php HTTP/1...
```

- `\` -> is required to "escape" the forward slash in our pattern
- `sed` command _does not_ change the `apache.log` file directly; instead, it only outputs the modified version
- `-i` → modifie fichier (risque perte données)
- `>` to save the output to the original or another file.

### awk

Filtrage conditionnel par champs.

```bash
awk '$9 >= 400' apache.log
```

→ erreurs HTTP (>=400)

- `$9` field (which in this log example refers to the HTTP status codes), requiring it to be greater than or equal to `400`

it is highly encouraged to read more about their options and use cases [here(opens in new tab)](https://www.theunixschool.com/p/awk-sed.html).

### grep

Recherche patterns.

```bash
grep "admin" apache.log
```

Options :

- `-c` → count
- `-n` → numéro ligne
- `-v` → exclusion

```bash
grep -v "/index.php" apache.log | grep "203.64.78.90"
```

→ filtrage combiné

CLI = rapide, flexible, essentiel  
Limite → moins adapté gros volumes vs SIEM (Splunk, ELK)

 It is highly encouraged to read more about it on the official GNU manual page [here(opens in new tab)](https://www.gnu.org/software/grep/manual/grep.html)

---

## Log Analysis Tools: Regular Expressions

Regex = patterns pour rechercher, filtrer, extraire, transformer du texte.  
Utilisé en log analysis + parsing + SIEM.

https://tryhackme.com/room/catregex ->  fantastic resource for learning and practicing regex
### Regular Expressions for grep

Utilisation avec `grep -E` (pattern avancé).

Ex :

```
grep -E 'post=1[0-9]' apache-ex2.log
```

Pattern :

- `post=` → match literal
- `1[0-9]` → 10 → 19

→ filtrage précis

### Regular Expressions for Log Parsing

Parsing = transformer logs non structurés → champs exploitables.

Ex log :

```
126.47.40.189 - - [28/Jul/2023:15:30:45 +0000] "GET /admin.php HTTP/1.1"
```

Champs utiles :

- IP
- timestamp
- méthode HTTP
- URL
- user-agent

[RegExr(opens in new tab)](https://regexr.com/) is an online tool to help teach, build, and test regular expression patterns. To follow along, copy the above log entry and paste it into the "**Text**" section of the tool.

#### Extraction IP (regex)

```
\b([0-9]{1,3}\.){3}[0-9]{1,3}\b
```

Structure :

- `\b` → boundary
- `[0-9]{1,3}` → 1 à 3 digits
- `\.` → point
- `{3}` → répété 3 fois

→ extraction IPv4

![[intro to log analysis ip extraction.png]]

### Example: Logstash and Grok

Grok = parsing logs (Elastic / Logstash)  
Syntax :

```
%{SYNTAX:SEMANTIC}
```

Regex custom possible

Ex config :

```
filter {
  grok {
    match => { "message" => "(?<ipv4_address>\b([0-9]{1,3}\.){3}[0-9]{1,3}\b)" }
  }
}
```

→ extraction IP → champ `ipv4_address`

Regex = essentiel pour :

- filtrage précis
- parsing logs
- structuration données SIEM

The [Logstash room](https://tryhackme.com/jr/logstash) and [the official Grok documentation(opens in new tab)](https://www.elastic.co/guide/en/logstash/current/plugins-filters-grok.html) are fantastic resources for further exploring Logstash input and filter configurations!

---

## Log Analysis Tools: CyberChef

CyberChef = outil multi-fonctions (“Cyber Swiss Army Knife”) pour manipulation de données.

Fonctions :

- encoding / decoding
- encryption / hashing
- parsing logs / extraction

### Understanding CyberChef

Interface :

- Operations → actions disponibles
- Recipe → chaîne d’opérations
- Input → données source
- Output → résultat

Ex :

- input : `dHJ5aGFja21l`
- opération : decode base64
- output : `tryhackme`

Magic :

- détecte automatiquement encodage + opérations possibles

### Regex with CyberChef

Regex utilisable pour extraction.

Ex :

- objectif → extraire IPs d’un log SSH
- pattern :

```
\b([0-9]{1,3}\.){3}[0-9]{1,3}\b
```

Option :

- Output format → `List matches`

→ sortie = uniquement IPs (sans bruit)

### Uploading Files in CyberChef

- upload fichiers / dossiers
- support archives (`.zip`, `.tar.gz`)
- opérations intégrées pour extraction

→ pratique pour logs volumineux

---

## Sigma

Sigma = outil open-source (YAML) pour décrire/détecter événements dans logs.

Utilisation :

- détection événements
- création requêtes SIEM
- identification menaces

Ex règle :

```yaml
title: Failed SSH Logins
description: Searches sshd logs for failed SSH login attempts
status: experimental
author: CMNatic
logsource: 
    product: linux
    service: sshd

detection:
    selection:
        type: 'sshd'
        a0|contains: 'Failed'
        a1|contains: 'Illegal'
    condition: selection

falsepositives:
    - Users forgetting or mistyping their credentials
level: medium
```

|Clé|Description|
|---|---|
|title|objectif règle|
|description|détail|
|status|état (ex: experimental)|
|author|auteur|
|logsource|source logs|
|detection|conditions de détection|
|falsepositives|cas légitimes|
|level|sévérité|

→ intégration SIEM pour détection automatique

I recommend checking out the [Sigma](https://tryhackme.com/room/sigma) room on TryHackMe

### Yara

Yara = outil pattern matching (YAML-like) basé sur :

- strings
- regex
- patterns binaires

Utilisation :

- malware analysis
- log analysis

Ex règle :

```yara
rule IPFinder {
    meta:
        author = "CMNatic"
    strings:
        $ip = /([0-9]{1,3}\.){3}[0-9]{1,3}/ wide ascii
 
    condition:
        $ip
}
```

|Clé|Description|
|---|---|
|rule|nom règle|
|meta|métadonnées|
|strings|patterns recherchés|
|condition|condition déclenchement|

Exécution :

```
yara ipfinder.yar apache2.txt
```

→ déclenche si IP détectée

Extensions :

- multiples IPs
- ranges (subnet, ASN)
- IP en hex
- seuil occurrences
- corrélation avec actions spécifiques

check out the [Yara](https://tryhackme.com/room/yara) room on TryHackMe.

---

To expand your SIEM and centralized logging solution capabilities, visit the **[Advanced Splunk](https://tryhackme.com/module/advanced-splunk)** and **[Advanced ELK](https://tryhackme.com/module/advanced-elk)** modules.