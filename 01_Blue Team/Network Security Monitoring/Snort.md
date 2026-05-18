## Introduction to IDS/IPS

### Intrusion Detection System (IDS)

Passif — génère des alertes, ne bloque pas.

|Type|Portée|
|---|---|
|NIDS|Réseau entier|
|HIDS|Endpoint unique|

### Intrusion Prevention System (IPS)

Actif — détecte **et** bloque/termine la connexion.

|Type|Portée|
|---|---|
|NIPS|Réseau entier (signature)|
|NBA (Behaviour-based)|Réseau entier (anomalie) — nécessite une période de baselining|
|WIPS|Réseau sans-fil|
|HIPS|Endpoint unique|

> NBA vs NIPS : même principe, mais NBA apprend le trafic normal avant de détecter les anomalies → plus efficace contre les menaces inconnues, mais risqué si breach pendant le training.

_NBA = **Network Behaviour Analysis**_

### Detection/Prevention Techniques

|Technique|Principe|
|---|---|
|Signature-Based|Patterns de menaces connues|
|Behaviour-Based|Comparaison normal/anormal → détecte les menaces inconnues|
|Policy-Based|Comparaison avec les politiques de sécurité configurées|

### Snort

NIDS/NIPS open-source, rule-based. Maintenu par Martin Roesch + Cisco Talos.

**3 modes :**

|Mode|Fonction|
|---|---|
|Sniffer|Affiche les paquets IP dans la console|
|Packet Logger|Logge tout le trafic IP entrant/sortant|
|NIDS/NIPS|Logge ou drop les paquets selon les règles définies|

---

## First Interaction with Snort

bash

```bash
snort -V                              # Vérifier la version

sudo snort -c /etc/snort/snort.conf -T  # Valider la configuration
```

|Paramètre|Description|
|---|---|
|`-V` / `--version`|Version de l'instance|
|`-c`|Spécifier le fichier de configuration|
|`-T`|Tester/valider la configuration|
|`-q`|Quiet mode — supprime le banner et les infos initiales|

> `snort.conf` = fichier central : règles, plugins, mécanismes de détection, actions par défaut, outputs. Un seul utilisable à la fois au runtime.

---

## Operation Mode 1: Sniffer Mode

|Paramètre|Description|
|---|---|
|`-v`|Verbose — output TCP/IP dans la console|
|`-d`|Affiche le payload des paquets|
|`-e`|Affiche les headers link-layer (MAC, type, len)|
|`-X`|Dump complet en HEX|
|`-i eth0`|Spécifie l'interface à sniffer|

Les paramètres sont combinables :

bash

```bash
sudo snort -v
sudo snort -vd
sudo snort -de
sudo snort -v -d -e
sudo snort -X
sudo snort -v -i eth0
```

> `-d` inclut déjà le niveau `-v`. `-X` affiche tout le paquet en hex, headers compris.

---

## Operation Mode 2: Packet Logger Mode

|Paramètre|Description|
|---|---|
|`-l`|Logger mode — output dans `/var/log/snort` par défaut|
|`-K ASCII`|Log en format ASCII lisible (dossiers par IP)|
|`-r`|Lire un fichier de log existant|
|`-n`|Nombre de paquets à traiter puis stop|

bash

```bash
sudo snort -dev -l .                        # Log binaire (tcpdump) dans le dossier courant
sudo snort -dev -K ASCII -l .               # Log ASCII, classé par IP
sudo snort -r snort.log.1638459842          # Lire un log binaire
sudo snort -dvr logname.log -n 10           # Lire les 10 premiers paquets
```

**Filtres BPF (Berkeley Packet Filters) avec `-r` :**

bash

```bash
sudo snort -r logname.log icmp
sudo snort -r logname.log tcp
sudo snort -r logname.log 'udp and port 53'
sudo snort -r logname.log -X
```

Please use the following resources to understand how the BPF works and its use.

