- Learn how to properly investigate alerts in a SOC environment.
- Understand how to investigate brute-force attacks on Linux systems.
- Discover the persistence mechanism on Windows systems.
- Analyse a web shell on a vulnerable web server.
- Learn how to investigate alerts for three given scenarios using Splunk.

---

## Initial Access Alert

### Alert Scenario

|Champ|Valeur|
|---|---|
|Alert Name|Brute Force Activity Detection|
|Time|17/09/2025 9:00:21 AM|
|Target Host|tryhackme-2404|
|Source IP|10.10.242.248|

Index Splunk : `linux-alert`

### Investigating the Alert

IP locale -> attaquant déjà dans le réseau (VPN compromis ou autre). Heure normale (9h) -> pas d'anomalie horaire.

**Recherche 1 - Activité login depuis l'IP :**

spl

```spl
index="linux-alert" sourcetype="linux_secure" 10.10.242.248 
| search "Accepted password for" OR "Failed password for" OR "Invalid user"
| sort + _time
```

Résultat : volume élevé d'événements + tentatives sur des utilisateurs inexistants (enumération).

![[alert_triage_with_splunk_login.png]]

**Recherche 2 - Nombre de tentatives par user :**

spl

```spl
index="linux-alert" sourcetype="linux_secure" 10.10.242.248
| rex field=_raw "^\d{4}-\d{2}-\d{2}T[^\s]+\s+(?<log_hostname>\S+)"
| rex field=_raw "sshd\[\d+\]:\s*(?<action>Failed|Accepted)\s+\S+\s+for(?: invalid user)? (?<username>\S+) from (?<src_ip>\d{1,3}(?:\.\d{1,3}){3})"
| eval process="sshd"
| stats count values(src_ip) as src_ip values(log_hostname) as hostname values(process) as process by username
```

Résultat : `john.smith` -> 503 tentatives -> brute force confirmé.

**Recherche 3 - Succès de connexion :**

spl

```spl
index="linux-alert" sourcetype="linux_secure" 10.10.242.248
| rex field=_raw "^\d{4}-\d{2}-\d{2}T[^\s]+\s+(?<log_hostname>\SD+)"
| rex field=_raw "sshd\[\d+\]:\s*(?<action>Failed|Accepted)\s+\S+\s+for(?: invalid user)? (?<username>\S+) from (?<src_ip>\d{1,3}(?:\.\d{1,3}){3})"
| eval process="sshd"
| stats count values(action) values(src_ip) as src_ip values(log_hostname) as hostname values(process) as process by username
```

Résultat : login `Accepted` uniquement pour `john.smith` -> **True Positive** -> escalade SOC L2 + IR.

### Next Investigation Steps

Questions ouvertes pour SOC L2 / IR :

- Pourquoi l'IP source est locale ? Depuis combien de temps l'attaquant est dans le réseau ?
- Comment l'attaquant a obtenu les usernames ?
- Que s'est-il passé après l'accès à `tryhackme-2404` ?

-> Containment & remediation par l'équipe IR.

### Questions
#### How many failed login attempts were made on the user john.smith?

filter :

```splunk
index="linux-alert" user_name="john.smith"
| search "Failed password"
```

**Answer** : 500

#### What was the duration of the brute force attack in minutes?

filters : 

```splunk
index="linux-alert" user_name="john.smith"
| search "Failed password"
| sort + _time
```

first result = 9:01

```splunk
index="linux-alert" user_name="john.smith"
| search "Failed password"
| sort - _time
```

first result : 9:06

**Answer** : 5

#### What username was the attacker able to privilege escalate to?

filter :

```splunk
index="linux-alert" user=root | sort + _time | search "su" src_user_name="john.smith"
```

**Answer** : root

#### What is the name of the user account created by the attacker for persistence?

filter :

```splunk
index="linux-alert"
| search "adduser"
```

**Answer** : system-utm

---

## Persistence Alert

### Alert Scenario

|Champ|Valeur|
|---|---|
|Alert Name|Potential Task Scheduler Persistence Identified|
|Time|30/08/2025 10:06:07 AM|
|Host|WIN-H015|
|User|oliver.thompson|
|Task Name|AssessmentTaskOne|

Index Splunk : `win-alert`

### Investigating the Alert

**Avant d'aller dans le SIEM :**

- Type de host : préfixe `WIN` -> workstation (vs `SRV`, `WEB`, `MSQL` -> serveurs)
- Rôle du user via identity table : oliver.thompson -> System Engineer -> création de tâche planifiée plausible mais à vérifier
- Vérifier localisation et horaires de travail du user

**Recherche principale :**

spl

```spl
index="win-alert" EventCode=4698 AssessmentTaskOne
| table _time EventCode user_name host Task_Name Message
```

Event ID `4698` = scheduled task created. Ne pas filtrer par host -> vérifie si l'activité est isolée ou multi-machines.

Résultat : 1 seul événement.

**Analyse du champ `Message` :**

- **Triggers** : tâche exécutée tous les jours à la même heure -> anormal sur workstation
- **Exec** : `certutil` télécharge `rv.exe` depuis le domaine `tryhotme` -> sauvegardé dans `%Temp%` sous le nom `DataCollector.exe`
- **Lancement** : `Start-Process` (PowerShell) execute `DataCollector.exe`
- **Principals** : exécution sous `oliver.thompson`

-> Persistence confirmée. Vérifier le domaine `tryhotme` sur des plateformes Threat Intelligence (infrastructure attaquant connue ?).

Classification : **True Positive** -> escalade SOC L2.

### Next Investigation Steps

Questions ouvertes pour SOC L2 :

- Comment la tâche planifiée a-t-elle été créée ?
- Comment l'attaquant a accédé à `WIN-H015` ?
- Comment le compte `oliver.thompson` a été compromis ?

