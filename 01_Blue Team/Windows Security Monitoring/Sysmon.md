## Sysmon Overview

**Sysmon** est un service Windows + driver kernel. Il **persiste après un redémarrage** et logue en continu : créations de processus, connexions réseau, modifications de temps de fichiers. Events stockés dans : `Applications and Services Logs/Microsoft/Windows/Sysmon/Operational`. Utilisé avec un SIEM pour agrégation/analyse.

### Sysmon Config Overview

Sysmon nécessite un fichier de config. Référence : [SwiftOnSecurity/sysmon-config](https://github.com/SwiftOnSecurity/sysmon-config).

Deux approches :

- **Exclude-heavy** (SwiftOnSecurity) → filtre le bruit, réduit le volume d'events
- **Include-heavy** (ION-Storm fork) → approche proactive, plus d'alertes

29 Event IDs disponibles. Les plus importants :

---

## Syntaxe des règles de config

La structure est toujours la même :

xml

```xml
<EventName onmatch="include|exclude">
    <Champ condition="is|contains|end with|image">valeur</Champ>
</EventName>
```

- **Balise parent** → event ciblé (`ProcessCreate`, `NetworkConnect`…)
- **`onmatch`** → `include` (on logue) / `exclude` (on ignore)
- **Balise enfant** → champ inspecté (`CommandLine`, `DestinationPort`…)
- **`condition`** → `is` (égal), `contains` (contient), `end with` (finit par), `image` (nom du binaire)
- **Valeur** → ce qu'on cherche

### Event ID 1: Process Creation

Surveille les créations de processus. Tags : `CommandLine`, `Image`.

xml

```xml
<ProcessCreate onmatch="exclude">
    <CommandLine condition="is">C:\Windows\system32\svchost.exe -k appmodel -p -s camsvc</CommandLine>
</ProcessCreate>
```

Exclut `svchost.exe` des logs — évite le bruit généré par un processus système légitime et connu.

### Event ID 3: Network Connection

Surveille les connexions réseau. Tags : `Image`, `DestinationPort`.

xml

```xml
<NetworkConnect onmatch="include">
    <Image condition="image">nmap.exe</Image>
    <DestinationPort name="Alert,Metasploit" condition="is">4444</DestinationPort>
</NetworkConnect>
```

Inclut dans les logs toute connexion réseau impliquant `nmap.exe`, ou à destination du port `4444` (port par défaut de Metasploit).

### Event ID 7: Image Loaded

Surveille les DLLs chargées par les processus — utile pour détecter DLL Injection / Hijacking. Attention : génère une charge système élevée. Tags : `Image`, `Signed`, `ImageLoaded`, `Signature`.

xml

```xml
<ImageLoad onmatch="include">
    <ImageLoaded condition="contains">\Temp\</ImageLoaded>
</ImageLoad>
```

Inclut dans les logs toute DLL chargée depuis un répertoire `\Temp\` — chargement depuis Temp est anormal et suspect.

### Event ID 8: CreateRemoteThread

Surveille l'injection de code dans d'autres processus via `CreateRemoteThread`. Fonction légitime, mais couramment détournée par les malwares. Tags : `SourceImage`, `TargetImage`, `StartAddress`, `StartFunction`.

xml

```xml
<CreateRemoteThread onmatch="include">
    <StartAddress name="Alert,Cobalt Strike" condition="end with">0B80</StartAddress>
    <SourceImage condition="contains">\</SourceImage>
</CreateRemoteThread>
```

Deux règles combinées : la première inclut les threads dont l'adresse mémoire se termine par `0B80`, indicateur connu d'un beacon Cobalt Strike. La seconde inclut les processus injectés sans processus parent — anomalie à investiguer.

### Event ID 11: File Created

Logue les fichiers créés ou écrasés sur le disque. Tag : `TargetFilename`.

xml

```xml
<FileCreate onmatch="include">
    <TargetFilename name="Alert,Ransomware" condition="contains">HELP_TO_SAVE_FILES</TargetFilename>
</FileCreate>
```

Inclut dans les logs tout fichier dont le nom contient `HELP_TO_SAVE_FILES` — chaîne typique déposée par des ransomwares.

### Event ID 12 / 13 / 14: Registry Event

Surveille les modifications du registre — vecteur courant de persistance et d'abus de credentials. Tag : `TargetObject`.

xml

```xml
<RegistryEvent onmatch="include">
    <TargetObject name="T1484" condition="contains">Windows\System\Scripts</TargetObject>
</RegistryEvent>
```

Inclut dans les logs toute modification de clé de registre dans `Windows\System\Scripts` — répertoire fréquemment utilisé par les attaquants pour établir une persistance.

### Event ID 15: FileCreateStreamHash

Surveille la création de fichiers dans un **Alternate Data Stream (ADS)** — technique utilisée pour cacher des malwares dans des fichiers NTFS. Tag : `TargetFilename`.

xml

```xml
<FileCreateStreamHash onmatch="include">
    <TargetFilename condition="end with">.hta</TargetFilename>
</FileCreateStreamHash>
```

Inclut dans les logs tout fichier `.hta` créé dans un ADS.

### Event ID 22: DNS Event

Logue toutes les requêtes DNS. Tag : `QueryName`.

xml

```xml
<DnsQuery onmatch="exclude">
    <QueryName condition="end with">.microsoft.com</QueryName>
</DnsQuery>
```

Exclut des logs toutes les requêtes DNS vers `*.microsoft.com` — réduit le bruit des domaines Microsoft légitimes pour ne garder que les anomalies.

---

## Installing Sysmon

Télécharger le binaire depuis [Microsoft Sysinternals](https://docs.microsoft.com/en-us/sysinternals/downloads/sysmon).

Ou installer tous les outils Sysinternals via PowerShell :

powershell

```powershell
Download-SysInternalsTools C:\Sysinternals
```

### Starting Sysmon

Lancer PowerShell ou CMD en administrateur, puis :

powershell

```powershell
Sysmon.exe -accepteula -i ..\Configurations\swift.xml
```

- `-accepteula` → accepte la licence automatiquement
- `-i` → spécifie le fichier de config à utiliser

Events visibles dans : `Applications and Services Logs/Microsoft/Windows/Sysmon/Operational`

> La config peut être changée à tout moment en désinstallant/mettant à jour via le menu d'aide Sysmon.

---

## Malicious Activity Overview

_cf : \Documents\thm\sysmon\Filtering_1610225088511.evtx_

Sysmon permet de détecter : ransomware, persistance, Mimikatz, Metasploit, C2 beacons. L'efficacité repose avant tout sur un fichier de config bien construit.

### Sysmon "Best Practices"

**Exclude > Include** — Exclure en priorité plutôt qu'inclure, pour éviter de rater des events critiques.

**Connaître son environnement** — Comprendre ce qui est normal dans le réseau avant de créer des règles, sinon impossible de distinguer le légitime du suspect.

**CLI > GUI** — `Get-WinEvent` ou `wevtutil.exe` offrent plus de contrôle que l'Event Viewer.

### Filtering Events with Event Viewer

Filtres disponibles : Event ID, keywords, ou XML (fastidieux, peu scalable). Menu : `Filter Current Log` dans le panneau Actions.

### Filtering Events with PowerShell

Utilise `Get-WinEvent` avec des requêtes XPath.

|Filtre|Syntaxe|
|---|---|
|Par Event ID|`*/System/EventID=<ID>`|
|Par attribut XML|`*/EventData/Data[@Name="<attribut>"]`|
|Par valeur|`*/EventData/Data=<valeur>`|

Exemple — connexions réseau sur le port 4444 :

powershell

```powershell
Get-WinEvent -Path <Path> -FilterXPath '*/System/EventID=3 and */EventData/Data[@Name="DestinationPort"] and */EventData/Data=4444'
```

### Questions
#### How many event ID 3 events are in C:\Users\THM-Analyst\Desktop\Scenarios\Practice\Filtering.evtx?

```powershell
Get-WinEvent -Path C:\Users\THM-Analyst\Desktop\Scenarios\Practice\Filtering.evtx -FilterXPath '*/System/EventID=3' | Measure-Object
```

**Answer** : 73,591

#### What is the UTC time of the first network event in the same logfile? Note that UTC time is shown only in the "Details" tab.

in EventViewer import the correct file then go to the details in the first event then copy paste the value ine utctime

![[sysmon t4q3.png]]

---

## Hunting Metasploit

_cf : Documents\thm\sysmon\Hunting_Metasploit_1609814643558.evtx_

Metasploit utilise le port `4444` par défaut (aussi `5555`). Toute connexion vers ces ports, vers une IP connue ou non, doit être investiguée.

#### Hunting Network Connections

Config ION-Storm — inclure les connexions sur les ports Metasploit :

xml

```xml
<NetworkConnect onmatch="include">
    <DestinationPort condition="is">4444</DestinationPort>
    <DestinationPort condition="is">5555</DestinationPort>
</NetworkConnect>
```

#### Hunting for Open Ports with PowerShell

powershell

```powershell
Get-WinEvent -Path <Path> -FilterXPath '*/System/EventID=3 and */EventData/Data[@Name="DestinationPort"] and */EventData/Data=4444'
```

- `EventID=3` → network connection
- `Data[@Name="DestinationPort"]` → filtre sur le champ DestinationPort
- `Data=4444` → valeur recherchée

Un event détecté expose `ProcessID` et `Image` — utiles pour la suite de l'investigation.

---

## Detecting Mimikatz

_cf : Documents\thm\sysmon\Hunting_Mimikatz_1609818374565.evtx_

Mimikatz sert principalement à dumper les credentials depuis la mémoire via **LSASS**. Peut être obfusqué pour bypasser l'AV.

### Detecting File Creation

Détection basique — cherche un fichier nommé `mimikatz` sur le disque :

xml

```xml
<FileCreate onmatch="include">
    <TargetFileName condition="contains">mimikatz</TargetFileName>
</FileCreate>
```

Peu fiable contre un attaquant qui renomme le binaire.

### Hunting Abnormal LSASS Behavior

Tout process autre que `svchost.exe` qui accède à `lsass.exe` est suspect. Config en deux règles — exclure `svchost.exe`, inclure les accès à `lsass.exe` :

xml

```xml
<ProcessAccess onmatch="exclude">
    <SourceImage condition="image">svchost.exe</SourceImage>
</ProcessAccess>
<ProcessAccess onmatch="include">
    <TargetImage condition="image">lsass.exe</TargetImage>
</ProcessAccess>
```

### Detecting LSASS Behavior with PowerShell

powershell

```powershell
Get-WinEvent -Path <Path> -FilterXPath '*/System/EventID=10 and */EventData/Data[@Name="TargetImage"] and */EventData/Data="C:\Windows\system32\lsass.exe"'
```

- `EventID=10` → ProcessAccess
- `TargetImage=lsass.exe` → on cible les accès à LSASS

---
## Hunting Malware

_cf : Documents\thm\sysmon\Hunting_Rats_1609823155048.evtx_

Deux types ciblés : **RATs** (Remote Access Trojans) et **backdoors**.

**RAT** = payload d'accès distant avec interface client-serveur et techniques d'évasion AV intégrées. Exemples : Xeexe, Quasar.

Méthode : **hypothesis-based hunting** — identifier le malware ciblé, puis adapter la config Sysmon en conséquence.

### Hunting RATs and C2 Servers

Même approche que Metasploit — surveiller les ports suspects connus. Config ION-Storm :

xml

```xml
<NetworkConnect onmatch="include">
    <DestinationPort condition="is">1034</DestinationPort>
    <DestinationPort condition="is">1604</DestinationPort>
</NetworkConnect>
<NetworkConnect onmatch="exclude">
    <Image condition="image">OneDrive.exe</Image>
</NetworkConnect>
```

**Limite** : cette config ne détecte que les ports connus (`1034`, `1604`). Un RAT sur un port custom (ex: `8080`) passe inaperçu. Les ports/IPs sont des **IoCs bas dans la Pyramide of Pain** — facilement changés par l'attaquant. Ne jamais s'appuyer uniquement sur des ports connus.

> **Attention** : ION-Storm exclut le port `53` (DNS) par défaut. Or certains malwares utilisent le port `53` pour leurs communications — ne jamais appliquer une config aveuglément.

### Hunting for Common Back Connect Ports with PowerShell

powershell

```powershell
Get-WinEvent -Path <Path> -FilterXPath '*/System/EventID=3 and */EventData/Data[@Name="DestinationPort"] and */EventData/Data=<Port>'
```

Remplacer `<Port>` par le port suspect ciblé (ex: `8080`, `1034`…).

---

## Hunting Persistence

_cf : Documents\thm\sysmon\Persistence Events_1608591416249.zip_

Deux vecteurs ciblés : **dossiers de démarrage** et **clés de registre Run**.

### Hunting Startup Persistence

Surveille les fichiers créés dans `\Start Menu` ou `\Startup\` — MITRE T1547 :

xml

```xml
<FileCreate onmatch="include">
    <TargetFilename name="T1023" condition="contains">\Start Menu</TargetFilename>
    <TargetFilename name="T1165" condition="contains">\Startup\</TargetFilename>
</FileCreate>
```

Filtrer par `RuleName = T1023` dans Event Viewer pour isoler les anomalies.

### Hunting Registry Key Persistence

Surveille les modifications de clés de registre Run — MITRE T1112 :

xml

```xml
<RegistryEvent onmatch="include">
    <TargetObject name="T1060,RunKey" condition="contains">CurrentVersion\Run</TargetObject>
    <TargetObject name="T1484" condition="contains">Group Policy\Scripts</TargetObject>
    <TargetObject name="T1060" condition="contains">CurrentVersion\Windows\Run</TargetObject>
</RegistryEvent>
```

Exemple détecté : `malicious.exe` ajouté dans `HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Run\Persistence` depuis `%windir%\System32\malicious.exe`.

Filtrer par `RuleName = T1060` pour isoler l'anomalie. Investigation : vérifier la clé de registre **et** le fichier à son emplacement.

---

## Detecting Evasion Techniques

_cf : Documents\thm\sysmon\DetectionEvasionEvents-1768847997671.zip_

Techniques couvertes : **Alternate Data Streams (ADS)** et **injections** (DLL Injection, Thread Hijacking, Process Hollowing).

### Hunting Alternate Data Streams

ADS permet de cacher un fichier dans un stream NTFS alternatif, invisible dans l'Explorateur. Event ID 15 logue et hash les streams détectés.

xml

```xml
<FileCreateStreamHash onmatch="include">
    <TargetFilename condition="contains">Downloads</TargetFilename>
    <TargetFilename condition="contains">Temp\7z</TargetFilename>
    <TargetFilename condition="ends with">.hta</TargetFilename>
    <TargetFilename condition="ends with">.bat</TargetFilename>
</FileCreateStreamHash>
```

Exemple concret — `dir /r` révèle un ADS :

```
not_malicious.exe
not_malicious.exe:malware:$DATA  ← fichier caché dans un stream alternatif
```

### Detecting Remote Threads

`CreateRemoteThread` est utilisé pour DLL Injection, Thread Hijacking, Process Hollowing. Config — exclure les sources légitimes connues :

xml

```xml
<CreateRemoteThread onmatch="exclude">
    <SourceImage condition="is">C:\Windows\system32\svchost.exe</SourceImage>
    <TargetImage condition="is">C:\Program Files (x86)\Google\Chrome\Application\chrome.exe</TargetImage>
</CreateRemoteThread>
```

Exemple détecté : `powershell.exe` injecte dans `notepad.exe` via **Reflective PE Injection**.

### Detecting Evasion Techniques with PowerShell

powershell

```powershell
# ADS
Get-WinEvent -Path <Path> -FilterXPath '*/System/EventID=15'

# Remote Threads
Get-WinEvent -Path <Path> -FilterXPath '*/System/EventID=8'
```

La config fait déjà le gros du filtrage — un simple filtre sur l'EventID suffit.

---

## Practical Investigations

_cf : Documents\thm\sysmon\Investigations_1610236531732.zip_

|Investigation|Scénario|Logs|
|---|---|---|
|1|Fichier malveillant déposé via USB|`Investigations\Investigation-1.evtx`|
|2|Fichier masqué en HTML, exécution de code, bypass AV|`Investigations\Investigation-2.evtx`|
|3.1 / 3.2|Persistance établie par l'attaquant|`Investigations\Investigation-3.1.evtx` / `3.2.evtx`|
|4|Communications C2 / botnet|`Investigations\Investigation-4.evtx`|

_cf : https://tryhackme.com/room/sysmon


