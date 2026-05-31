
**Contexte :** Triage d'alertes SOC via Kibana (Elastic Stack) sur un serveur IIS/Windows.

**Objectifs :**

- Analyser des logs de sécurité dans Kibana
- Identifier des IoCs
- Corréler des événements multi-sources
- Retracer une compromission à travers une série d'alertes SOC

---

## Scenario Briefing

**Contexte :** SomeCorp -> activité suspecte sur infrastructure -> multiple alertes SOC à trier.

**Setup Kibana :**

1. Démarrer la VM -> attendre ~5 min -> ouvrir `https://LAB_WEB_URL.p.thmlabs.com/`
2. Data view : sélectionner `Alert Triage With Elastic`
3. Time range : `Entire data range`

**Premier filtre à entrer :**

```
_index:weblogs
```

-> filtre uniquement les logs IIS (web server)

> Explorer les champs disponibles dans le panneau gauche du dashboard.

### Questions
#### How many logs are available for analysis within the entire time range?

**Answer** : 1467

#### What is the field value for the `client.ip` in the `weblogs` index?

![[alert_triage_elastic_t2q2.png]]

**Answer** : 203.0.113.55

---

## Investigating Web Attacks

![[alert_triage_elastic_soc_alert.png]]

**Contexte alerte :** IP `203.0.113.55` -> requêtes `POST` vers `proxyLogon.ecp`

### Étape 1 - Visualiser les requêtes POST de l'IP suspecte

Requête :

```
_index:weblogs and client.ip:203.0.113.55 and http.request.method:POST
```

Colonnes à ajouter (bouton `+` dans le panneau gauche) :

|Champ|Utilité|
|---|---|
|`client.ip`|IP source|
|`user.agent`|Détection d'automatisation|
|`http.request.method`|GET/POST|
|`url.path`|Page ciblée|
|`http.response.status_code`|Résultat|

![[alert_triage_elastic_visualiser.png]]

