## Incident Handling - Life Cycle

4 phases :

**1. Preparation** -> Politiques, contrôles de sécurité (EDR/SIEM/IDS/IPS), formation du staff.

**2. Detection and Analysis** -> Réception des alertes, investigation, root cause analysis, threat hunting.

**3. Containment, Eradication, and Recovery** -> Isolation de l'hôte infecté, nettoyage du réseau, reprise de contrôle.

**4. Post-Incident Activity / Lessons Learnt** -> Identification des failles, ajout de règles de détection, formation si nécessaire.

---

## Incident Handling: Scenario

**Contexte** : Défacement du site `http://www.imreallynotbatman.com` (Wayne Enterprises). Investigation -> phase Detection and Analysis.

**Objectif** : Mapper les activités de l'attaquant sur les 7 phases du Cyber Kill Chain via Splunk.

### Cyber Kill Chain

Reconnaissance -> Weaponization -> Delivery -> Exploitation -> Installation -> Command & Control -> Actions on Objectives

> L'ordre n'est pas forcément suivi pendant l'investigation. OSINT utilisable pour combler les lacunes.

### Interesting log Sources

|Source|Contenu|
|---|---|
|`wineventlog`|Windows Event logs|
|`winRegistry`|Création/modification/suppression de clés de registre|
|`XmlWinEventLog`|Sysmon (important pour investigation)|
|`fortigate_utm`|Logs firewall Fortinet|
|`iis`|Logs serveur web IIS|
|`Nessus:scan`|Résultats scanner de vulnérabilités|
|`Suricata`|Alertes IDS (déclencheur + cause)|
|`stream:http`|Flux réseau HTTP|
|`stream:DNS`|Flux réseau DNS|
|`stream:icmp`|Flux réseau ICMP|

**Index à utiliser** : `index=botsv1`

---

## Reconnaissance Phase

**Objectif** : Identifier l'IP effectuant de la reconnaissance sur `imreallynotbatman.com`.

### Étape 1 - Identifier les sources de logs pertinentes

splunk

```splunk
index=botsv1 imreallynotbatman.com
```

Sources retournées : `suricata`, `stream:http`, `fortigate_utm`, `iis`

### Étape 2 - Analyser le trafic HTTP entrant

splunk

```splunk
index=botsv1 imreallynotbatman.com sourcetype=stream:http
```

-> Champ `src_ip` (panneau gauche) révèle deux IPs :

- `40.80.148.42` -> pourcentage élevé des logs = suspect principal
- `23.22.63.114`

Cliquer sur une IP l'ajoute directement à la requête. Examiner : User-Agent, POST requests, URIs.

### Étape 3 - Valider via Suricata

splunk

```splunk
index=botsv1 imreallynotbatman.com src=40.80.148.42 sourcetype=suricata
```

-> Vérifier les alertes déclenchées sur cette IP. Le champ des signatures n'est pas visible par défaut -> cliquer sur **More fields** et chercher manuellement.

**IP de reconnaissance confirmée** : `40.80.148.42`

### Questions
#### One suricata alert highlighted the CVE value associated with the attack attempt. What is the CVE value?

With this filter :

```splunk
index="botsv1" imreallynotbatman.com sourcetype="suricata" src_ip="40.80.148.42" event_type=alert
```

in this filed :

![[splunk201_t4q1.png]]

**Answer** : CVE-2014-6271

#### What is the CMS our web server is using?

Filtre : 

```splunk
index="botsv1" imreallynotbatman.com sourcetype="suricata" src_ip="40.80.148.42"
```

![[splunk201_t4q2.png]]

**Answer** : joomla

#### What is the web scanner, the attacker used to perform the scanning attempts?

filter :

```splunk
index="botsv1" imreallynotbatman.com sourcetype="suricata" src_ip="40.80.148.42"
```


![[splunk201_t4q3.png]]

**Answer** : Acunetix

#### What is the IP address of the server imreallynotbatman.com?

![[splunk201_t4q4.png]]

**Answer** : 192.168.250.70

----

## Exploitation Phase