### Questions
#### What is the ProcessId of the process that created this malicious task?

```splunk
index="win-alert" AssessmentTaskOne
```

check for the event with the command to see de process id

**Answer** : 5816

#### What is the name of the parent process for the process that created this malicious task?

with the same previous filter we have de parent process id and its name

**Answer** : 4128

#### Which local group did the attacker enumerate during discovery?


```splunk
index="win-alert" CommandLine
| search "net"
```

we can see a command launched by oliver that is : `net localgroup Administrators`

**Answer** : Administrators

#### What is the name of the workstation from which the Threat Actor logged into this host?

```splunk
index="win-alert" CommandLine
| search "Administrator"
```

check the computer name field

![[alert_triage_with_splunk_t3q4.png]]

**Answer** : DEV-QA-SERVER

---

## Web Shell Alert

### Alert Scenario

|Champ|Valeur|
|---|---|
|Alert Name|Potential Web Shell Upload Detected|
|Time|14/09/2025 09:31:51 AM|
|Resource|[http://web.trywinme.thm](http://web.trywinme.thm)|
|Suspicious IP|171.251.232.40|

Index Splunk : `web-alert`

### Investigating the Alert

**Threat Intelligence sur l'IP :** [AbuseIPDB](https://www.abuseipdb.com) -> IP `171.251.232.40` flaggée malveillante +3000 fois, origine vietnamienne.

**Recherche 1 - Activité globale de l'IP :**

spl

```spl
index=web-alert 171.251.232.40
| table _time clientip useragent uri_path method status 
| sort + _time
```

User-Agent `Hydra` détecté -> brute force contre `wp-login.php`.

**Recherche 2 - Exclure Hydra, chercher d'autres activités :**

spl

```spl
index=web-alert 171.251.232.40 useragent!="Mozilla/5.0 (Hydra)" 
| table _time clientip useragent uri_path referer referer_domain method status
```

Résultat clé : POST sur `admin-ajax.php` avec referer `theme-editor.php?file=b374k.php` -> le theme editor ne devrait pas référencer un `.php` arbitraire -> upload/interaction avec un web shell.

**Recherche 3 - Focus sur le web shell :**

spl

```spl
index=web-alert 171.251.232.40 b374k.php
| table _time clientip useragent uri_path referer referer_domain method status
| sort + _time
```

Résultat : accès confirmé à `b374k.php` + 4 POST requests réussies -> exécution de commandes via le web shell. L'upload initial n'est pas visible dans les logs.

**Identification du web shell :** Recherche Google `b374k.php web shell` -> web shell connu et documenté -> confirmation supplémentaire.

**Chronologie de l'attaque :**

1. Brute force Hydra sur `wp-login.php`
2. Accès au theme editor -> upload de `b374k.php`
3. Exécution de commandes via le web shell (4 POST)

Classification : **True Positive** -> escalade SOC L2 + IR.

### Next Investigation Steps

Questions ouvertes pour SOC L2 :

- Le brute force Hydra a-t-il réussi ?
- Comment l'upload du web shell a-t-il eu lieu (non visible dans les logs L1) ?
- Quelles commandes ont été exécutées via `b374k.php` ?

-> Containment & remediation par l'équipe IR.

### Quesitons
#### What time did the brute-force activity using Hydra begin? Answer Format Example: 2025-01-15 12:30:45

```splunk
index="web-alert"
```

check the filed for user agent and add hydra to the filter and sort its by time

```splunk
index="web-alert" useragent="Mozilla/5.0 (Hydra)"
| sort + _time
```

take the first timestamp

**Answer** : 2025-09-14 21:20:27

#### Which user agent did the attacker use when interacting with the web shell?

```splunk
index="web-alert"
```

check the useragent field. there is a second usergent with 25 suspicious requests. we can see that a web shell called "b374k.php" was uploaded. we can use add it to the filter

```splunk
index="web-alert" useragent="Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/138.0.0.0 Safari/537.36" useragent="Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/138.0.0.0 Safari/537.36"
| search "b374k.php"
```

we can see 5 events, one is the upload, so the answer is 4

**Answer** : 4

---

## Résumé - Alert Triage with Splunk

### Méthodologie SOC L1

Avant d'ouvrir le SIEM : analyser les détails de l'alerte (host, user, IP, heure) et contextualiser (type de machine, rôle du user, TI sur l'IP).

Dans le SIEM : affiner les requêtes progressivement, ne pas filtrer trop tôt (ex: ne pas filtrer par host pour détecter une propagation).

Classification finale : True Positive -> escalade SOC L2 + IR / False Positive -> clôture documentée.

### 3 Scénarios couverts

|Scénario|Vecteur|Indicateurs clés|Outil/Log|
|---|---|---|---|
|Initial Access|SSH Brute Force|503 tentatives sur `john.smith`, login `Accepted` confirmé|`linux_secure`, `linux-alert`|
|Persistence|Scheduled Task|`certutil` + domaine suspect + `Start-Process` PowerShell|EventCode `4698`, `win-alert`|
|Web Shell|Upload + exécution|Hydra sur `wp-login.php`, `b374k.php` via theme editor|`web-alert`|

### Points clés à retenir

- IP locale -> attaquant déjà dans le réseau
- EventCode `4698` = création de tâche planifiée Windows
- `certutil` utilisé pour télécharger des binaires malveillants -> red flag
- User-Agent `Hydra` dans les logs web = brute force
- Referer anormal (`theme-editor.php?file=x.php`) -> interaction avec web shell
- Toujours croiser les IPs suspectes sur [AbuseIPDB](https://www.abuseipdb.com) et autres plateformes TI
- Le SOC L1 identifie et escalade -> l'IR contient et remédie