**Conclusion :** Requêtes automatisées liées à la vulnérabilité [ProxyLogon](https://proxylogon.com/) -> True Positive.

### Étape 2 - Confirmer un web shell (alerte suivante, +7 min)

![[alert_triage_elastic_soc_alert2.png]]

Indicateur clé : paramètre `cmd=` dans l'URL -> exécution de commandes via web shell.

Requête :

```
_index:weblogs and client.ip:203.0.113.55 and http.request.method:GET and errorEE.aspx
```

-> Trier `Old-New` pour voir les premières commandes exécutées.

-> Les commandes apparaissent dans le champ `url.path`.

![[alert_triage_elastic_visualiser2.png]]

**Conclusion :** Web shell confirmé -> escalader les deux alertes en **True Positive**.

### Questions
#### How many `POST` requests did the IP address `203.0.113.55` make to `proxyLogon.ecp`?

```elastic
_index:weblogs and client.ip:203.0.113.55 and http.request.method : "POST" 
```

**Answer** : 3

#### Which `user.agent` paired with the IP address `203.0.113.55` made the `POST` requests?

add user.agent as a column

**Answer** : python-requests/2.25.1

**Conclusion** -> True Positive
#### How many logs contain the `cmd=` query parameter in the `url.path` field?

```elastic
_index:weblogs and client.ip:203.0.113.55 and http.request.method : "GET"  and errorEE.aspx
```

Sort fields : Old-new

add url.pth as a column to see the `cmd=` query parameter

**Answer** : 20

#### Which command was run utilizing `errorEE.aspx` on `Jul 20, 2025 @ 04:45:50.000`?

set the time range to Jul 20, 2025 @ 04:30 -> Jul 20, 2025 @ 05;00

**Answer** : hostname

**Conclusion** -> True Positive

---

## Uncovering Account Activity

![[alert_triage_elastic_soc_alert3.png]]

**Alerte SOC-20250720-0014 :** Connexion `Administrator` hors heures ouvrées -> `winserv2019.some.corp` -> 05:11:22

### Étape 1 - Confirmer le logon (Event ID 4624)

```
@timestamp >= "2025-07-20T05:11:22" and winlog.event_id:4624 and host.name:winserv2019.some.corp and winlog.event_data.TargetUserName:Administrator
```

Colonnes utiles :

|Champ|Utilité|
|---|---|
|`winlog.event_id`|ID de l'événement Windows|
|`host.name`|Hostname cible|
|`winlog.event_data.TargetUserName`|Compte connecté|
|`winlog.logon.type`|Mode de connexion (ex: RDP)|
|`winlog.event_data.IpAddress`|IP source|

![[alert_triage_elastic_visualiser3.png]]

**Résultat :** Logon confirmé depuis `203.0.113.55` -> même IP que les alertes web.

### Étape 2 - Corréler avec Sysmon (Event ID 1 - Process Creation)

```
@timestamp >= "2025-07-20T05:11:22" and winlog.event_id:1 and user.name:Administrator
```

Colonnes utiles :

|Champ|Utilité|
|---|---|
|`user.name`|Compte ayant lancé le processus|
|`process.parent.name`|Processus parent|
|`process.command_line`|Commande complète|

![[alert_triage_elastic_visualiser4.png]]

**Résultat :** Chaîne de processus alignée avec l'initialisation de session Windows -> logon confirmé mais insuffisant pour conclure à une compromission.

### Étape 3 - Investiguer la création de compte (Alerte SOC-20250720-0015)

![[alert_triage_elastic_soc_alert4.png]]

**Alerte :** Nouveau compte créé par `Administrator` -> 05:13:09 -> criticité maximale.

```
@timestamp >= "2025-07-20T05:13:10.000" and winlog.channel:Security and winlog.task:"User Account Management"
```

Colonnes utiles : `winlog.event_id`, `winlog.task`, `message`

-> Trier `Old-New` pour voir les événements les plus anciens en premier.

> `winlog.task` -> contexte lisible du type d'activité représenté par l'événement.

### Questions
#### What is the `winlog.record_id` of the Administrator `4624` logon event?

```elastic
@timestamp >= "2025-07-20T05:11:22" and winlog.event_id:4624 and host.name:winserv2019.some.corp and winlog.event_data.TargetUserName:Administrator
```

add winlog.record_id as a column

**Answer** : 17166

#### What is the `process.pid` of the Sysmon `1` event that occurred on `Jul 20, 2025 @ 05:11:27.996`?

```elastic
@timestamp >= "2025-07-20T05:11:27" and winlog.event_id:1 and host.name:winserv2019.some.corp
```

add process.pid as a column

**Answer** : 964

#### What is the `winlog.event_id` for the new user account being created?

i know this id

**Answer** : 4720

#### What is the name of the new user account?

```elastic
@timestamp >= "2025-07-20T05:13:10.000" and winlog.channel:Security and winlog.task:"User Account Management"
```

add message as a column and check inside it :

[...]Domain: SOME Logon ID: 0x4ff5f New Account: Security ID: S-1-5-21-363149898-3377733843-3686914969-1114 Account Name: svc_backup Account Domain: [...]

**Answer** : svc_backup

---

## Exposing Command Execution

![[alert_triage_elastic_soc_alert5.png]]

**Alerte SOC-20250720-0016 :** `cmd.exe` suspect -> `Administrator` -> 05:13:15

### Scope des processus enfants de cmd.exe

```
@timestamp >= "2025-07-20T05:13:15" and process.parent.name:cmd.exe and user.name:Administrator
```

Colonnes : `process.command_line`, `process.name`, `process.parent.name`

![[alert_triage_elastic_visualiser5.png]]

### Corrélation avec Event ID 4732 (Security Group Management)

```
@timestamp >= "2025-07-20T05:13:15" and (winlog.event_id:4732 or process.parent.name:cmd.exe)
```

-> Corrèle Sysmon (Event 1) + Security (Event 4732) -> confirme ajout du nouveau compte dans des groupes via `net`.

![[alert_triage_elastic_visualiser6.png]]

### PowerShell Usage

```
@timestamp >= "2025-07-20T05:13:15" and event.module:powershell and event.code:4104
```

Colonne à ajouter : `powershell.file.script_block_text` -> trier `Old-New`

**Premières commandes visibles :** `whoami` et `whoami /priv` -> discovery classique post-compromission.

![[alert_triage_elastic_visualiser7.png]]

### No Alert Created - Rar.exe

```
process.name: "Rar.exe"
```

-> Aucune alerte générée (logiciel légitime chez le client) mais usage suspect à investiguer.

Questions à se poser :

- Quel compte a exécuté `Rar.exe` ? Quand ? Via quel processus parent ?
- Le timing corrèle-t-il avec le compte backdoor créé via CMD ?

**Timeline reconstruite :**

|Étape|Action|
|---|---|
|1|Exploitation ProxyLogon sur IIS|
|2|Connexion RDP en `Administrator` depuis `203.0.113.55`|
|3|Création backdoor + ajout groupes via `cmd.exe`|
|4|Discovery PowerShell (`whoami`, `whoami /priv`)|
|5|Usage de `Rar.exe` par le nouveau compte|

### Questions
#### What command does the attacker use to add the new account to the "Remote Desktop Users" group?

```elastic
@timestamp >= "2025-07-20T05:13:15" and process.parent.name:cmd.exe and user.name:Administrator
```

**Answer** : net localgroup "Remote Desktop Users" svc_backup /add

#### What is the `winlog.record_id` of the `4732` Security

```elastic
@timestamp >= "2025-07-20T05:13:15" and (winlog.event_id:4732 or process.parent.name:cmd.exe)
```

add winlog.record_id as a column

![[alert_triage_elastic_t5q2.png]]

**Answer** : 17254

#### What PowerShell command did the attacker run on `Jul 20, 2025 @ 05:16:14.628`?

```elastic
@timestamp >= "2025-07-20T05:16:14" and event.module:powershell
```

add powershell.file.script_block_text as a column

**Answer** : net group "Domain Admins" /domain

#### What is the name of the archive that the attacker creates using the `Rar.exe` executable?

```elastic
process.name : "Rar.exe" 
```

add process.command_line as a column :

"C:\Program Files\WinRAR\rar.exe" a -hpSpring2025! -m5 C:\Temp\finance_it_archive.rar C:\Users\asmith\Documents\* C:\IT\Admin\Scripts\*

**Answer** : finance_it_archive.rar

---

## Résumé - Alert Triage with Elastic

**Room :** [https://tryhackme.com/room/alerttriagewithelastic](https://tryhackme.com/room/alerttriagewithelastic)
### Contexte

SomeCorp -> serveur IIS/Windows compromis -> investigation via Kibana (Elastic Stack) en corrélant weblogs, Windows Security Events et Sysmon.
#### Timeline de l'attaque

|Heure|Action|Source|
|---|---|---|
|~04:45|Exploitation ProxyLogon (`POST` vers `proxyLogon.ecp`) via script Python|weblogs|
|~04:45|Déploiement web shell `errorEE.aspx` + exécution de commandes (`cmd=`)|weblogs|
|05:11:22|Connexion RDP `Administrator` depuis `203.0.113.55` (Event 4624)|Security|
|05:13:09|Création du compte backdoor `svc_backup` (Event 4720)|Security|
|05:13:15|Ajout de `svc_backup` aux groupes via `net localgroup` (Event 4732)|Security + Sysmon|
|05:16:14|Discovery PowerShell : `whoami`, `whoami /priv`, `net group "Domain Admins" /domain`|PowerShell 4104|
|Post-compromise|Archivage de données avec `Rar.exe` -> `finance_it_archive.rar`|Sysmon|

### IoCs clés

|Indicateur|Valeur|
|---|---|
|IP attaquant|`203.0.113.55`|
|User-agent|`python-requests/2.25.1`|
|Web shell|`errorEE.aspx`|
|Vuln exploitée|ProxyLogon|
|Compte backdoor|`svc_backup`|
|Archive exfiltrée|`finance_it_archive.rar`|
|Commande RAR|`rar.exe a -hpSpring2025! -m5 C:\Temp\finance_it_archive.rar C:\Users\asmith\Documents\* C:\IT\Admin\Scripts\*`|

### Points clés méthodologiques

**Pivots d'investigation :**

- weblogs -> Security Events -> Sysmon -> PowerShell logs
- Toujours corréler l'IP source entre les sources de logs
- `winlog.task` -> contexte lisible sur le type d'activité
- Event ID 4104 (PowerShell ScriptBlock) -> voir les commandes en clair

**Rappel Event IDs importants :**

|Event ID|Signification|
|---|---|
|4624|Logon réussi|
|4720|Création de compte|
|4732|Ajout à un groupe de sécurité|
|1 (Sysmon)|Process Creation|
|4104 (PS)|ScriptBlock logging|

**Leçon clé :** Toutes les actions malveillantes ne génèrent pas d'alerte (`Rar.exe` -> logiciel légitime). Le rôle du L1 SOC est de corréler le contexte pour conclure malgré l'absence d'alerte.