- [https://en.wikipedia.org/wiki/Berkeley_Packet_Filter(opens in new tab)](https://en.wikipedia.org/wiki/Berkeley_Packet_Filter)
- [https://biot.com/capstats/bpf.html(opens in new tab)](https://biot.com/capstats/bpf.html)
- [https://www.tcpdump.org/manpages/tcpdump.1.html](https://www.tcpdump.org/manpages/tcpdump.1.html)

**Format binaire vs ASCII :**

- Binaire (défaut) : un seul fichier `snort.log.XXXXXXXX` — lisible par Snort, tcpdump, Wireshark
- ASCII (`-K ASCII`) : dossiers nommés par IP, fichiers lisibles en texte — **non lisible par Snort `-r`**

> Les logs générés en `sudo` appartiennent à root. Utiliser `sudo chown username fichier` ou `sudo chown username -R dossier` pour changer le propriétaire.

---

## Operation Mode 3: IDS/IPS

### Paramètres principaux

|Paramètre|Description|
|---|---|
|`-c`|Définir le fichier de configuration|
|`-T`|Tester la configuration|
|`-N`|Désactiver le logging|
|`-D`|Background (daemon) mode|
|`-A`|Mode d'alerte|

### Modes d'alerte `-A`

|Mode|Console|Contenu|
|---|---|---|
|`console`|Oui|Alertes fast-style|
|`cmg`|Oui|Headers + payload hex/text|
|`fast`|Non|Message, timestamp, IP src/dst, ports|
|`full`|Non|Toutes les informations possibles (défaut)|
|`none`|Non|Pas d'alerte, log binaire uniquement|

bash

```bash
sudo snort -c /etc/snort/snort.conf -A console
sudo snort -c /etc/snort/snort.conf -A cmg
sudo snort -c /etc/snort/snort.conf -A fast
sudo snort -c /etc/snort/snort.conf -A full
sudo snort -c /etc/snort/snort.conf -A none
sudo snort -c /etc/snort/snort.conf -N            # Pas de log
sudo snort -c /etc/snort/snort.conf -D            # Daemon mode
```

**Sans fichier de config (test de règles uniquement) :**

bash

```bash
sudo snort -c /etc/snort/rules/local.rules -A console
```

### Mode IPS (drop)

Nécessite 2 interfaces minimum + module DAQ (Data Acquisition) afpacket :

bash

```bash
sudo snort -c /etc/snort/snort.conf -q -Q --daq afpacket -i eth0:eth1 -A console
```

> En IPS, les alertes affichent `[Drop]` au lieu de `[**]`.

**Gérer le daemon :**

bash

```bash
ps -ef | grep snort
sudo kill -9 <PID>
```

---

## Operation Mode 4: PCAP Investigation

|Paramètre|Description|
|---|---|
|`-r` / `--pcap-single=`|Lire un seul pcap|
|`--pcap-list=""`|Lire plusieurs pcaps (séparés par des espaces)|
|`--pcap-show`|Affiche le nom du pcap en cours dans la console|

bash

```bash
# Pcap unique
sudo snort -c /etc/snort/snort.conf -q -r icmp-test.pcap -A console -n 10

# Plusieurs pcaps
sudo snort -c /etc/snort/snort.conf -q --pcap-list="icmp-test.pcap http2.pcap" -A console

# Plusieurs pcaps avec distinction par fichier
sudo snort -c /etc/snort/snort.conf -q --pcap-list="icmp-test.pcap http2.pcap" -A console --pcap-show
```

> Sans `--pcap-show`, les alertes de plusieurs pcaps sont mélangées et intraçables. Toujours l'utiliser en analyse multi-fichiers.

---

## Snort Rule Structure

### Structure d'une règle

```
action protocol src_ip src_port direction dst_ip dst_port (options)
```

```
alert tcp 192.168.1.0/24 any -> any 80 (msg:"HTTP Traffic"; sid:100001; rev:1;)
```

![[snort rule structure.png]]

### Actions

|Action|Effet|
|---|---|
|`alert`|Génère une alerte + log|
|`log`|Log uniquement|
|`drop`|Bloque + log|
|`reject`|Bloque + log + termine la session|

### Protocoles

Snort 2 : `ip`, `tcp`, `udp`, `icmp` uniquement. Pour FTP → filtrer `tcp port 21`.

### IP & Ports

```
192.168.1.56          # IP unique
192.168.1.0/24        # Subnet
[192.168.1.0/24, 10.1.1.0/24]  # Plusieurs ranges
!192.168.1.0/24       # Exclusion
any                   # Tout
```

```
any 21        # Port exact
any !21       # Exclusion
any 1:1024    # Range 1-1024
any :1024     # 0 à 1024
any 1025:     # 1025 et +
any [21,23]   # Liste de ports
```

### Direction

```
->   # Source vers destination
<>   # Bidirectionnel
```

> Pas de `<-` dans Snort.

### There are three main rule options in Snort.

- General Rule Options - Fundamental rule options for Snort. 
- Payload Rule Options - Rule options that help to investigate the payload data. These options are helpful to detect specific payload patterns.
- Non-Payload Rule Options - Rule options that focus on non-payload data. These options will help create specific patterns and identify network issues.

### General Rule Options

|Option|Description|
|---|---|
|`msg:`|Message affiché dans l'alerte|
|`sid:`|ID unique de la règle (`>=1000000` pour règles user)|
|`reference:`|Référence externe (ex: `cve,CVE-XXXX`)|
|`rev:`|Numéro de révision de la règle|

### Payload Detection Rule Options

| Option          | Description                                                                   |
| --------------- | ----------------------------------------------------------------------------- |
| `content:`      | Match ASCII ou HEX dans le payload (case-sensitive)                           |
| `nocase;`       | Désactive la sensibilité à la casse                                           |
| `fast_pattern;` | Priorise le contenu pour la recherche initiale (obligatoire si multi-content) |

```
content:"GET";                  # ASCII
content:"|47 45 54|";           # HEX
content:"GET"; nocase;
content:"GET"; fast_pattern; content:"www";
```

### Non-Payload Detection Rule Options

|Option|Exemple|Description|
|---|---|---|
|`id:`|`id:123456;`|Filtre sur l'IP ID field|
|`flags:`|`flags:S;`|Filtre TCP flags (F/S/R/P/A/U)|
|`dsize:`|`dsize:100<>300;`|Taille du payload|
|`sameip;`|`sameip;`|IP src == IP dst|

Filtre TCP flags :
- F - FIN
- S - SYN
- R - RST
- P - PSH
- A - ACK
- U - URG

### Fichier de règles locales

bash

```bash
sudo gedit /etc/snort/rules/local.rules
```

### Rules created for questions

#### Use "task9.pcap". Write a rule to filter IP ID "35369" and run it against the given pcap file. What is the request name of the detected packet? You may use this command: "snort -c local.rules -A full -l . -r task9.pcap"

`alert ICMP any any <> any any (msg: “ID requested”; id:35369; sid: 1000001; rev:1;)`
 
**Answer** : TIMESTAMP REQUEST

#### Create a rule to filter packets with **Syn** flag and run it against the given pcap file. What is the number of detected packets?

`alert TCP any any <> any any (msg: "wesh le sang"; flags:S; sid: 100001; rev:1;)`

**Answer** : 1

#### Write a rule to filter packets with **Push-Ack** flags and run it against the given pcap file. What is the number of detected packets?

`alert TCP any any <> any any (msg: "wesh le sang"; flags:PA; sid: 100001; rev:1;)`

**Answer** : 216

#### Create a rule to filter **UDP** packets with the same source and destination IP and run it against the given pcap file. What is the number of packets that show the same source and destination address?

`alert UDP any any <> any any (msg: "wesh le sang"; sameip; sid: 100001; rev:1;)`

**Answer** : 7


---

## Snort2 Operation Logic: Points to Remember

### Main Components of Snort

|Composant|Rôle|
|---|---|
|Packet Decoder|Collecte et prépare les paquets|
|Pre-processors|Arrange et modifie les paquets pour le moteur de détection|
|Detection Engine|Applique les règles, analyse les paquets|
|Logging and Alerting|Génère les logs et alertes|
|Outputs and Plugins|Intégration syslog/mysql, plugins de gestion des règles|

### Snort Rules

|Type|Accès|Détail|
|---|---|---|
|Community|Gratuit, sans inscription|GPLv2, public|
|Registered|Gratuit, inscription requise|Contient les subscriber rules avec 30j de délai|
|Subscriber|Payant|Ruleset principal, mis à jour 2x/semaine (mar/jeu)|

You can download and read more on the rules [here(opens in new tab)](https://www.snort.org/downloads).

### Fichiers clés

```
/etc/snort/snort.conf    # Configuration principale
/etc/snort/rules/local.rules  # Règles utilisateur
```

> Ne jamais remplacer `snort.conf` — toujours éditer manuellement ou via des outils dédiés.

### snort.conf — Sections importantes

Let's start with an overview of the main configuration file (snort.conf). sudo gedit /etc/snort/snort.conf

**Step 1 — Network variables**

|Variable|Description|Exemple|
|---|---|---|
|`HOME_NET`|Réseau à protéger|`192.168.1.0/24`|
|`EXTERNAL_NET`|Réseau externe|`any` ou `!$HOME_NET`|
|`RULE_PATH`|Chemin des règles|`/etc/snort/rules`|
|`SO_RULE_PATH`|Règles registered/subscriber|`$RULE_PATH/so_rules`|
|`PREPROC_RULE_PATH`|Règles plugin|`$RULE_PATH/plugin_rules`|

**Step 2 — Decoder / IPS mode**

|Tag|Description|Exemple|
|---|---|---|
|`#config daq`|Sélection du module DAQ|`afpacket`|
|`#config daq_mode`|Activation inline (IPS)|`inline`|
|`#config logdir`|Chemin de log par défaut|`/var/logs/snort`|

**DAQ modules**

|Module|Mode|
|---|---|
|`pcap`|Défaut — Sniffer|
|`afpacket`|Inline — IPS|
|`ipq`|Inline Linux (Netfilter)|
|`nfq`|Inline Linux|
|`ipfw`|Inline OpenBSD/FreeBSD|
|`dump`|Test inline/normalisation|

**Step 6** — Configuration des outputs (format logs/alertes).

**Step 7 — Ruleset**

```
include $RULE_PATH/local.rules       # Règles locales (actif par défaut)
#include $RULE_PATH/rulename         # Règles externes (décommenter pour activer)
```

> `#` = commentaire. Décommenter une ligne pour l'activer.

---

## Conclusion

In this room, we covered Snort, its definition, operation, and how to create and use rules to investigate threats.

- Understanding and practising the fundamentals is crucial before creating advanced rules and using additional options.
- Do not create complex rules at once; try adding options step by step to easily notice possible syntax errors or other problems.
- Do not reinvent the wheel; use or modify/enhance it if there is a smooth rule.
- Take a backup of the configuration files before making any changes.
- Never delete a rule that works properly. Comment on it if you don't need it.
- Test newly created rules before migrating them to production.

Now, we invite you to complete a series of Snort challenges, starting from [Snort Challenge - The Basics](https://tryhackme.com/room/snortchallenges1).

cf : C:\Users\\Documents\ressources\snort\SnortCheatsheet.pdf