**Contexte** : Webserver cible = `192.168.250.70`, CMS = Joomla, page admin = `/joomla/administrator/index.php`

### Compter les requêtes par IP source

splunk

```splunk
index=botsv1 imreallynotbatman.com sourcetype=stream*
| stats count(src_ip) as Requests by src_ip
| sort - Requests
```

### Trafic entrant vers le webserver

splunk

```splunk
index=botsv1 sourcetype=stream:http dest_ip="192.168.250.70"
```

-> Champs utiles : `src_ip`, `http_method`, `form_data`, `http_user_agent`, `uri`

### Isoler les POST sur le panel admin Joomla

splunk

```splunk
index=botsv1 sourcetype=stream:http dest_ip="192.168.250.70" http_method=POST uri="/joomla/administrator/index.php" form_data=*username*passwd*
| table _time uri src_ip dest_ip form_data
```

-> `username` = toujours `admin`, `passwd` = multiple valeurs -> brute-force

### Extraire les credentials avec rex

splunk

```splunk
index=botsv1 sourcetype=stream:http dest_ip="192.168.250.70" http_method=POST form_data=*username*passwd*
| rex field=form_data "passwd=(?<creds>\w+)"
| table _time src_ip uri http_user_agent creds
```

**Résultats** :

- `23.22.63.114` -> brute-force automatisé via script Python
- `40.80.148.42` -> 1 seule tentative, mot de passe `batman`, via Mozilla

### Questions
#### What was the URI which got multiple brute force attempts?

**Answer** : /joomla/administrator/index.php

#### Against which username was the brute force attempt made?

**Answer** : admin

#### What was the correct password for admin access to the content management system running **imreallynotbatman.com**?

**Answer** : batman

#### How many unique passwords were attempted in the brute force attempt?

filtre :

```splunk
index=botsv1 sourcetype=stream:http dest_ip="192.168.250.70" http_method=POST uri="/joomla/administrator/index.php"
```


![[splunk201_t5q4.png]]

**Asnwer** : 412

#### What IP address is likely attempting a brute force password attack against **imreallynotbatman.com**?

cf previous screen

**Answer** : 23.22.63.114

#### After finding the correct password, which IP did the attacker use to log in to the admin panel?

filtre :

```splunk
index=botsv1 sourcetype=stream:http dest_ip="192.168.250.70" http_method=POST form_data=*username*passwd*
| rex field=form_data "passwd=(?<creds>\w+)"
| table _time src_ip uri http_user_agent creds
```

![[splunk201_t5q6.png]]


---

## Installation Phase

**Objectif** : Identifier les fichiers malveillants uploadés et exécutés sur `192.168.250.70`.

### Rechercher des exécutables uploadés

splunk

```splunk
index=botsv1 sourcetype=stream:http dest_ip="192.168.250.70" *.exe
```

-> Champ `part_filename{}` révèle deux fichiers :

- `3791.exe` -> exécutable
- `agent.php` -> backdoor PHP

### Confirmer l'IP source de l'upload

splunk

```splunk
index=botsv1 sourcetype=stream:http dest_ip="192.168.250.70" "part_filename{}"="3791.exe"
```

-> Champ `c_ip` = IP cliente ayant uploadé le fichier.

### Vérifier si le fichier a été exécuté

splunk

```splunk
index=botsv1 "3791.exe"
```

-> Sources contenant des traces : `XmlWinEventLog` (Sysmon), `wineventlog`, `fortigate_utm`

splunk

```splunk
index=botsv1 "3791.exe" sourcetype="XmlWinEventLog" EventCode=1
```

-> `EventCode=1` = Process Creation (Sysmon) -> confirme l'exécution du fichier sur le serveur compromis.

### Questions
#### Sysmon also collects the Hash value of the processes being created. What is the MD5 HASH of the program 3791.exe?

```splunk
index=botsv1 "3791.exe" sourcetype="XmlWinEventLog" EventCode=1
```

then clic hashes filtre, so the filter is : 

