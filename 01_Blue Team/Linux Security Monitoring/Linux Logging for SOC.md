## Working With Text Logs

### Log Format

Logs Linux = fichiers texte bruts dans `/var/log/`. Pas d'Event Viewer, pas d'event codes, pas de format strict. Format varie selon la distro.

Format syslog standard :

```
<timestamp> <hostname> <service>[PID]: <message>
```

### Working With Logs

Fichier clé : `/var/log/syslog` — flux agrégé d'événements système.

bash

```bash
cat /var/log/syslog | head
```

### Filtering Logs

bash

```bash
cat /var/log/syslog | grep CRON       # filtrer sur un mot-clé
cat /var/log/syslog | grep -v CRON    # exclure un mot-clé
```

### Discovering Logs

bash

```bash
ls -l /var/log                            # lister les fichiers de logs disponibles
grep -R -E "auth|login|session" /var/log  # chercher un mot-clé dans tous les logs
```

Fichiers notables :

|Fichier|Contenu|
|---|---|
|`auth.log`|Authentifications|
|`syslog`|Événements système agrégés|
|`kern.log`|Kernel|
|`dpkg.log`|Paquets installés|

### Logging Caveats

Format, verbosité et emplacement des logs sont modifiables sous Linux. Les logs peuvent différer ou ne pas exister selon la distro.

---

## Authentication Logs

Fichier : `/var/log/auth.log` (ou `/var/log/secure` sur RHEL) Contient : authentifications, gestion utilisateurs, commandes sudo, cron.

![[authentication_logs.png]]

### Login and Logout Events

bash

```bash
cat /var/log/auth.log | grep -E 'session opened|session closed'
```

|Service dans le log|Contexte|
|---|---|
|`login:session`|Login local clavier|
|`sshd:session`|Login SSH|
|`samba:session`|Login SMB|
|`cron:session`|Exécution cron job|
|`sudo:session`|`sudo su` / élévation|

### SSH-Specific Events

bash

```bash
cat /var/log/auth.log | grep "sshd" | grep -E 'Accepted|Failed'
```

Format : `<Failed|Accepted> <auth-method> for <user> from <ip>`

### Miscellaneous Events

bash

```bash
# Gestion utilisateurs
cat /var/log/auth.log | grep -E '(passwd|useradd|usermod|userdel)\['

# Commandes sudo (détection d'actions malveillantes)
cat /var/log/auth.log | grep -E 'COMMAND='
```

Signaux d'alerte typiques :

- `useradd` / `usermod` → création ou ajout au groupe `sudo`
- `userdel` → suppression de compte (ex: nettoyage de backdoor)
- `COMMAND=` → arrêt d'EDR, modification firewall, `sudo su`

---

## Common Linux Logs

### Generic System Logs

| Fichier                                 | Contenu                    |
| --------------------------------------- | -------------------------- |
| `/var/log/kern.log`                     | Messages kernel            |
| `/var/log/syslog` (ou `messages`)       | Événements système agrégés |
| `/var/log/dpkg.log` / `/var/log/apt`    | Paquets (Debian)           |
| `/var/log/dnf.log` / `/var/log/yum.log` | Paquets (RHEL)             |

Utiles en DFIR, rarement exploités au quotidien en SOC (bruités, peu structurés).

### App-Specific Logs

Exemple Nginx : `/var/log/nginx/access.log` Format : `<ip> - - [<date>] "<method> <path> HTTP/x.x" <status> <size>`

### Bash History

bash

```bash
cat /home/<user>/.bash_history   # commandes des sessions précédentes
history                          # session courante + passées
```

**Limitations — contournements connus :**

bash

```bash
 echo "commande précédée d'un espace → non loguée"

nano legit.sh && ./legit.sh      # commandes dans un script = non tracées

sh                               # /bin/sh ne sauvegarde pas l'historique
```

→ Ne trace pas les commandes non-interactives (cron, services). Peu fiable en SOC.

---

## Runtime Monitoring

### Runtime Monitoring

Par défaut, Linux ne loge pas : création de processus, modifications de fichiers, événements réseau → **runtime events**. 

