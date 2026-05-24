## Task Manager

Utilitaire GUI Windows natif : visualiser les processus, l'usage CPU/mémoire, tuer un processus.

**Accès** : clic droit sur la Taskbar → Task Manager → More details

**Colonnes utiles à activer** (clic droit sur header) :

|Colonne|Utilité|
|---|---|
|PID|Identifiant unique par processus|
|Type|Apps / Background / Windows process|
|Publisher|Auteur du programme|
|Command line|Commande complète de lancement|
|Image path name|Chemin de l'exécutable|

**Limitation critique** : pas de vue Parent-Child → impossible de vérifier le processus parent. Exemple : `svchost.exe` devrait toujours être enfant de `services.exe`. Task Manager ne permet pas de le confirmer.

**Alternatives** : Process Hacker, Process Explorer (affichent la hiérarchie parent-enfant).

**process hacker**

![[process_hacker.png]]

**process explorer**

![[process_explorer.png]]

**Équivalents CLI** :

```
tasklist
Get-Process   # PowerShell
ps            # PowerShell
wmic
```

---

## System

**PID fixe : toujours 4**

Héberge les threads kernel-mode (s'exécutent uniquement en kernel space : `Ntoskrnl.exe`, drivers). Pas d'espace d'adressage user-mode.

**what is user-mode and kernel-mode** : https://learn.microsoft.com/en-us/windows-hardware/drivers/gettingstarted/user-mode-and-kernel-mode

**Comportement normal** :

|Attribut|Valeur|
|---|---|
|Image Path|`C:\Windows\system32\ntoskrnl.exe`|
|Parent Process|System Idle Process (PID 0)|
|Instances|1|
|User Account|Local System|
|Start Time|Au boot|
|Session|0|

**Signes suspects** :

- Parent process ≠ System Idle Process (0)
- Plus d'une instance
- PID ≠ 4
- Session ≠ 0

---
## User Mode vs Kernel Mode

|                | User Mode                              | Kernel Mode                             |
| -------------- | -------------------------------------- | --------------------------------------- |
| Espace mémoire | Privé par processus                    | Partagé entre tous                      |
| Isolation      | Oui — les processus ne s'affectent pas | Non — un driver peut corrompre un autre |
| Accès OS       | Interdit                               | Total                                   |
| Crash          | Tue le processus seulement             | Crashe tout le système (BSOD)           |
| Exemples       | Applications, logiciels                | Drivers, `Ntoskrnl.exe`                 |

![[user_vs_kernel_mode.png]]

---

## System > smss.exe

**Session Manager Subsystem** — premier processus user-mode lancé par le kernel.

Rôle : créer les sessions Windows, lancer les sous-systèmes, définir les variables d'environnement, créer les fichiers de pagination (virtual memory).

you can read more about the NT Architecture [here](https://en.wikipedia.org/wiki/Architecture_of_Windows_NT)

![[windows architecture.png]]

**Sessions créées** :

|Session|Processus lancés|
|---|---|
|Session 0 (OS)|`csrss.exe` + `wininit.exe`|
|Session 1 (User)|`csrss.exe` + `winlogon.exe`|

Pour chaque nouvelle session : `smss.exe` se copie, crée la session, puis se termine.

**Comportement normal** :

|Attribut|Valeur|
|---|---|
|Image Path|`C:\Windows\System32\smss.exe`|
|Parent Process|System (PID 4)|
|Instances|1 master (les enfants se terminent)|
|User Account|Local System|
|Start Time|Quelques secondes après le boot|

**Signes suspects** :

- Parent ≠ System (4)
- Image path ≠ `C:\Windows\System32`
- Plus d'une instance active
- User ≠ SYSTEM
- Entrées registry inattendues dans `HKLM\System\CurrentControlSet\Control\Session Manager\Subsystems`

---

## csrss.exe

**Client Server Runtime Process** — côté user-mode du sous-système Windows. Processus critique : le tuer = crash système.

Rôle : gestion des fenêtres console, création/suppression de threads, rendre l'API Windows disponible, mapping des lettres de lecteur, gestion du shutdown.

**Comportement normal** :

|Attribut|Valeur|
|---|---|
|Image Path|`C:\Windows\System32\csrss.exe`|
|Parent Process|Non visible (smss.exe se termine avant)|
|Instances|2 minimum (Session 0 + Session 1)|
|User Account|Local System|
|Start Time|Quelques secondes après le boot|

**Signes suspects** :

- Un parent process visible (smss.exe devrait être déjà terminé)
- Image path ≠ `C:\Windows\System32`
- Faute d'orthographe subtile (`csrsss.exe`, `cssrs.exe`...)
- User ≠ SYSTEM

---

## wininit.exe

**Windows Initialization Process** — tourne dans Session 0, lance les processus système critiques.

Lance : `services.exe`, `lsass.exe`, `lsaiso.exe` (ce dernier uniquement si Credential Guard est activé).

**Comportement normal** :

|Attribut|Valeur|
|---|---|
|Image Path|`C:\Windows\System32\wininit.exe`|
|Parent Process|Non visible (smss.exe se termine avant)|
|Instances|1|
|User Account|Local System|
|Start Time|Quelques secondes après le boot|

**Signes suspects** :

- Un parent process visible
- Image path ≠ `C:\Windows\System32`
- Faute d'orthographe subtile
- Plus d'une instance
- User ≠ SYSTEM

---
## wininit.exe > services.exe

**Service Control Manager (SCM)** — gère tous les services Windows (chargement, démarrage, arrêt).

- Base de données des services : `HKLM\System\CurrentControlSet\Services`
- Interrogeable via `sc.exe`
- Charge les drivers marqués auto-start en mémoire
- Met à jour `HKLM\System\Select\LastKnownGood` à chaque connexion réussie

Parent de : `svchost.exe`, `spoolsv.exe`, `msmpeng.exe`, `dllhost.exe`

**Comportement normal** :

|Attribut|Valeur|
|---|---|
|Image Path|`C:\Windows\System32\services.exe`|
|Parent Process|`wininit.exe`|
|Instances|1|
|User Account|Local System|
|Start Time|Quelques secondes après le boot|

**Signes suspects** :

- Parent ≠ `wininit.exe`
- Image path ≠ `C:\Windows\System32`
- Faute d'orthographe subtile
- Plus d'une instance
- User ≠ SYSTEM

---

## wininit.exe > services.exe > svchost.exe

**Service Host** — héberge et gère les services Windows implémentés comme DLLs.

La DLL de chaque service est stockée dans :

```
HKLM\SYSTEM\CurrentControlSet\Services\<SERVICE NAME>\Parameters\ServiceDLL
```

**Paramètre clé : `-k`** Toute instance légitime de svchost.exe est appelée avec `-k` dans sa binary path. `-k` groupe les services similaires dans un même processus pour réduire la consommation de ressources. Depuis Windows 10 v1703 sur machines > 3.5 GB RAM : chaque service tourne dans son propre processus.

**Cible fréquente de malware** : usurpation du nom (`scvhost.exe`, `svchost32.exe`) ou injection d'une DLL malveillante.

**Comportement normal** :

|Attribut|Valeur|
|---|---|
|Image Path|`C:\Windows\System32\svchost.exe`|
|Parent Process|`services.exe`|
|Instances|Nombreuses|
|User Account|SYSTEM, Network Service, Local Service, ou user connecté|
|Start Time|Secondes après le boot ou plus tard|

**Signes suspects** :

- Parent ≠ `services.exe`
- Image path ≠ `C:\Windows\System32`
- Faute d'orthographe subtile
- Absence du paramètre `-k`

---

## lsass.exe

**Local Security Authority Subsystem Service** — applique la politique de sécurité Windows.

Rôle : authentification des connexions, changements de mots de passe, création des access tokens, écriture dans le Security Log. Crée les tokens pour : SAM, Active Directory, NETLOGON. Config auth : `HKLM\System\CurrentControlSet\Control\Lsa`

**Cible fréquente** : dump de credentials via `mimikatz`, ou usurpation du nom.

**Comportement normal** :

|Attribut|Valeur|
|---|---|
|Image Path|`C:\Windows\System32\lsass.exe`|
|Parent Process|`wininit.exe`|
|Instances|1|
|User Account|Local System|
|Start Time|Quelques secondes après le boot|

**Signes suspects** :

- Parent ≠ `wininit.exe`
- Image path ≠ `C:\Windows\System32`
- Faute d'orthographe subtile
- Plus d'une instance
- User ≠ SYSTEM

---

## winlogon.exe

**Windows Logon** — gère la Secure Attention Sequence (SAS) : `CTRL+ALT+DELETE`.

Rôle : authentification utilisateur, chargement du profil (`NTUSER.DAT` → `HKCU`), lancement du shell via `userinit.exe`, verrouillage d'écran, screensaver.

Lancé par `smss.exe` dans Session 1 (avec `csrss.exe`).

**Comportement normal** :

|Attribut|Valeur|
|---|---|
|Image Path|`C:\Windows\System32\winlogon.exe`|
|Parent Process|Non visible (smss.exe se termine avant)|
|Instances|1+ (1 par session RDP ou Fast User Switching)|
|User Account|Local System|
|Start Time|Quelques secondes après le boot|

**Signes suspects** :

- Un parent process visible
- Image path ≠ `C:\Windows\System32`
- Faute d'orthographe subtile
- User ≠ SYSTEM
- Valeur `Shell` dans la registry ≠ `explorer.exe`

---

## explorer.exe

**Windows Explorer** — interface utilisateur : accès aux fichiers, dossiers, Start Menu, Taskbar.

Lancé par `userinit.exe` (qui se termine ensuite) via la valeur registry : `HKLM\Software\Microsoft\Windows NT\CurrentVersion\Winlogon\Shell`

**Comportement normal** :

|Attribut|Valeur|
|---|---|
|Image Path|`C:\Windows\explorer.exe`|
|Parent Process|Non visible (`userinit.exe` se termine avant)|
|Instances|1+ par utilisateur connecté interactivement|
|User Account|L'utilisateur connecté|
|Start Time|À la première connexion interactive|

**Signes suspects** :

- Un parent process visible
- Image path ≠ `C:\Windows`
- User inconnu
- Faute d'orthographe subtile
- Connexions TCP/IP sortantes

---

## Conclusion

The information for this room is derived from multiple sources.

- [https://0xcybery.github.io/blog/Core-Processes-In-Windows-System(opens in new tab)](https://0xcybery.github.io/blog/Core-Processes-In-Windows-System)[(opens in new tab)](https://www.threathunting.se/tag/windows-process/)
- [https://www.sans.org/posters/hunt-evil/(opens in new tab)](https://www.sans.org/posters/hunt-evil/)[(opens in new tab)](https://www.sans.org/security-resources/posters/hunt-evil/165/download)
- [https://docs.microsoft.com/en-us/sysinternals/resources/windows-internals](https://docs.microsoft.com/en-us/sysinternals/resources/windows-internals)

---

## Résumé

**L'idée centrale de toute la room :** Les processus Windows légitimes ont toujours les mêmes caractéristiques (parent, chemin, nombre d'instances). Un malware va imiter ces processus en changeant un petit détail.


**La chaîne de démarrage Windows :**

```
System (PID 4)
  └── smss.exe
        ├── csrss.exe      (Session 0 et 1)
        ├── wininit.exe    (Session 0)
        │     ├── services.exe
        │     │     └── svchost.exe
        │     └── lsass.exe
        └── winlogon.exe   (Session 1)
              └── userinit.exe
                    └── explorer.exe
```

**Les 4 signes suspects valables pour TOUS les processus :**

|Quoi vérifier|Suspect si...|
|---|---|
|Parent process|Pas le bon parent|
|Image path|Pas dans `C:\Windows\System32`|
|Nom du processus|Faute d'orthographe subtile|
|Nombre d'instances|Trop d'instances|