```splunk
index=botsv1 "3791.exe" sourcetype="XmlWinEventLog" EventCode=1 Hashes="SHA1=65DF73D77324D008C83C3E57B445DF0FD43A3A51,MD5=AAE3F5A29935E6ABCC2C2754D12A9AF0,SHA256=EC78C938D8453739CA2A370B9C275971EC46CAF6E479DE2B2D04E97CC47FA45D,IMPHASH=481F47BBB2C9C21E108D65F52B04C448"
```

**Answer** : AAE3F5A29935E6ABCC2C2754D12A9AF0

#### Looking at the logs, which user executed the program 3791.exe on the server?

with same filter above, go to filed User

![[splunk201_t6q2.png]]

**Answer** : NT AUTHORITY\IUSR

#### Search hash on the virustotal. What other name is associated with this file 3791.exe?

put the previous hash in search bar in VT, then see the other name of the .exe

**Answer** : ab.exe

---

## Action on Objectives

**Objectif** : Identifier ce qui a causé le défacement du site.

### Vérifier le trafic entrant vers le webserver

splunk

```splunk
index=botsv1 dest=192.168.250.70 sourcetype=suricata
```

-> Aucune IP externe trouvée. Inverser la direction.

### Vérifier le trafic sortant depuis le webserver

splunk

```splunk
index=botsv1 src=192.168.250.70 sourcetype=suricata
```

