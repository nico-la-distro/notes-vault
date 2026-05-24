## What Is Logged

### Logging Overview

Les logs servent à : Incident Response, Threat Hunting, Alerting & Triage.

### Anatomy of a Log Entry

Logs stockés en binaire dans `C:\Windows\System32\winevt\Logs` (format `.evtx`). Chaque fichier EVTX = une catégorie (ex: Application Logs, Security Logs).

### Reading Event Logs

Outil : **Event Viewer** → `Win + R` → `eventvwr`

|Élément|Description|
|---|---|
|Log Sources|Panel gauche — un item par fichier EVTX|
|Keywords|Succès ou échec de l'action|
|Date and Time|Heure système (pas UTC)|
|Event ID|Numéro unique par type d'event (ex: `4625` = failed login)|
|Log Details|Contenu brut, onglet "Details" pour le XML|
|Filters Menu|"Filter Current Log" / "Find" pour filtrer|

![[event_viewer.png]]

### What Is Logged

+500 Event IDs rien que pour les Security logs. Tous les events ne sont pas loggés par défaut ni documentés. Référence : [ultimatewindowssecurity.com](https://www.ultimatewindowssecurity.com/securitylog/encyclopedia/)

---

## Security Log: Authentication

### Overview

Security log = log le plus utile par défaut.

|Event ID|Objectif|Loggé sur|Limites|
|---|---|---|---|
|4624 (Successful Logon)|Détecter RDP/network logins suspects|Machine cible|Très bruyant sur serveurs chargés|
|4625 (Failed Logon)|Détecter brute force, password spraying, scanning|Machine cible|Inconsistant, nombreux edge cases|

### Structure of 4624

Champs clés : **Logon ID**, **Logon Type**, **Username**, **Source IP**, **Hostname**

![[structure_enventid_4624.png]]

### Usage of 4624/4625

**Detect RDP Brute Force → Event 4625**

- Filtrer : Logon Type `3` (NLA activé, moderne) ou `10` (NLA désactivé, ancien/mal configuré)
- Red flags :
    - Multiples usernames tentés (`admin`, `helpdesk`, `cctv`) → password spraying
    - Nombreux échecs sur un seul compte (`Administrator`) → brute force
    - `Workstation Name` hors pattern corporate (ex: `kali` au lieu de `THM-PC-06`)
    - Source IP inattendue (ex: imprimante → serveur Windows)

**Analyse RDP Logons → Event 4624**

- Filtrer : Logon Type `10` (RDP)
- Si NLA activé → chaque logon type `10` est précédé d'un `4624` type `3`
    - Le vrai `Workstation Name` est dans cet event type `3` précédent
- Red flags : brute force précédente ou IP/hostname suspect
- Si login jugé malveillant → noter le **Logon ID** (ex: `0x5D6AC`) = identifiant de session unique pour la suite de l'investigation

_**NLA (Network Level Authentication)** : mécanisme d'authentification RDP qui exige que l'utilisateur s'authentifie avant l'établissement de la session RDP. Activé par défaut sur les systèmes modernes. Sans NLA, la session s'ouvre avant l'auth → logon type `10` directement visible._

---

## Security Log: User Management

### Overview

|Event ID|Description|Usage malveillant|
|---|---|---|
|4720 / 4722 / 4738|Compte créé / activé / modifié|Backdoor account ou réactivation d'ancien compte|
|4725 / 4726|Compte désactivé / supprimé|Désactiver des comptes SOC pour ralentir la réponse|
|4723 / 4724|Password changé / réinitialisé|Reset du password pour prendre le contrôle du compte|
|4732 / 4733|Ajout / retrait d'un groupe de sécurité|Ajouter un compte backdoor au groupe `Administrators`|

### Structure of User Management Events

- **Subject** : qui fait l'action — contient le **Logon ID** (corrélable avec `4624`)
- **Object** : cible de l'action (nommé `New Account`, `Member`, etc. selon l'event)
- **Details** : groupe cible (4732/4733) ou attributs du nouveau compte (4720)

![[structure_user_managment_events.png]]

### Usage of User Management Events

**Hunt for Backdoored Users → Events 4720 / 4732**

Red flags :

- Aucun membre IT ne confirme l'action
- Action en dehors des heures de travail / week-end
- Nom du Subject inconnu ou suspect (ex: `adm.old.2008`)
- Nom du compte cible hors naming pattern (ex: `backup` au lieu de `thm_svc_backup`)

Si action confirmée malveillante :

1. Copier le **Logon ID** du `4720` / `4732`
2. Retrouver le `4624` correspondant avec ce même Logon ID
3. Appliquer l'analyse RDP/auth de la section précédente

---

## Sysmon: Process Monitoring

### Overview

|Event Code|Objectif|Limites|
|---|---|---|
|4688 (Security Log)|Log chaque nouveau process + command line + parent|Désactivé par défaut|
|1 (Sysmon)|Remplace 4688, ajoute hash, signature, plus de champs|Outil externe, non installé par défaut|

### Sysmon vs Security Log

Sysmon = outil gratuit Microsoft Sysinternals, standard de facto pour le monitoring avancé. Préférer Sysmon à 4688 : moins de bruit, plus de champs utiles.

Logs dans Event Viewer : `Applications & Services → Microsoft → Windows → Sysmon → Operational`

### Sysmon Event ID 1 in Action

