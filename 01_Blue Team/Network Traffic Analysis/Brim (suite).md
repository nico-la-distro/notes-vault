### Introduction

[BRIM](https://www.brimdata.io/) -> application desktop open-source pour traiter des fichiers pcap et logs. Focus : recherche et analytique.

_Ne pas interagir directement avec les domaines et adresses IP de la room._

---

## What is Brim?

![[brim_what_is_it.png]]

Application desktop open-source pour pcap et logs. Utilise le format Zeek.

Inputs supportés : fichiers pcap (tcpdump, tshark, Wireshark) + logs structurés (Zeek).

Stack technique :

- **Zeek** -> moteur de génération de logs
- **Zed Language** -> langage de requête (keywords, filtres, pipelines)
- **ZNG** -> format de stockage des data streams
- **Electron + React** -> UI cross-platform

### Why Brim?

Wireshark rame sur les gros pcaps (>1 GB). Brim offre une GUI simple et puissante pour réduire le temps d'analyse.

### Brim vs Wireshark vs Zeek

Bonne pratique : pcaps moyens -> Wireshark / création de logs + corrélation -> Zeek / analyse multi-logs -> Brim.

|Fonctionnalité|Brim|Wireshark|Zeek|
|---|---|---|---|
|Sniffing|-|+|+|
|Pcap processing|+|+|+|
|Log processing|+|-|+|
|Packet decoding|-|+|+|
|Scripting|-|-|+|
|Signature support|+|-|+|
|File extraction|-|+|+|
|Pcap > 1 GB|Moyen|Faible|Bon|
|Ease of management|4/5|4/5|3/5|

---

## The Basics

### Landing Page

3 sections principales :

- **Pools** -> fichiers pcap/logs importés
- **Queries** -> requêtes disponibles
- **History** -> requêtes exécutées

### Pools and Log Details

Un pcap importé est automatiquement traité : Brim génère les logs Zeek, les corrèle et affiche tout dans une **timeline** (dates de début/fin de capture).

![[brim_gui.png]]

Hover sur un champ -> détails du champ (utile pour construire des requêtes custom). Résultats exportables via la fonction export près de la timeline.

**Panneau de corrélation** (log details pane) -> IP src/dst, durée, fichiers logs associés. Permet de savoir où chercher ensuite.

![[brim_correlation.png]]

Clic droit sur un champ :

|Action|Description|
|---|---|
|Filter value|Filtrer sur cette valeur|
|Count fields|Compter les occurrences|
|Sort A-Z / Z-A|Trier|
|View details|Voir le détail|
|Whois lookup|Lookup IP|
|Open in Wireshark|Voir les paquets associés|

![[brim_whois_lookup.png]]

### Queries and History

- Double-clic sur une query -> charge la requête dans la search bar + l'ajoute à l'historique
- **12 requêtes premade** dans le dossier "Brim" -> bons templates pour apprendre la syntaxe
- Ajouter une requête custom -> bouton "+" près du menu Queries
- Import d'un seul pcap -> génère automatiquement **9 types de logs Zeek**

![[brim_queries.png]]

### Questions
#### Process the "sample.pcap" file and look at the details of the first DNS log that appear on the dashboard. What is the "qclass_name"?

![[brim_t3q1.png]]

**Answer** : C_INTERNET

#### Look at the details of the first NTP log that appear on the dashboard. What is the "duration" value?

![[brim_t3q2.png]]

**Answer** : 0.005

#### Look at the details of the STATS packet log that is visible on the dashboard. What is the "reassem_tcp_size"?

![[brim_t3q3.png]]

**Answer** : 540

---

## Default Queries

12 requêtes premade dans Brim. Voici ce que chacune apporte :

|Query|Utilité|
|---|---|
|Overall Activity|Vue générale des logs disponibles dans le pcap|
|Windows Specific Networking|Activité réseau Windows : SMB, logins, named pipes, endpoints|
|Unique Network Connections|Liste des connexions uniques -> détection d'anomalies et beaconing|
|Transferred Data|Corrélation connexions/volume de données -> anomalies de transfert|
|DNS Queries|Liste des requêtes DNS -> trafic DNS anormal|
|HTTP Methods|Liste des méthodes HTTP (GET, POST...) -> trafic HTTP anormal|
|File Activity|Fichiers détectés : MIME type, nom, MD5, SHA1 -> data leakage|
|IP Subnet Statistics|Liste des subnets IP -> communications hors scope|
|Suricata Alerts (x3)|Alertes Suricata : par catégorie / src-dst / subnet|

**Suricata** -> moteur de détection open-source (IDS/IPS), développé par l'OISF. Fonctionne comme [Snort](https://tryhackme.com/room/snort), utilise les mêmes signatures.

### Questions
#### Investigate the files. What is the name of the detected GIF file?

![[brim_t4q1.png]]

**Answer** : cat01_with_hidden_text.gif

#### Investigate the conn logfile. What is the number of the identified city names?

```brim
_path=="conn" | cut geo.resp.city | sort | uniq -c
```

![[brim_t4q2.png]]

**Answer** : 2

#### Investigate the Suricata alerts. What is the Signature id of the alert category "Potential Corporate Privacy Violation"?

![[brim_t4q3.png]]

**Answer** : 2,012,887

---

## Use Cases

### Brim Query Reference

|Objectif|Syntaxe|Exemple|
|---|---|---|
|Recherche basique|valeur libre|`10.0.0.1`|
|Opérateurs logiques|`and`, `or`, `not`|`192 and NTP`|
|Filtrer un champ|`field == "value"`|`id.orig_h==192.168.121.40`|
|Lister un log|`_path=="log name"`|`_path=="conn"`|
|Compter par champ|`count() by field`|`count() by _path`|
|Trier|`sort` / `sort -r`|`count() by _path \| sort -r`|
|Extraire des champs|`cut field1, field2`|`_path=="conn" \| cut id.orig_h, id.resp_p`|
|Valeurs uniques|`uniq`|`... \| sort \| uniq`|

> Toujours utiliser les filtres sur les champs. La recherche libre (blind search) est peu performante dans Brim.

### Communicated Hosts

```
_path=="conn" | cut id.orig_h, id.resp_h | sort | uniq
```

### Frequently Communicated Hosts

```
_path=="conn" | cut id.orig_h, id.resp_h | sort | uniq -c | sort -r
```

### Most Active Ports

```
_path=="conn" | cut id.resp_p, service | sort | uniq -c | sort -r count
_path=="conn" | cut id.orig_h, id.resp_h, id.resp_p, service | sort id.resp_p | uniq -c | sort -r
```

### Long Connections

Connexions longues -> indicateur possible de backdoor.

```
_path=="conn" | cut id.orig_h, id.resp_p, id.resp_h, duration | sort -r duration
```

### Transferred Data

Volume de données élevé -> possible exfiltration ou téléchargement de malware.

```
_path=="conn" | put total_bytes := orig_bytes + resp_bytes | sort -r total_bytes | cut uid, id, orig_bytes, resp_bytes, total_bytes
```

### DNS and HTTP Queries

Détection de canaux C2 et hôtes compromis.

```
_path=="dns" | count() by query | sort -r
_path=="http" | count() by uri | sort -r
```

### Suspicious Hostnames

```
_path=="dhcp" | cut host_name, domain
```

### Suspicious IP Addresses

```
_path=="conn" | put classnet := network_of(id.resp_h) | cut classnet | count() by classnet | sort -r
```

### Detect Files

Détection de transferts de malware ou fichiers sensibles via corrélation de hash.

```
filename!=null
```

### SMB Activity

Exploitation, lateral movement, partage de fichiers malveillants.

```
_path=="dce_rpc" OR _path=="smb_mapping" OR _path=="smb_files"
```

### Known Patterns

Alertes générées par Zeek/Suricata contre des patterns d'attaques connus.

```
event_type=="alert" or _path=="notice" or _path=="signatures"
```

---

## Exercise: Threat Hunting with Brim | Malware C2 Detection

Scénario : campagne CobaltStrike via clic sur lien -> téléchargement fichier -> trafic anormal.

### Workflow d'investigation

**1. Vue générale des logs disponibles**

```
count() by _path | sort -r
```

**2. Hôtes fréquemment communiquants**

```
cut id.orig_h, id.resp_p, id.resp_h | sort | uniq -c | sort -r count
```

-> Les IP `10.22.xx` et `104.168.xx` sont suspectes.

**3. Ports et services actifs**

```
_path=="conn" | cut id.resp_p, service | sort | uniq -c | sort -r count
```

-> Volume DNS anormalement élevé.

**4. Analyse DNS**

```
_path=="dns" | count() by query | sort -r
```

-> Requêtes DNS hors normes -> enrichissement avec **VirusTotal** -> 3 IP malveillantes identifiées : `45.147.xx`, `68.138.xx`, `185.70.xx`.

**5. Requêtes HTTP**

```
_path=="http" | cut id.orig_h, id.resp_h, id.resp_p, method, host, uri | uniq -c | sort value.uri
```

-> Requête de téléchargement de fichier depuis l'IP suspecte `104.xx` -> confirmé malveillant via VirusTotal -> associé à **CobaltStrike**.

**6. Alertes Suricata**

```
event_type=="alert" | count() by alert.severity, alert.category | sort count
```

-> Vue d'ensemble des activités malveillantes détectées.

### Points clés

- CobaltStrike utilise rarement un seul canal C2 -> toujours chercher des canaux C2 secondaires.
- Deux approches démontrées : analyse manuelle des logs + Suricata alerts.
- Enrichir les findings avec VirusTotal pour valider les hypothèses.

### Questions
#### What is the name of the file downloaded from the CobaltStrike C2 connection?

```
_path=="http" | cut id.orig_h, id.resp_h, id.resp_p, method, host, uri | uniq -c | sort value.uri
```

![[brim_t6q1.png]]

**Answer** : 4564.exe

#### What is the number of CobaltStrike connections using port 443?

```
_path=="conn" | cut id.resp_h, id.resp_p, service | sort id.resp_h | uniq -c | sort -r count
```

![[brim_t6q2.png]]

**Answer** : 328

#### There is an additional C2 channel in used the given case. What is the name of the secondary C2 channel?

i searched with suricata alerts by caterogy, then i filtered with "A Network Trojan was detected" and i find an other ip where the 10.22.xx.xx communicated with

![[brim_t6q3.png]]

**Answer** : IcedID

---

## Exercise: Threat Hunting with Brim | Crypto Mining

Scénario : détection d'activité de cryptomining (cryptojacking) sur le réseau.

> Le cryptojacking = installation d'un miner sur une machine sans consentement -> exploite CPU, bande passante et électricité de l'entreprise. Peut aussi créer des backdoors via les outils tiers installés.

### Workflow d'investigation

**1. Hôtes fréquemment communiquants**

```
cut id.orig_h, id.resp_p, id.resp_h | sort | uniq -c | sort -r
```

-> IP `192.168.xx` suspecte.

**2. Ports et services actifs**

```
_path=="conn" | cut id.resp_p, service | sort | uniq -c | sort -r count
```

-> Ports inhabituels multiples détectés.

**3. Volume de données transférées**

```
_path=="conn" | put total_bytes := orig_bytes + resp_bytes | sort -r total_bytes | cut uid, id, orig_bytes, resp_bytes, total_bytes
```

-> Trafic massif depuis l'IP suspecte confirmé.

**4. Alertes Suricata**

```
event_type=="alert" | count() by alert.severity, alert.category | sort count
```

-> Catégorie "Crypto Currency Mining" confirmée.

**5. Connexions associées à l'IP suspecte**

```
_path=="conn" | 192.168.1.100
```

-> Identification du serveur de mining.

**6. Mapping MITRE ATT&CK via Suricata**

```
event_type=="alert" | cut alert.category, alert.metadata.mitre_technique_name, alert.metadata.mitre_technique_id, alert.metadata.mitre_tactic_name | sort | uniq -c
```

|Category|Technique|ID|Tactic|
|---|---|---|---|
|Crypto Currency Mining|Resource Hijacking|T1496|Impact|

### Points clés

- Peu de logs disponibles -> s'appuyer rapidement sur Suricata pour confirmer.
- Enrichir avec VirusTotal pour identifier le serveur de mining.
- Les cas de mining interne ne contiennent pas forcément de malware -> ne pas chercher uniquement des IOC classiques.

### Questions
#### How many connections used port 19999?

```
_path=="conn" | cut id.resp_p | sort -r | uniq -c 
```

![[brim_t7q1.png]]

**Answer** : 22

#### What is the name of the service used by port 6666?

```
_path=="conn" | cut id.resp_p, service | sort -r | uniq -c 
```

![[brim_t7q2.png]]

**Answer** : irc

#### What is the amount of transferred total bytes to "101.201.172.235:8888"?

```
_path=="conn" | put total_bytes := orig_bytes + resp_bytes | sort id.resp_h | cut uid, id, orig_bytes, resp_bytes, total_bytes
```

![[brim_t7q3.png]]

**Answer** : 3,729

#### What is the detected MITRE tactic id?

first i checked the suricata alerts, then i filtered with "Crypto Currency Mining Activity Detected" and check for the MITRE tactic id

![[brim_t7q4.png]]

**Answer** : TA0040

---

## Résumé - Room Brim

### L'outil

- Brim -> GUI open-source pour analyser des pcaps et logs Zeek/Suricata
- Idéal pour les gros pcaps et la corrélation multi-logs
- Stack : Zeek (logs) + Zed Language (requêtes) + Suricata (alertes)

### Syntaxe Zed essentielle

```
_path=="conn" | cut id.orig_h, id.resp_h | sort | uniq -c | sort -r
count() by _path | sort -r
event_type=="alert" | count() by alert.severity, alert.category | sort count
filename!=null
```

### Méthodologie d'investigation (applicable à tous les cas)

1. Vue générale des logs disponibles -> `count() by _path`
2. Hôtes fréquemment communiquants -> détecter les IP suspectes
3. Ports/services actifs -> détecter les usages anormaux
4. DNS + HTTP -> détecter les canaux C2 et téléchargements
5. Volume de données -> détecter l'exfiltration ou le mining
6. Suricata alerts -> confirmer et catégoriser
7. Enrichissement externe -> VirusTotal pour valider les hypothèses

### Cas d'usage couverts

|Cas|Indicateurs clés|
|---|---|
|Malware C2 (CobaltStrike)|DNS anormal + téléchargement HTTP + IP malveillantes|
|Crypto Mining|Ports inhabituels + trafic massif + alerte Suricata T1496|

### Réflexes à retenir

- Toujours filtrer par champs -> ne jamais faire de blind search
- CobaltStrike -> chercher plusieurs canaux C2
- Peu de logs disponibles -> aller directement aux alertes Suricata
- Suricata mappe les techniques MITRE ATT&CK -> exploiter cette info
- Enrichir systématiquement avec VirusTotal

---
## SUITE

Now, we invite you to complete the Brim challenge room: [**Masterminds**](https://tryhackme.com/room/mastermindsxlq)

