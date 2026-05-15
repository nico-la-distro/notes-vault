# Commandes Windows — Référence exhaustive

> [!info] Légende Chaque tableau présente les équivalents **PowerShell** et **CMD** côte à côte.
> 
> - Les alias PowerShell courants sont indiqués en note.
> - `—` signifie qu'il n'existe pas d'équivalent natif dans cet environnement.

---

## Table des matières

- [[#Navigation]]
- [[#Lister & explorer]]
- [[#Fichiers & dossiers]]
- [[#Contenu de fichiers]]
- [[#Recherche]]
- [[#Pipeline & texte]]
- [[#Processus]]
- [[#Services]]
- [[#Réseau]]
- [[#Système & environnement]]
- [[#Utilisateurs & droits]]
- [[#Compression & archives]]
- [[#Registre Windows]]
- [[#Tâches planifiées]]
- [[#Divers & utilitaires]]

---

## Navigation

|PowerShell|CMD|Description|
|---|---|---|
|`Set-Location 'C:\dossier'`|`cd C:\dossier`|Changer de répertoire|
|`Set-Location -Path D:\`|`cd /d D:\`|Changer de lecteur (`/d` indispensable en CMD)|
|`Get-Location`|`cd` (sans argument)|Afficher le répertoire courant|
|`Push-Location 'C:\temp'`|`pushd C:\temp`|Sauvegarder le répertoire et changer|
|`Pop-Location`|`popd`|Revenir au répertoire sauvegardé|
|`Set-Location ~`|`cd %USERPROFILE%`|Aller dans le répertoire home|

> [!tip] Alias PowerShell `Set-Location` → `cd`, `chdir`, `sl` — `Get-Location` → `pwd`, `gl`

---

## Lister & explorer

|PowerShell|CMD|Description|
|---|---|---|
|`Get-ChildItem`|`dir`|Lister le contenu du répertoire|
|`Get-ChildItem -Force`|`dir /a`|Inclure les fichiers cachés et système|
|`Get-ChildItem -File`|`dir /a-d`|Fichiers seulement|
|`Get-ChildItem -Directory`|`dir /ad`|Dossiers seulement|
|`Get-ChildItem -Recurse`|`dir /s`|Lister récursivement|
|`Get-ChildItem -Filter '*.txt'`|`dir *.txt`|Filtrer par extension|
|`Get-ChildItem -Recurse -Filter '*.log'`|`dir /s /b *.log`|Recherche récursive par nom (`/b` = noms seuls)|
|`Get-ChildItem \| Sort-Object Length`|`dir /os`|Trier par taille|
|`Get-ChildItem \| Sort-Object LastWriteTime -Descending`|`dir /od`|Trier par date (plus récent en premier)|
|`tree`|`tree`|Arborescence des dossiers|
|`tree /F`|`tree /f`|Arborescence avec les fichiers|

> [!tip] Alias PowerShell `Get-ChildItem` → `ls`, `dir`, `gci`

---

## Fichiers & dossiers

|PowerShell|CMD|Description|
|---|---|---|
|`New-Item -ItemType File 'fichier.txt'`|`type nul > fichier.txt`|Créer un fichier vide|
|`New-Item -ItemType Directory 'dossier'`|`mkdir dossier`|Créer un dossier|
|`New-Item -ItemType Directory -Force -Path 'a\b\c'`|`mkdir a\b\c`|Créer toute une arborescence|
|`Copy-Item 'src.txt' 'dst.txt'`|`copy src.txt dst.txt`|Copier un fichier|
|`Copy-Item 'src.txt' 'C:\dest\'`|`copy src.txt C:\dest\`|Copier vers un dossier|
|`Copy-Item 'dossier' 'dest' -Recurse`|`xcopy dossier dest\ /e /i /y`|Copier un dossier récursivement|
|`Copy-Item -Recurse -Force`|`robocopy src dst /e /z`|Copie robuste (reprend sur erreur réseau)|
|`Move-Item 'src.txt' 'dst.txt'`|`move src.txt dst.txt`|Déplacer ou renommer un fichier|
|`Rename-Item 'ancien.txt' 'nouveau.txt'`|`ren ancien.txt nouveau.txt`|Renommer un fichier|
|`Remove-Item 'fichier.txt'`|`del fichier.txt`|Supprimer un fichier|
|`Remove-Item 'fichier.txt' -Force`|`del /f fichier.txt`|Supprimer en forçant (lecture seule)|
|`Remove-Item '*.tmp' -Force`|`del /f /q *.tmp`|Supprimer en masse silencieusement|
|`Remove-Item 'dossier' -Recurse -Force`|`rmdir /s /q dossier`|Supprimer un dossier et son contenu|
|`Test-Path 'fichier.txt'`|`if exist fichier.txt echo oui`|Vérifier si un fichier existe|
|`Test-Path 'dossier' -PathType Container`|`if exist dossier\ echo oui`|Vérifier si un dossier existe|
|`(Get-Item 'f.txt').Length`|`for %i in (f.txt) do echo %~zi`|Taille d'un fichier en octets|
|`Get-Item 'f.txt' \| Select-Object LastWriteTime`|`for %i in (f.txt) do echo %~ti`|Date de modification|
|`New-Item -ItemType SymbolicLink -Name lien -Target cible`|`mklink lien cible`|Créer un lien symbolique (fichier)|
|`New-Item -ItemType SymbolicLink -Name lien -Target dossier`|`mklink /d lien dossier`|Créer un lien symbolique (dossier)|
|`attrib +r fichier.txt`|`attrib +r fichier.txt`|Mettre en lecture seule|
|`attrib -r -h fichier.txt`|`attrib -r -h fichier.txt`|Enlever lecture seule et masqué|

### robocopy — options essentielles

```cmd
robocopy source destination *.* /e /z /b /r:3 /w:5 /log:copie.log
```

|Option|Description|
|---|---|
|`/e`|Copier les sous-dossiers, y compris les vides|
|`/z`|Mode redémarrable (reprend si coupure réseau)|
|`/b`|Mode backup (contourne les ACL)|
|`/mir`|Miroir — supprime ce qui n'existe plus dans la source|
|`/r:3`|3 tentatives en cas d'échec|
|`/w:5`|Attendre 5 secondes entre les tentatives|
|`/xf *.tmp`|Exclure les fichiers `.tmp`|
|`/xd temp logs`|Exclure les dossiers `temp` et `logs`|
|`/log:copie.log`|Écrire un journal|

> [!tip] Alias PowerShell `New-Item` → `ni` — `Copy-Item` → `cp`, `copy` — `Move-Item` → `mv`, `move` — `Remove-Item` → `rm`, `del`, `erase`

---

## Contenu de fichiers

|PowerShell|CMD|Description|
|---|---|---|
|`Get-Content 'fichier.txt'`|`type fichier.txt`|Afficher le contenu|
|`Get-Content 'fichier.txt' -Raw`|—|Contenu en une seule chaîne|
|`Get-Content 'fichier.txt' -Tail 20`|—|Afficher les 20 dernières lignes (tail)|
|`Get-Content 'fichier.txt' -Head 10`|—|Afficher les 10 premières lignes (head)|
|`Get-Content 'fichier.txt' -Wait`|—|Suivre en temps réel (tail -f)|
|`Get-Content 'fichier.txt' -Encoding UTF8`|`type fichier.txt`|Lire avec encodage spécifique|
|`Set-Content 'f.txt' 'texte'`|`echo texte > f.txt`|Écrire dans un fichier (écrase)|
|`Add-Content 'f.txt' 'ligne'`|`echo ligne >> f.txt`|Ajouter une ligne à la fin|
|`Clear-Content 'f.txt'`|`type nul > f.txt`|Vider un fichier sans le supprimer|
|`(Get-Content 'f.txt').Count`|`find /c /v "" f.txt`|Compter les lignes|
|`Get-Content 'f.txt' \| Measure-Object -Word`|—|Compter les mots|
|`more fichier.txt`|`more fichier.txt`|Afficher avec pagination|

> [!tip] Alias PowerShell `Get-Content` → `cat`, `gc`, `type`

---

## Recherche

|PowerShell|CMD|Description|
|---|---|---|
|`Select-String 'motif' fichier.txt`|`findstr "motif" fichier.txt`|Rechercher un motif dans un fichier|
|`Select-String -Pattern 'motif' -Path '*.txt' -Recurse`|`findstr /s "motif" *.txt`|Recherche récursive dans les fichiers|
|`Select-String -NotMatch 'motif' fichier.txt`|`findstr /v "motif" f.txt`|Lignes ne contenant pas le motif|
|`Select-String -CaseSensitive 'Motif' fichier.txt`|`findstr /c "Motif" f.txt`|Recherche sensible à la casse|
|`Select-String -Pattern 'regex' -AllMatches`|`findstr /r "regex" f.txt`|Utiliser une expression régulière|
|`Select-String 'motif' fichier.txt \| Select-Object LineNumber,Line`|`findstr /n "motif" f.txt`|Afficher les numéros de ligne|
|`Select-String 'motif' *.txt \| Select-Object Filename -Unique`|`findstr /m "motif" *.txt`|Noms des fichiers contenant le motif|
|`Get-ChildItem -Recurse -Filter '*.txt'`|`dir /s /b *.txt`|Trouver des fichiers par nom|
|`Get-ChildItem -Recurse \| Where-Object Length -gt 1MB`|—|Fichiers par taille|
|`Get-ChildItem -Recurse \| Where-Object LastWriteTime -gt (Get-Date).AddDays(-7)`|—|Fichiers modifiés ces 7 derniers jours|
|`where.exe notepad.exe`|`where notepad.exe`|Localiser un exécutable dans le PATH|
|`Get-Command notepad`|`where notepad`|Trouver une commande|

> [!tip] Alias PowerShell `Select-String` → `sls`

---

## Pipeline & texte

|PowerShell|CMD|Description|
|---|---|---|
|`... \| Where-Object { $_.Name -like 'a*' }`|`... \| findstr /i "^a"`|Filtrer les lignes ou objets|
|`... \| Sort-Object`|`... \| sort`|Trier|
|`... \| Sort-Object -Descending`|`... \| sort /r`|Trier en ordre inverse|
|`... \| Sort-Object -Unique`|`... \| sort /unique`|Trier et dédupliquer|
|`... \| Select-Object -First 10`|—|Garder les N premiers éléments|
|`... \| Select-Object -Last 10`|—|Garder les N derniers éléments|
|`... \| Select-Object -Skip 5`|—|Ignorer les N premiers éléments|
|`... \| Measure-Object`|`... \| find /c ""`|Compter les éléments|
|`... \| Measure-Object -Sum -Average -Maximum -Minimum`|—|Statistiques sur une propriété numérique|
|`... \| Select-Object -ExpandProperty Name`|—|Extraire une seule propriété|
|`... \| Format-Table -AutoSize`|—|Afficher en tableau formaté|
|`... \| Format-List`|—|Afficher en liste propriété : valeur|
|`... \| Format-Wide -Column 4`|—|Afficher en colonnes larges|
|`... \| Out-GridView`|—|Afficher dans une fenêtre graphique filtrable|
|`... \| Out-File 'f.txt' -Encoding UTF8`|`... > f.txt`|Rediriger la sortie vers un fichier|
|`... \| Out-File 'f.txt' -Append`|`... >> f.txt`|Ajouter à un fichier existant|
|`... \| Tee-Object -FilePath 'f.txt'`|`... \| tee f.txt`|Afficher ET écrire dans un fichier|
|`... \| clip`|`... \| clip`|Copier la sortie dans le presse-papier|
|`... \| ConvertTo-Csv`|—|Convertir en CSV|
|`... \| ConvertTo-Json`|—|Convertir en JSON|
|`... \| ConvertTo-Html`|—|Convertir en HTML|
|`... \| Export-Csv 'f.csv' -NoTypeInformation`|—|Exporter en fichier CSV|

> [!tip] Alias PowerShell `Where-Object` → `?`, `where` — `ForEach-Object` → `%`, `foreach`

---

## Processus

|PowerShell|CMD|Description|
|---|---|---|
|`Get-Process`|`tasklist`|Lister tous les processus|
|`Get-Process chrome`|`tasklist /fi "imagename eq chrome.exe"`|Filtrer par nom|
|`Get-Process \| Sort-Object CPU -Descending \| Select-Object -First 10`|—|Top 10 par consommation CPU|
|`Get-Process \| Sort-Object WorkingSet -Descending \| Select-Object -First 10`|—|Top 10 par consommation mémoire|
|`Get-Process \| Where-Object CPU -gt 10`|—|Processus consommant plus de 10s CPU|
|`(Get-Process chrome).Id`|`tasklist /fi "imagename eq chrome.exe" /fo csv`|PID d'un processus par nom|
|`Stop-Process -Name 'notepad'`|`taskkill /im notepad.exe`|Tuer un processus par nom|
|`Stop-Process -Id 1234 -Force`|`taskkill /pid 1234 /f`|Tuer un processus par PID|
|`Stop-Process -Name 'chrome' -Force`|`taskkill /im chrome.exe /f /t`|Tuer un processus et ses enfants|
|`Start-Process 'notepad.exe'`|`start notepad.exe`|Lancer un programme|
|`Start-Process 'prog.exe' -Wait`|`start /wait prog.exe`|Lancer et attendre la fin|
|`Start-Process 'prog.exe' -WindowStyle Hidden`|`start /b prog.exe`|Lancer en arrière-plan|
|`Start-Process 'prog.exe' -Verb RunAs`|`runas /user:Administrateur prog.exe`|Lancer en tant qu'administrateur|
|`Start-Process 'prog.exe' -ArgumentList '/arg1','/arg2'`|`start prog.exe /arg1 /arg2`|Lancer avec des arguments|
|`$PID`|`echo %ERRORLEVEL%` (approx)|PID du processus courant|
|`Wait-Process -Id 1234`|—|Attendre la fin d'un processus|

> [!tip] Alias PowerShell `Get-Process` → `ps`, `gps` — `Stop-Process` → `kill`, `spps`

---

## Services

|PowerShell|CMD / sc|Description|
|---|---|---|
|`Get-Service`|`net start` / `sc query type= all`|Lister tous les services|
|`Get-Service \| Where-Object Status -eq 'Running'`|`net start`|Services en cours d'exécution|
|`Get-Service \| Where-Object Status -eq 'Stopped'`|`sc query state= inactive`|Services arrêtés|
|`Get-Service -Name Spooler`|`sc query Spooler`|État d'un service|
|`Start-Service -Name Spooler`|`net start Spooler`|Démarrer un service|
|`Stop-Service -Name Spooler`|`net stop Spooler`|Arrêter un service|
|`Stop-Service -Name Spooler -Force`|`sc stop Spooler`|Arrêter un service (forcer)|
|`Restart-Service -Name Spooler`|`net stop Spooler && net start Spooler`|Redémarrer un service|
|`Suspend-Service -Name Spooler`|`net pause Spooler`|Suspendre un service|
|`Resume-Service -Name Spooler`|`net continue Spooler`|Reprendre un service suspendu|
|`Set-Service -Name Spooler -StartupType Automatic`|`sc config Spooler start= auto`|Démarrage automatique|
|`Set-Service -Name Spooler -StartupType Manual`|`sc config Spooler start= demand`|Démarrage manuel|
|`Set-Service -Name Spooler -StartupType Disabled`|`sc config Spooler start= disabled`|Désactiver un service|
|`New-Service -Name 'MonService' -BinaryPathName 'C:\app\svc.exe'`|`sc create MonService binPath= "C:\app\svc.exe"`|Créer un service|
|`Remove-Service -Name 'MonService'`|`sc delete MonService`|Supprimer un service|

---

## Réseau

|PowerShell|CMD|Description|
|---|---|---|
|`Test-Connection google.com`|`ping google.com`|Ping|
|`Test-Connection google.com -Count 4 -Quiet`|`ping -n 4 google.com`|Ping silencieux (retourne vrai/faux)|
|`Test-NetConnection -ComputerName host -Port 80`|—|Tester si un port est ouvert|
|`Test-NetConnection host -TraceRoute`|`tracert host`|Tracer la route vers un hôte|
|`Resolve-DnsName google.com`|`nslookup google.com`|Résolution DNS|
|`Resolve-DnsName google.com -Type MX`|`nslookup -type=MX google.com`|Résolution d'un type précis|
|`Get-NetIPAddress`|`ipconfig`|Configuration réseau|
|`Get-NetIPAddress -AddressFamily IPv4`|`ipconfig \| findstr IPv4`|Adresses IPv4 seulement|
|`Get-NetIPConfiguration`|`ipconfig /all`|Configuration complète|
|`Clear-DnsClientCache`|`ipconfig /flushdns`|Vider le cache DNS|
|`Get-DnsClientCache`|`ipconfig /displaydns`|Afficher le cache DNS|
|`Get-NetTCPConnection`|`netstat -an`|Connexions et ports ouverts|
|`Get-NetTCPConnection -State Listen`|`netstat -an \| findstr LISTEN`|Ports en écoute|
|`Get-NetTCPConnection -LocalPort 80`|`netstat -an \| findstr ":80 "`|Processus sur un port spécifique|
|`Get-NetRoute`|`route print`|Table de routage|
|`Get-NetNeighbor`|`arp -a`|Table ARP|
|`Get-NetAdapter`|`ipconfig /all`|Adaptateurs réseau|
|`Get-NetAdapter \| Where-Object Status -eq 'Up'`|—|Adaptateurs connectés|
|`Disable-NetAdapter -Name 'Ethernet'`|`netsh interface set interface "Ethernet" disabled`|Désactiver un adaptateur|
|`Enable-NetAdapter -Name 'Ethernet'`|`netsh interface set interface "Ethernet" enabled`|Activer un adaptateur|
|`Invoke-WebRequest 'https://url' -OutFile 'fichier'`|`certutil -urlcache -split -f url fichier`|Télécharger un fichier|
|`(Invoke-WebRequest 'https://url').Content`|—|Récupérer le contenu d'une URL|
|`Invoke-RestMethod 'https://api/endpoint'`|—|Appel REST (JSON auto-parsé)|
|`New-PSDrive -Name Z -PSProvider FileSystem -Root '\\srv\share'`|`net use Z: \\srv\share`|Monter un partage réseau|
|`Remove-PSDrive -Name Z`|`net use Z: /delete`|Démonter un partage réseau|
|`Get-SmbShare`|`net share`|Partages locaux|

---

## Système & environnement

|PowerShell|CMD|Description|
|---|---|---|
|`$env:USERNAME`|`echo %USERNAME%`|Nom d'utilisateur courant|
|`$env:COMPUTERNAME`|`echo %COMPUTERNAME%`|Nom de l'ordinateur|
|`$env:PATH`|`echo %PATH%`|Variable PATH|
|`$env:TEMP`|`echo %TEMP%`|Répertoire temporaire|
|`$env:USERPROFILE`|`echo %USERPROFILE%`|Répertoire home|
|`$env:APPDATA`|`echo %APPDATA%`|Répertoire AppData\Roaming|
|`$env:PROGRAMFILES`|`echo %PROGRAMFILES%`|Répertoire Program Files|
|`$env:PATH += ';C:\mon\chemin'`|`set PATH=%PATH%;C:\mon\chemin`|Ajouter au PATH (session courante)|
|`[System.Environment]::SetEnvironmentVariable('X','v','Machine')`|`setx X valeur /m`|Définir une variable d'env. permanente|
|`[System.Environment]::GetEnvironmentVariable('PATH','Machine')`|`reg query "HKLM\SYSTEM\...\Environment" /v PATH`|Lire une variable d'env. permanente|
|`Get-Date`|`echo %DATE% && echo %TIME%`|Date et heure courantes|
|`Get-Date -Format 'yyyy-MM-dd HH:mm:ss'`|`powershell -c "Get-Date -Format ..."`|Date formatée ISO|
|`hostname`|`hostname`|Nom de l'hôte|
|`$PSVersionTable`|`ver`|Version de PowerShell / Windows|
|`Get-ComputerInfo`|`systeminfo`|Informations système complètes|
|`(Get-CimInstance Win32_OperatingSystem).Caption`|`wmic os get Caption`|Nom de l'OS|
|`(Get-CimInstance Win32_ComputerSystem).TotalPhysicalMemory`|`wmic memorychip get Capacity`|Mémoire RAM totale|
|`Get-CimInstance Win32_Processor`|`wmic cpu get Name,NumberOfCores`|Informations CPU|
|`Get-CimInstance Win32_LogicalDisk`|`wmic logicaldisk get Caption,FreeSpace,Size`|Espace disque|
|`Get-PSDrive`|`wmic logicaldisk list brief`|Lecteurs disponibles|
|`[System.Math]::Round(3.14159, 2)`|`set /a 3+0` (entiers seulement)|Calcul arithmétique|
|`Start-Sleep -Seconds 5`|`timeout /t 5 /nobreak`|Attendre N secondes|
|`Clear-Host`|`cls`|Effacer l'écran|
|`Restart-Computer -Force`|`shutdown /r /t 0`|Redémarrer immédiatement|
|`Stop-Computer -Force`|`shutdown /s /t 0`|Éteindre immédiatement|
|`exit`|`exit`|Quitter le terminal|

---

## Utilisateurs & droits

### Utilisateurs locaux

|PowerShell|CMD|Description|
|---|---|---|
|`Get-LocalUser`|`net user`|Lister les utilisateurs locaux|
|`Get-LocalUser -Name alice`|`net user alice`|Détails d'un utilisateur|
|`New-LocalUser -Name 'bob' -Password (Read-Host -AsSecureString) -FullName 'Bob Martin'`|`net user bob * /add /fullname:"Bob Martin"`|Créer un utilisateur|
|`Set-LocalUser -Name 'bob' -FullName 'Robert Martin'`|`net user bob /fullname:"Robert Martin"`|Modifier un utilisateur|
|`Set-LocalUser -Name 'bob' -Password (Read-Host -AsSecureString)`|`net user bob *`|Changer le mot de passe|
|`Enable-LocalUser -Name 'bob'`|`net user bob /active:yes`|Activer un compte|
|`Disable-LocalUser -Name 'bob'`|`net user bob /active:no`|Désactiver un compte|
|`Remove-LocalUser -Name 'bob'`|`net user bob /delete`|Supprimer un utilisateur|

### Groupes locaux

|PowerShell|CMD|Description|
|---|---|---|
|`Get-LocalGroup`|`net localgroup`|Lister les groupes locaux|
|`Get-LocalGroupMember -Group Administrators`|`net localgroup Administrators`|Membres d'un groupe|
|`Add-LocalGroupMember -Group Administrators -Member bob`|`net localgroup Administrators bob /add`|Ajouter au groupe Administrateurs|
|`Remove-LocalGroupMember -Group Administrators -Member bob`|`net localgroup Administrators bob /delete`|Retirer du groupe Administrateurs|
|`New-LocalGroup -Name 'MonGroupe'`|`net localgroup MonGroupe /add`|Créer un groupe local|
|`Remove-LocalGroup -Name 'MonGroupe'`|`net localgroup MonGroupe /delete`|Supprimer un groupe local|

### Identité & permissions

|PowerShell|CMD|Description|
|---|---|---|
|`whoami`|`whoami`|Identité courante|
|`whoami /priv`|`whoami /priv`|Privilèges de l'utilisateur courant|
|`whoami /groups`|`whoami /groups`|Groupes de l'utilisateur courant|
|`Get-Acl 'fichier.txt'`|`icacls fichier.txt`|Voir les permissions d'un fichier|
|`Get-Acl 'fichier.txt' \| Format-List`|`icacls fichier.txt`|Détail des permissions|
|`Set-Acl 'fichier.txt' $acl`|`icacls f.txt /grant bob:(F)`|Modifier les permissions|
|`icacls 'C:\dossier' /grant 'Users:(OI)(CI)RX'`|`icacls C:\dossier /grant Users:(OI)(CI)RX`|Droits récursifs sur un dossier|
|`icacls 'C:\dossier' /reset /t`|`icacls C:\dossier /reset /t`|Réinitialiser les permissions|
|`Start-Process ... -Verb RunAs`|`runas /user:Administrateur cmd`|Exécuter en tant qu'autre utilisateur|

---

## Compression & archives

|PowerShell|CMD|Description|
|---|---|---|
|`Compress-Archive -Path 'C:\src' -DestinationPath 'archive.zip'`|`powershell -c "Compress-Archive ..."`|Créer une archive ZIP|
|`Compress-Archive -Path 'C:\src\*' -DestinationPath 'archive.zip'`|—|Compresser le contenu d'un dossier|
|`Compress-Archive -Path 'f.txt' -DestinationPath 'arch.zip' -Update`|—|Ajouter un fichier à une archive existante|
|`Compress-Archive -CompressionLevel Optimal -Path 'src' -DestinationPath 'arch.zip'`|—|Compression maximale|
|`Expand-Archive -Path 'archive.zip' -DestinationPath 'C:\dest'`|`expand archive.zip C:\dest` (fichiers seuls)|Extraire une archive ZIP|
|`Expand-Archive -Path 'archive.zip' -DestinationPath 'C:\dest' -Force`|—|Extraire en écrasant|

```powershell
# Lister le contenu d'un ZIP sans extraire
Add-Type -AssemblyName System.IO.Compression.FileSystem
[IO.Compression.ZipFile]::OpenRead('archive.zip').Entries | Select-Object Name,Length
```

---

## Registre Windows

|PowerShell|CMD / reg|Description|
|---|---|---|
|`Get-Item 'HKLM:\SOFTWARE\Microsoft\Windows NT\CurrentVersion'`|`reg query "HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion"`|Lire une clé de registre|
|`Get-ItemProperty 'HKLM:\...' -Name 'ProductName'`|`reg query "HKLM\..." /v ProductName`|Lire une valeur spécifique|
|`Get-ChildItem 'HKLM:\SOFTWARE\'`|`reg query HKLM\SOFTWARE`|Lister les sous-clés|
|`Set-ItemProperty 'HKCU:\...' -Name 'cle' -Value 'valeur'`|`reg add "HKCU\..." /v cle /d valeur /f`|Créer ou modifier une valeur (REG_SZ)|
|`Set-ItemProperty 'HKCU:\...' -Name 'cle' -Value 1 -Type DWord`|`reg add "HKCU\..." /v cle /t REG_DWORD /d 1 /f`|Créer une valeur DWORD|
|`New-Item 'HKCU:\Software\MonApp' -Force`|`reg add "HKCU\Software\MonApp"`|Créer une clé|
|`Remove-ItemProperty 'HKCU:\...' -Name 'cle'`|`reg delete "HKCU\..." /v cle /f`|Supprimer une valeur|
|`Remove-Item 'HKCU:\Software\MonApp' -Recurse`|`reg delete "HKCU\Software\MonApp" /f`|Supprimer une clé et ses sous-clés|
|`Test-Path 'HKCU:\Software\MonApp'`|`reg query "HKCU\Software\MonApp" >nul 2>&1`|Vérifier si une clé existe|
|`reg export "HKCU\Software\MonApp" sauvegarde.reg`|`reg export "HKCU\Software\MonApp" sauvegarde.reg`|Exporter vers un fichier .reg|
|`reg import sauvegarde.reg`|`reg import sauvegarde.reg`|Importer depuis un fichier .reg|

### Types de valeurs de registre

|Type|Alias reg|Description|
|---|---|---|
|`String`|`REG_SZ`|Chaîne de caractères|
|`ExpandString`|`REG_EXPAND_SZ`|Chaîne avec variables d'environnement|
|`Binary`|`REG_BINARY`|Données binaires|
|`DWord`|`REG_DWORD`|Entier 32 bits|
|`QWord`|`REG_QWORD`|Entier 64 bits|
|`MultiString`|`REG_MULTI_SZ`|Tableau de chaînes|

---

## Tâches planifiées

|PowerShell|CMD / schtasks|Description|
|---|---|---|
|`Get-ScheduledTask`|`schtasks /query`|Lister toutes les tâches|
|`Get-ScheduledTask -TaskName 'MaTache'`|`schtasks /query /tn MaTache /fo LIST`|Détails d'une tâche|
|`Get-ScheduledTask \| Where-Object State -eq 'Ready'`|`schtasks /query /fo CSV \| findstr "Ready"`|Tâches activées|
|`Start-ScheduledTask -TaskName 'MaTache'`|`schtasks /run /tn MaTache`|Exécuter une tâche manuellement|
|`Stop-ScheduledTask -TaskName 'MaTache'`|`schtasks /end /tn MaTache`|Arrêter une tâche en cours|
|`Enable-ScheduledTask -TaskName 'MaTache'`|`schtasks /change /tn MaTache /enable`|Activer une tâche|
|`Disable-ScheduledTask -TaskName 'MaTache'`|`schtasks /change /tn MaTache /disable`|Désactiver une tâche|
|`Unregister-ScheduledTask -TaskName 'MaTache' -Confirm:$false`|`schtasks /delete /tn MaTache /f`|Supprimer une tâche|

```powershell
# Créer une tâche planifiée complète
$action    = New-ScheduledTaskAction -Execute 'PowerShell.exe' -Argument '-File C:\scripts\backup.ps1'
$trigger   = New-ScheduledTaskTrigger -Daily -At '03:00'
$settings  = New-ScheduledTaskSettingsSet -RunOnlyIfNetworkAvailable -WakeToRun
$principal = New-ScheduledTaskPrincipal -UserId 'SYSTEM' -RunLevel Highest

Register-ScheduledTask -TaskName 'Backup-Nuit' `
    -Action $action `
    -Trigger $trigger `
    -Settings $settings `
    -Principal $principal `
    -Description 'Sauvegarde nocturne'
```

```cmd
:: CMD — créer une tâche planifiée quotidienne
schtasks /create /tn "Backup-Nuit" /tr "powershell -File C:\scripts\backup.ps1" /sc daily /st 03:00 /ru SYSTEM /f
```

---

## Divers & utilitaires

|PowerShell|CMD|Description|
|---|---|---|
|`Get-Clipboard`|—|Lire le presse-papier|
|`Set-Clipboard 'texte'`|`echo texte \| clip`|Écrire dans le presse-papier|
|`Get-History`|`doskey /history`|Historique des commandes|
|`Get-History \| Select-Object -Last 20`|—|20 dernières commandes|
|`Invoke-History 42`|—|Rappeler la commande n°42|
|`Clear-History`|—|Vider l'historique de la session|
|`Get-Command *process*`|—|Trouver des commandes par mot-clé|
|`Get-Command -Module ActiveDirectory`|—|Toutes les commandes d'un module|
|`Get-Help Get-Process -Full`|`commande /?`|Aide complète d'une commande|
|`Get-Help Get-Process -Examples`|—|Exemples uniquement|
|`Get-Alias ls`|—|À quoi correspond un alias|
|`New-Alias ll Get-ChildItem`|`doskey ll=dir $*`|Créer un alias|
|`Get-Member`|—|Propriétés et méthodes d'un objet|
|`Measure-Command { Get-ChildItem -Recurse }`|—|Mesurer le temps d'exécution|
|`& 'C:\chemin\prog.exe' arg1 arg2`|`C:\chemin\prog.exe arg1 arg2`|Exécuter un programme|
|`$LASTEXITCODE`|`echo %ERRORLEVEL%`|Code de retour de la dernière commande|
|`$?`|—|Succès de la dernière commande (vrai/faux)|
|`Write-Host 'texte' -ForegroundColor Green`|`color 0A` (tout l'écran)|Affichage coloré|
|`Read-Host 'Votre nom'`|`set /p NOM=Votre nom :`|Lire une saisie utilisateur|
|`[System.Console]::Beep(440, 500)`|—|Émettre un bip sonore|
|`[System.Windows.Forms.MessageBox]::Show('Message')`|`msg * "Message"`|Afficher une boîte de dialogue|
|`notepad $PROFILE`|—|Ouvrir le profil PowerShell|
|`$PROFILE`|—|Chemin du fichier de profil PS|

---

## 🔗 Voir aussi

- `Get-Help about_*` — documentation intégrée complète
- `Get-Verb` — liste des verbes PowerShell approuvés
- [docs.microsoft.com/powershell](https://docs.microsoft.com/en-us/powershell/)
- [ss64.com/nt](https://ss64.com/nt/) — référence CMD complète