|Groupe|Champs clés|
|---|---|
|Process Info|PID, path (Image), command line|
|Parent Info|Contexte du parent → construction du process tree|
|Binary Info|Hash, signature, métadonnées PE|
|User Context|User + **Logon ID** (corrélable avec Security logs)|

**Analyse Process Launch → Sysmon Event ID 1**

Vérifier le process :

- `Image` dans un répertoire suspect (`C:\Temp`, `C:\Users\Public`)
- Nom suspect (`aa.exe`, `jqyvpqldou.exe`)
- Hash MD5/SHA256 positif sur [VirusTotal](https://www.virustotal.com)

Vérifier le parent :

- Parent présente les mêmes red flags (nom, path, hash)
- Parent incohérent (ex: Notepad qui lance des commandes CMD)

Remonter le process tree si nécessaire :

1. Trouver l'event précédent où `ProcessId` = `ParentProcessId` de l'event courant
2. Répéter les vérifications ci-dessus

Tracer l'attack chain : filtrer tous les events Security + Sysmon avec le même **Logon ID**

---

## Sysmon: Files and Network

### Overview

|Event ID|Alternative Security Log|Objectif|
|---|---|---|
|11 / 13 (File Create / Registry Value Set)|4656 / 4657 (désactivés par défaut)|Détecter fichiers droppés ou modifications registre (persistence)|
|3 / 22 (Network Connection / DNS Query)|Aucune alternative directe|Détecter trafic suspect ou vers destinations malveillantes|

### Structure of Sysmon Events

Tous les events Sysmon partagent les mêmes champs process (highlighted en orange). Les champs Logon ID et parent process sont absents → utiliser le **ProcessId** pour retrouver l'event ID 1 correspondant et obtenir le contexte complet.

![[structure_sysmon_event.png]]

### Usage of Sysmon Events

**Analyse Process Activities → corréler via ProcessId**

1. Copier le `ProcessId` depuis l'event ID 1
2. Filtrer tous les events Sysmon avec ce même `ProcessId`

Red flags — connexions réseau (Event 3 / 22) :

- Connexion vers IP externe sur port `80` ou port non-standard (ex: `4444`)
- IP de destination connue malveillante (vérifier VirusTotal)
- DNS query vers domaine suspect (`*.top`, `*.click`, `hpdaykfpadvsl.com`)

Red flags — fichiers & registre (Event 11 / 13) :

- Fichier droppé dans `C:\Temp` ou `C:\Users\Public`
- Fichier droppé = script (`.bat`, `.ps1`) ou exécutable (`.exe`, `.com`)
- Fichier ou clé registre créé à des fins de persistence

---

## PowerShell: Logging Commands

### Overview

Sysmon Event ID 1 ne capture que le lancement de `powershell.exe` — pas les commandes exécutées dans la session. Tout ce qui se passe à l'intérieur (lecture de fichiers, download, discovery) est invisible pour Sysmon.

### How It Works

PowerShell est un outil tout-en-un : des centaines de commandes peuvent s'exécuter dans une seule session sans créer de nouveaux processus → Sysmon inefficace ici.

### PowerShell History File

plaintext

```plaintext
C:\Users\<USER>\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadline\ConsoleHost_history.txt
```

Fichier texte plain, créé automatiquement, mis à jour à chaque commande entrée.

Points clés :

- Un fichier par utilisateur actif sur le système
- Persiste après reboot, sauf suppression manuelle
- Ne log **pas** les outputs ni le contenu des scripts (ex: `powershell .\script.ps1`)
- Utile pour tracer : system discovery, téléchargements malveillants, commandes manuelles

---

## Résumé — Windows Logging for SOC

### Les sources de logs à connaître

|Source|Outil|Emplacement|
|---|---|---|
|Security Log|Event Viewer|`C:\Windows\System32\winevt\Logs`|
|Sysmon|Event Viewer|`Applications & Services → Microsoft → Windows → Sysmon → Operational`|
|PowerShell History|Fichier texte|`C:\Users\<USER>\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadline\ConsoleHost_history.txt`|

### Les Event IDs critiques

|Event ID|Source|Signification|
|---|---|---|
|4624|Security|Logon réussi|
|4625|Security|Logon échoué|
|4720 / 4732|Security|Compte créé / ajout groupe|
|4724 / 4738|Security|Password reset / compte modifié|
|1|Sysmon|Process creation|
|3 / 22|Sysmon|Network connection / DNS query|
|11 / 13|Sysmon|File create / Registry value set|

### La logique de corrélation

Tout s'articule autour de deux identifiants :

- **Logon ID** → relie un `4624` à tous les events Security et Sysmon de la même session
- **ProcessId** → relie un Sysmon Event ID 1 à tous les autres events Sysmon du même process

### Méthodologie d'investigation

1. Identifier le point d'entrée → `4625` (brute force) puis `4624` (logon réussi), noter le **Logon ID**
2. Identifier le process malveillant → Sysmon Event ID 1, vérifier image path + hash (VirusTotal)
3. Tracer les actions du process → filtrer par **ProcessId** (réseau, fichiers, registre)
4. Vérifier les manipulations de comptes → `4720`, `4732`, `4724`
5. Consulter le PowerShell history pour les commandes manuelles

### Red flags universels

- Répertoires suspects : `C:\Temp`, `C:\Users\Public`
- Noms de processus aléatoires : `jqyvpqldou.exe`
- Ports non-standard : `4444`, ou port `80` depuis un process inattendu
- DNS vers `*.top`, `*.click`
- Actions hors heures de travail / week-end
- Noms de comptes hors naming pattern