Solution : outil additionnel (comme Sysmon sous Windows) → ici, **auditd**.

![[runtime_monitoring.png]]

### System Calls

Toute interaction avec l'OS passe par un **syscall** : ouvrir un fichier, créer un processus, accéder au réseau, etc. Linux en compte +300 (ex: `execve` pour exécuter un programme).

https://man7.org/linux/man-pages/man2/syscalls.2.html

Les EDR et outils de logging surveillent les syscalls et les retranscrivent en format lisible. Les syscalls sont quasi impossibles à contourner pour un attaquant → c'est la base du monitoring fiable.

![[system_calls.png]]

---

## Using Auditd

### Audit Daemon

Auditd = solution d'audit intégrée Linux. Règles dans `/etc/audit/rules.d/` → définissent quels syscalls surveiller. Logs : `/var/log/audit/audit.log` (brut) ou `ausearch` (filtré, lisible).

![[auditd_daemon.png]]

bash

```bash
ausearch -i -k <key>    # -i = format lisible, -k = filtrer par clé
```

### Champs clés d'un événement auditd

|Champ|Signification|
|---|---|
|`pid` / `ppid`|PID du processus et de son parent|
|`auid`|Utilisateur de login initial (SSH ou local)|
|`uid`|Utilisateur qui a exécuté la commande (peut différer après `sudo`/`su`)|
|`tty`|Identifiant de session|
|`exe`|Chemin absolu du binaire exécuté|
|`key`|Tag défini dans la règle auditd (pour filtrer)|

Blocs d'un événement : `PROCTITLE` (ligne de commande), `CWD` (répertoire courant), `PATH` (fichier accédé), `SYSCALL` (détails syscall).

### Auditd Alternatives

Tous fonctionnent sur le même principe : **monitoring des syscalls**.

- [Sysmon for Linux(opens in new tab)](https://github.com/microsoft/SysmonForLinux): A perfect choice if you already work with Sysmon and love XML
- [Falco(opens in new tab)](https://falco.org/): A modern, open-source solution, ideal for monitoring containerized systems
- [Osquery(opens in new tab)](https://osquery.io/): An interesting tool that can be broadly used for various security purposes
- [EDRs](https://tryhackme.com/room/introductiontoedrs): Most EDR solutions can track and monitor various Linux runtime events

---

## Résumé — Linux Logging for SOC 

### Logs texte, pas d'Event Viewer

Tout est dans `/var/log/`, lisible avec `cat`, `grep`, `head`. Pas de codes d'événements, format variable selon la distro.

### Fichiers essentiels à connaître

|Fichier|Contenu clé|
|---|---|
|`/var/log/auth.log`|Logins, sudo, gestion users — **le plus utile en SOC**|
|`/var/log/syslog`|Événements système agrégés|
|`/var/log/audit/audit.log`|Runtime events (auditd)|
|`~/.bash_history`|Historique commandes — peu fiable|

### Commandes de base

bash

```bash
grep -E "session opened|session closed" /var/log/auth.log   # logins/logouts
grep -E 'COMMAND=' /var/log/auth.log                        # commandes sudo
grep -R -E "auth|login|session" /var/log                    # découverte
ausearch -i -k <key>                                        # logs auditd filtrés
```

### Ce que Linux ne logue PAS par défaut

Création de processus, modifications de fichiers, événements réseau → nécessite **auditd** (ou Falco, Sysmon, EDR).

### Signaux d'alerte dans auth.log

- `useradd` / `usermod` + groupe `sudo` → création de backdoor
- `COMMAND=` + arrêt EDR / `sudo su` → escalade de privilèges
- `Failed password` en masse → brute force SSH

### Bash History — à retenir

Contournable par : espace en début de commande, script externe, shell alternatif (`sh`, `dash`). Ne pas en faire une source de preuve principale.

### Principe fondamental

Tous les outils de monitoring (auditd, Falco, EDR) reposent sur les **syscalls**. Comprendre les syscalls = comprendre pourquoi un événement est logué ou non.

