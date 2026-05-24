## Command and Control

### Attacks Without C2

RDP breach → commandes tapées directement dans la session. Pas de C2 nécessaire, mais setup C2 immédiat après breach = pratique courante.

### Simplest C2

Pour les autres vecteurs (ex: phishing), un processus malveillant connecte la machine victime au serveur attaquant en permanence.

**Deux variantes :**

- **Simple** : la pièce jointe elle-même établit le canal C2 (ex: CobaltStrike)
- **Avancée** : la pièce jointe télécharge un malware C2 secondaire → le cache (ex: `C:\Temp`) → l'exécute en processus discret. Résistant à la suppression du fichier original.

---

## Persistence Overview

Maintenir un accès fiable à long terme, survivant aux reboots et changements de mots de passe.

![[persitense_overview.png]]

### Persisting via RDP

Service exposé (RDP, mail, web app) → accès répété jusqu'au patch. Méthodes de persistence additionnelles :

- Backdoor / web shell dans le service breché
- Création d'un nouvel utilisateur ([T1136](https://attack.mitre.org/techniques/T1136/)) admin ([T1098](https://attack.mitre.org/techniques/T1098/007/)) pour RDP


```powershell
# 1. Two methods to create the "mr.backd00r" user 
CMD C:\> net user "mr.backd00r" "p@ssw0rd!" /add 
PS C:\> New-LocalUser "mr.backd00r" -Password [...] 

# 2. Two methods to add the user to Administrators 
CMD C:\> net localgroup Administrators "mr.backd00r" /add 
PS C:\> Add-LocalGroupMember "Administrators" -Member "mr.backd00r"
```

### Detecting Backdoored Users

|Event ID|Action|
|---|---|
|4720|Création de compte utilisateur|
|4732|Ajout à un groupe privilégié (Administrators, Remote Desktop Users)|
|4724|Reset de mot de passe (réutilisation d'un compte existant)|

Investigation : qui a créé le compte ? IP source ? Heure ? Autres events suspects dans la même session ?

![[detecting_backdoored_users.png]]

---

## Persistence: Tasks and Services

### Malware Persistence

Persistence via backdoored user → nécessite RDP. Inutilisable si vecteur initial = phishing ou USB.

Dans ces cas : besoin d'un malware actif qui maintient la connexion C2 après reboot.

### Services and Tasks

| Méthode         | Commande                                                                     | Event IDs                               |
| --------------- | ---------------------------------------------------------------------------- | --------------------------------------- |
| Windows Service | `sc create "BadService" binpath= "C:\malware.exe" start= auto`               | Sysmon 1 (sc.exe) / Security 4697       |
| Scheduled Task  | `schtasks /create /tn "BadTask" /tr "C:\malware.exe" /sc onstart /ru System` | Sysmon 1 (schtasks.exe) / Security 4698 |

### Detecting Services

3 méthodes de détection :

- Sysmon Event ID 1 → lancement de `sc.exe create`
- Security Event ID 4697 ou System Event ID 7045 → création de service
- Processus suspect avec parent `services.exe`

![[detecting_services.png]]

### Detecting Tasks

Plus faciles à configurer et cacher → méthode de persistence la plus fréquente ( [APT28(opens in new tab)](https://quointelligence.eu/2020/09/apt28-zebrocy-malware-campaign-nato-theme/#:~:text=Next%2C%20the%20malware%20creates%20a%20new%20scheduled%20task) and [APT41](https://cloud.google.com/blog/topics/threat-intelligence/apt41-us-state-governments/#:~:text=APT41%20has%20leveraged%20the%20following%20Windows%20scheduled%20tasks%20for%20persistence)).

3 méthodes de détection :

- Sysmon Event ID 1 → lancement de `schtasks.exe /create`
- Security Event ID 4698 → création de scheduled task
- Processus suspect avec parent `svchost.exe [...] -s Schedule`

![[detecting_tasks.png]]

---

## Persistence: Run Keys and Startup

Méthodes per-user (pas besoin de privilèges admin) → s'exécutent au login de l'utilisateur.

| Méthode        | Commande                                                                                               | Event ID                   |
| -------------- | ------------------------------------------------------------------------------------------------------ | -------------------------- |
| Startup Folder | `copy C:\malware.exe "%AppData%\Microsoft\Windows\Start Menu\Programs\Startup\malware.exe"`            | Sysmon 11 (file creation)  |
| Run Key        | `reg add "HKCU\Software\Microsoft\Windows\CurrentVersion\Run" /v BadKey /t REG_SZ /d "C:\malware.exe"` | Sysmon 13 (registry value) |
### Detecting Startup

Chemins :

```
# Per-user
C:\Users\<USER>\AppData\Roaming\Microsoft\Windows\Start Menu\Programs\Startup\

# All users
C:\ProgramData\Microsoft\Windows\Start Menu\Programs\StartUp\
```

Dossier normalement vide → toute création suspecte. Parent process : `explorer.exe`

![[detecting_starup.png]]

### Detecting Run Keys

```
# Per-user
HKCU\Software\Microsoft\Windows\CurrentVersion\Run

# All users
HKLM\Software\Microsoft\Windows\CurrentVersion\Run
```

Détecter via Sysmon Event ID 13 sur ces clés. Parent process : `explorer.exe`

![[detecting_run_keys.png]]

---

## Impact and Threat Detection Recap

### Need for Persistence

3 raisons principales :

|Objectif|Exemple réel|
|---|---|
|Botnet (crypto mining, data steal, C2)|Kraken Botnet|
|Espionnage longue durée|Volt Typhoon → 1 an dans le réseau électrique US|
|Pivot réseau / préparation ransomware|IcedID → Dagon Locker en 29 jours|

### Active Directory and Ransomware

Réseau Windows = Active Directory → cible principale : ransomware. Chiffrement des serveurs + exfiltration + ransom notes imprimées sur toutes les imprimantes.

### Threat Detection Recap

Toute attaque complexe commence par une seule brèche simple. Objectif SOC : détecter et stopper dès l'**Initial Access**, avant que l'**Impact** (ransomware) ne se produise.

Ce que couvre la série Windows Threat Detection :

- Comment les breaches commencent
- Comment les attaquants volent les données
- Comment ils restent non détectés (persistence)

![[threat_detection_recap.png]]

---

## Résumé — Windows Threat Detection 3

**Fil conducteur :** après l'Initial Access, l'attaquant établit un C2 et persiste pour rester actif longtemps.

### C2 (Command and Control)

- Sans C2 : commandes directes via RDP
- Avec C2 : processus malveillant connecté en permanence au serveur attaquant
- Variante avancée : attachment → télécharge malware → cache dans `C:\Temp` → exécute

### Persistence — 4 méthodes clés

|Méthode|Détection|
|---|---|
|Backdoor user (compte admin caché)|Security 4720 / 4732 / 4724|
|Windows Service|Sysmon 1 / Security 4697 / System 7045|
|Scheduled Task|Sysmon 1 / Security 4698|
|Startup Folder / Run Keys|Sysmon 11 / Sysmon 13|

### Pourquoi persister ?

1. Intégrer la machine à un botnet
2. Espionnage longue durée (ex: Volt Typhoon)
3. Pivot réseau → ransomware

### Philosophie SOC

Détecter le plus tôt possible → idéalement dès l'**Initial Access**, avant l'**Impact**. Chaque attaque complexe commence par une seule brèche simple.