-> Anormal : le serveur initie du trafic vers 3 IPs externes (normalement c'est le client qui initie).

### Investiguer une IP destination suspecte

splunk

```splunk
index=botsv1 src=192.168.250.70 sourcetype=suricata dest_ip=23.22.63.114
```

-> Champ `url` révèle 2 fichiers PHP + 1 fichier JPEG suspect.

### Tracer l'origine du fichier JPEG

splunk

```splunk
index=botsv1 url="/poisonivy-is-coming-for-you-batman.jpeg" dest_ip="192.168.250.70"
| table _time src dest_ip http.hostname url
```

**Résultat** : Le JPEG `poisonivy-is-coming-for-you-batman.jpeg` a été téléchargé depuis `prankglassinebracket.jumpingcrab.com` -> cause du défacement.

### Questions
#### What is the name of the file that defaced the imreallynotbatman.com website ?

![[splunk201_t7q1.png]]

**Answer** : poisonivy-is-coming-for-you-batman.jpeg

#### Fortigate Firewall 'fortigate_utm' detected SQL attempt from the attacker's IP 40.80.148.42. What is the name of the rule that was triggered during the SQL Injection attempt?

```splunk
index=botsv1 src=40.80.148.42
```

![[splunk201_t7q2.png]]

**Answer** : HTTP.URI.SQL.Injection

---

## Command and Control Phase

**Objectif** : Identifier l'IP/domaine C2 résolu via Dynamic DNS.

### Rechercher dans les logs firewall Fortinet

splunk

```splunk
index=botsv1 sourcetype=fortigate_utm "poisonivy-is-coming-for-you-batman.jpeg"
```

-> Champ `url` contient le FQDN du serveur C2.

### Confirmer via stream:http

splunk

```splunk
index=botsv1 sourcetype=stream:http dest_ip=23.22.63.114 "poisonivy-is-coming-for-you-batman.jpeg" src_ip=192.168.250.70
```

### Confirmer via DNS

splunk

```splunk
index=botsv1 sourcetype=stream:dns
```

-> Vérifier les requêtes DNS émises par `192.168.250.70` pendant la période d'infection.

**Résultat** : `prankglassinebracket.jumpingcrab.com` -> serveur C2 contacté après compromission du serveur.

---

## Weaponization Phase

**Objectif** : OSINT sur les domaines/IPs de l'attaquant pour reconstituer l'infrastructure.

**Domaine C2 identifié** : `prankglassinebracket.jumpingcrab.com` **IP associée** : `23.22.63.114`

### Robtex

Site : [https://www.robtex.com](https://www.robtex.com)

```
# Recherche domaine
https://www.robtex.com/dns-lookup/prankglassinebracket.jumpingcrab.com

# Recherche IP
https://www.robtex.com/ip-lookup/23.22.63.114
```

-> IP `23.22.63.114` associée à des domaines imitant Wayne Enterprise.

### VirusTotal

Site : [https://www.virustotal.com](https://www.virustotal.com)

-> Rechercher `23.22.63.114` -> onglet **Relations** -> liste des domaines associés. -> Domaine attaquant identifié : `www.po1s0n1vy.com` -> Rechercher ensuite `www.po1s0n1vy.com` sur VirusTotal pour plus de détails.

### Whois

```
https://whois.domaintools.com
```

-> Informations d'enregistrement du domaine (registrant, dates, etc.).

**Résumé infrastructure attaquant** :

- C2 : `prankglassinebracket.jumpingcrab.com`
- IP : `23.22.63.114`
- Domaine attribuable : `www.po1s0n1vy.com`

---

## Delivery Phase

**Objectif** : Retrouver les malwares liés à l'adversaire via OSINT.

**Contexte** : Groupe Poison Ivy -> vecteur d'attaque secondaire si compromission initiale échoue.

### ThreatMiner

```
https://www.threatminer.org/host.php?q=23.22.63.114
```

-> 3 fichiers associés à l'IP `23.22.63.114`. -> Hash malveillant identifié : `c99131e0169171935c5ac32615ed6261` -> Cliquer sur le hash pour accéder aux métadonnées du fichier.

### VirusTotal

```
https://www.virustotal.com
```

-> Rechercher le hash MD5 -> onglet **Details** pour les métadonnées du malware.

### Hybrid-Analysis

```
https://www.hybrid-analysis.com/sample/9709473ab351387aab9e816eff3910b9f28a7a70202e250ed46dba8f820f34a8?environmentId=100
```

-> Analyse comportementale complète du malware :

|Info disponible|
|---|---|
|Network Communication|DNS Requests|
|Contacted Hosts + géolocalisation|Strings|
|MITRE ATT&CK Mapping|Indicateurs malveillants|
|DLLs Imports/Exports|Mutex|
|File Metadata|Screenshots|

----

## Résumé - Splunk 201 : Cyber Kill Chain Investigation

**Contexte** : Défacement de `imreallynotbatman.com` (Wayne Enterprises). Index : `index=botsv1`.

### Cyber Kill Chain - Findings

|Phase|Findings|
|---|---|
|**Reconnaissance**|IP `40.80.148.42` scanne le serveur via Acunetix|
|**Weaponization**|Infrastructure attaquant : `prankglassinebracket.jumpingcrab.com`, IP `23.22.63.114`, domaine `po1s0n1vy.com`, email `Lillian.rose@po1s0n1vy.com`|
|**Delivery**|Malware secondaire : `MirandaTateScreensaver.scr.exe` (MD5 : `c99131e0169171935c5ac32615ed6261`)|
|**Exploitation**|Brute-force depuis `23.22.63.114` sur `/joomla/administrator/index.php` -> 142 tentatives, 1 succès. Accès final depuis `40.80.148.42`|
|**Installation**|Upload de `3791.exe` + `agent.php` sur `192.168.250.70`, exécution confirmée via Sysmon `EventCode=1`|
|**C2**|Serveur contacté : `prankglassinebracket.jumpingcrab.com` -> `23.22.63.114`|
|**Action on Objectives**|JPEG `poisonivy-is-coming-for-you-batman.jpeg` téléchargé depuis le C2 -> défacement|

### Réflexes Splunk retenus

- `sourcetype=stream:http` -> trafic HTTP, champs : `src_ip`, `form_data`, `http_user_agent`, `uri`
- `sourcetype=suricata` -> alertes IDS, champ : `alert.signature`
- `sourcetype=XmlWinEventLog` + `EventCode=1` -> exécution de processus (Sysmon)
- `| rex field=X "pattern=(?<new_field>\w+)"` -> extraction regex
- `| stats count by field` / `| table field1 field2` -> pivot rapide

### OSINT utilisés

- **Robtex** -> domaines/IPs liés
- **VirusTotal** -> hash, relations, métadonnées
- **ThreatMiner** -> fichiers associés à une IP
- **Hybrid-Analysis** -> comportement malware, MITRE ATT&CK

