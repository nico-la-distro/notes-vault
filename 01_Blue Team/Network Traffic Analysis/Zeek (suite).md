Zeek (ex-Bro) : analyseur de trafic réseau passif, open-source. Usages : NSM (Network Security Monitor), investigation, mesure de performance.

---

## Network Security Monitoring and Zeek

### Introduction to Network Monitoring Approaches

|              | Network Monitoring                          | Network Security Monitoring    |
| ------------ | ------------------------------------------- | ------------------------------ |
| Focus        | Uptime, performance, configuration          | Anomalies, intrusions, menaces |
| Utilisateurs | IT/Network team                             | SOC (tier 1-2-3)               |
| Limites      | Ne couvre pas les menaces internes/zero-day | —                              |

### What is ZEEK?

Zeek : framework passif d'analyse réseau, open-source (fork commercial : Corelight). Produit **50+ logs** en **7 catégories** -> orienté forensics et analyse de données.

### Zeek vs Snort

|                | Zeek                                                        | Snort                                    |
| -------------- | ----------------------------------------------------------- | ---------------------------------------- |
| Type           | NSM + IDS (orienté événements)                              | IDS/IPS (orienté signatures/paquets)     |
| Points forts   | Visibilité profonde, threat hunting, scripting, corrélation | Règles simples, support Cisco/communauté |
| Points faibles | Complexe, analyse manuelle/externe                          | Détection de menaces complexes limitée   |
| Usage          | Investigation, chained events                               | Détection/prévention d'attaques connues  |

### Zeek Architecture

- **Event Engine** : traitement des paquets (src/dst, protocole, session, extraction de fichiers)
- **Policy Script Interpreter** : analyse sémantique et corrélation via scripts Zeek

![[zeek_archi.png]]

### Zeek Frameworks

Logging, Notice, Input, Configuration, Intelligence, Cluster, Broker Communication, Supervisor, GeoLocation, File Analysis, Signature, Summary, NetControl, Packet Analysis, TLS Decryption

You can read more on frameworks [**here**(opens in new tab)](https://docs.zeek.org/en/master/frameworks/index.html).

### Zeek Outputs

Logs générés automatiquement dans le répertoire courant (pcap) ou `/opt/zeek/logs/` (service).

### Working with Zeek

bash

```bash
zeek -v                  # version
sudo su                  # élever les privilèges

zeekctl                  # mode interactif
zeekctl status|start|stop

zeek -C -r sample.pcap   # analyser un pcap (logs dans le répertoire courant)
ls -l                    # vérifier les logs générés
```

|Paramètre|Description|
|---|---|
|`-r`|Lire/traiter un fichier pcap|
|`-C`|Ignorer les erreurs de checksum|
|`-v`|Version|
|`zeekctl`|Module ZeekControl|

> Analyse des logs : `cat`, `cut`, `grep`, `sort`, `uniq`, `zeek-cut`

### Question
#### Investigate the **"sample.pcap"** file. What is the number of generated alert files?

![[zeek_t2q4.png]]

**Answer** : 8

---

## Zeek Logs

Logs en ASCII tabulés, corrélés via **UID** (identifiant unique par session).

### Catégories de logs

| Catégorie            | Description                             | Log Files                                                                                                                                                                                                                                                                                                                       |
| -------------------- | --------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Network              | Logs protocoles réseau                  | conn.log, dce_rpc.log, dhcp.log, dnp3.log, dns.log, ftp.log, http.log, irc.log, kerberos.log, modbus.log, modbus_register_change.log, mysql.log, ntlm.log, ntp.log, radius.log, rdp.log, rfb.log, sip.log, smb_cmd.log, smb_files.log, smb_mapping.log, smtp.log, snmp.log, socks.log, ssh.log, ssl.log, syslog.log, tunnel.log |
| Files                | Logs d'analyse de fichiers              | files.log, ocsp.log, pe.log, x509.log                                                                                                                                                                                                                                                                                           |
| NetControl           | Logs de contrôle réseau et flux         | netcontrol.log, netcontrol_drop.log, netcontrol_shunt.log, netcontrol_catch_release.log, openflow.log                                                                                                                                                                                                                           |
| Detection            | Logs de détection et indicateurs        | intel.log, notice.log, notice_alarm.log, signatures.log, traceroute.log                                                                                                                                                                                                                                                         |
| Network Observations | Logs de flux réseau                     | known_certs.log, known_hosts.log, known_modbus.log, known_services.log, software.log                                                                                                                                                                                                                                            |
| Miscellaneous        | Alertes externes, inputs, échecs        | barnyard2.log, dpd.log, unified2.log, unknown_protocols.log, weird.log, weird_stats.log                                                                                                                                                                                                                                         |
| Zeek Diagnostic      | Messages système, actions, statistiques | broker.log, capture_loss.log, cluster.log, config.log, loaded_scripts.log, packet_filter.log, print.log, prof.log, reporter.log, stats.log, stderr.log, stdout.log                                                                                                                                                              |

### Fréquence de mise à jour

|Fréquence|Log|Contenu|
|---|---|---|
|Daily|known_hosts.log|Hôtes ayant complété un handshake TCP|
|Daily|known_services.log|Services utilisés sur le réseau|
|Daily|known_certs.log|Certificats SSL|
|Daily|software.log|Logiciels détectés|
|Per Session|notice.log|Anomalies détectées par Zeek|
|Per Session|intel.log|Indicateurs malveillants|
|Per Session|signatures.log|Signatures déclenchées|

### Modèle d'investigation

|Overall Info|Protocol Based|Detection|Observation|
|---|---|---|---|
|conn.log|http.log|notice.log|known_hosts.log|
|files.log|dns.log|signatures.log|known_services.log|
|intel.log|ftp.log|pe.log|software.log|
|loaded_scripts.log|ssh.log|traceroute.log|weird.log|

Ordre d'investigation : Overall → Protocol → Detection → Observation.

### Opening a Zeek log with a text editor and buit-in commands

![[zeek_open_zeek_log_text_editor.png]]

### zeek-cut

Extrait des colonnes spécifiques d'un log. Utiliser les noms de **fields** (pas les types).

bash

```bash
cat conn.log | zeek-cut uid proto id.orig_h id.orig_p id.resp_h id.resp_p
```

```
CTMFXm1AcIsSnq2Ric  udp  192.168.121.2   51153  192.168.120.22  53
CLsSsA3HLB2N6uJwW   udp  192.168.121.10  50080  192.168.120.10  514
```

### Questions
#### Investigate the **sample.pcap** file. Investigate the **dhcp.log** file. What is the available hostname?

![[zeek_t3q2.png]]

**Answer** : Microknoppix

#### Investigate the **dns****.log** file. What is the number of unique DNS queries?

```bash
cat dns.log
```

checking for the correct field

![[zeek_t3q3.png]]

```bash
cat dns.log | zeek-cut query | uniq
```

![[zeek_t3q3_2.png]]

**Answer** : 2

#### Investigate the **conn.log** file. What is the longest connection duration?

check for the right field

```bash
cat conn.log | grep '#fields'
```

![[zeek_t3q4.png]]

filter with zeek-cut

```bash
cat conn.log | zeek-cut duration | sort -n -r | head -1
```

![[zeek_t3q4_2.png]]

**Answer** : 332.319364

---

## CLI Kung-Fu Recall: Processing Zeek Logs

Les GUI sont limitées sur de gros volumes de données. La maîtrise du CLI (+ BPF + regex) est indispensable pour extraire et manipuler les données réseau.

_BPF **Berkeley Packet Filter** : syntaxe de filtrage de paquets réseau utilisée dans des outils comme `tcpdump`, Wireshark, ou Zeek._

### Basics

bash

```bash
history          # historique des commandes
!10              # exécuter la commande n°10
!!               # réexécuter la dernière commande
```

### Read File

bash

```bash
cat sample.txt
head sample.txt  # 10 premières lignes
tail sample.txt  # 10 dernières lignes
```

### Find & Filter

bash

```bash
cat test.txt | cut -f 1        # couper le 1er champ (tabulation)
cat test.txt | cut -c1         # couper la 1ère colonne (caractère)
cat test.txt | grep 'keyword'  # filtrer par mot-clé
cat test.txt | sort            # tri alphabétique
cat test.txt | sort -n         # tri numérique
cat test.txt | uniq            # supprimer les doublons
cat test.txt | wc -l           # compter les lignes
cat test.txt | nl              # afficher avec numéros de lignes
```

### Advanced

bash

```bash
cat test.txt | sed -n '11p'          # afficher la ligne 11
cat test.txt | sed -n '10,15p'       # afficher lignes 10 à 15
cat test.txt | awk 'NR < 11 {print $0}'   # lignes avant 11
cat test.txt | awk 'NR == 11 {print $0}'  # ligne 11
```

### Special

bash

```bash
cat signatures.log | zeek-cut uid src_addr dst_addr
```

### Combinaisons utiles

|Commande|Usage|
|---|---|
|`sort \| uniq`|Supprimer les doublons|
|`sort \| uniq -c`|Supprimer doublons + compter occurrences|
|`sort -nr`|Tri numérique décroissant|
|`rev`|Inverser les caractères d'une chaîne|
|`cut -f 1`|Extraire le champ 1|
|`cut -d '.' -f 1-2`|Split sur `.`, garder les 2 premiers champs|
|`grep -v 'test'`|Exclure les lignes contenant "test"|
|`grep -v -e 'test1' -e 'test2'`|Exclure plusieurs patterns|
|`file`|Infos sur un fichier|
|`grep -rin Testvalue1 * \| column -t \| less -S`|Recherche récursive, colonnes alignées, vue paginée|

---

## Zeek Signatures

Signatures = pattern matching bas niveau (similaire à Snort), mais pas le point de détection principal -> Zeek s'appuie surtout sur son scripting.

### Structure d'une signature

```
signature <id> {
    <conditions>
    <action>
}
```

| **Signature id** | **Unique** signature name.                                                                                                                                                                      |
| ---------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Conditions**   | **Header:** Filtering the packet headers for specific source and destination addresses, protocol and port numbers.<br><br>**Content:** Filtering the packet payload for specific value/pattern. |
| **Action**       | **Default action:** Create the "signatures.log" file in case of a signature match.<br><br>**Additional action:** Trigger a Zeek script.                                                         |

|Bloc|Champ|Description|
|---|---|---|
|**Header**|`src-ip`, `dst-ip`|IP source/destination|
||`src-port`, `dst-port`|Ports|
||`ip-proto`|TCP, UDP, ICMP, ICMP6, IP, IP6|
|**Content**|`payload`|Payload brut|
||`http-request`|Requête HTTP décodée|
||`http-request-header/body`|Headers/body client|
||`http-reply-header/body`|Headers/body serveur|
||`ftp`|Input CLI FTP|
|**Context**|`same-ip`|Src = Dst|
|**Action**|`event`|Message d'alerte → `signatures.log` + `notice.log`|

Opérateurs : `==`, `!=`, `<`, `<=`, `>`, `>=` -> acceptent string, numérique, regex.

### Lancer Zeek avec une signature

bash

```bash
zeek -C -r sample.pcap -s sample.sig
```

Extension des fichiers de signature : `.sig`

### Analyse des logs

bash

```bash
cat signatures.log | zeek-cut src_addr dst_addr sig_id event_msg sub_msg
cat notice.log | zeek-cut id.orig_h id.resp_h msg sub
```

- `event_msg` / `msg` : message de l'alerte
- `sub_msg` / `sub` : banner applicatif (ex. requête HTTP brute)

### Exemple -> HTTP cleartext password

```
signature http-password { 
	ip-proto == tcp 
	dst-port == 80 
	payload /.*password.*/ 
	event "Cleartext Password Found!" 
}
```

Signature : détecter `password` dans le payload HTTP via regex `.*password.*`.

### Exemple -> FTP brute-force

Deux signatures combinées dans un même `.sig` :

- Règle 1 : détecter `USER admin` dans le contenu FTP
- Règle 2 : détecter la réponse `530` (login failed) → détection brute-force globale

```
signature ftp-brute { 
	ip-proto == tcp 
	payload /.*530.*Login.*incorrect.*/ 
	event "FTP Brute-force Attempt"
}
```

```
signature ftp-username { 
	ip-proto == tcp 
	ftp /.*USER.*/ 
	event "FTP Username Input Found!" 
} 

signature ftp-brute { 
	ip-proto == tcp 
	payload /.*530.*Login.*incorrect.*/ 
	event "FTP Brute-force Attempt!"
}
```

bash

```bash
zeek -C -r ftp.pcap -s ftp-admin.sig

cat notice.log | zeek-cut uid id.orig_h id.resp_h msg sub | sort -r | nl | uniq | sed -n '1001,1004p'
```

> Principe : logger logiquement, pas tout logger. Préférer des règles globales (code 530) aux règles trop spécifiques (nom de compte).

### Snort Rules dans Zeek

Le script `snort2bro` n'est plus supporté depuis le rebranding Bro → Zeek.

### Questions
#### Investigate the **http.pcap** file. Create the  HTTP signature shown in the task and investigate the pcap. What is the source IP of the first event?

first make the signature file

```bash
nano http-password.sig
```

```bash                         
signature http-password {
    ip-proto == tcp
    dst-port == 80
    payload /.*password.*/
    event "HERE A CLEAR PSWD!!!!!!!!!!"
}

```

then make the log files

```bash
zeek -C -r http.pcap -s http-password.sig
```

![[zeek_t5q2.png]]

check the fields of the signatures.log

```bash
cat signatures.log | grep '#fields'
```

![[zeek_t5q2_2.png]]

then use zeek-cut to investigate the log signatures with the appropriate fields

```bash
cat signatures.log | zeek-cut src_addr dst_addr event_msg sig_id
```

![[zeek_t5q2_3.png]]

**Answer** : 10.10.57.178

#### What is the source port of the second event?

just add source port (src_port) in the previous command

**Answer** : 38712

#### Investigate the conn.log. What is the total number of the sent and received packets from source port 38706?

check for the fileds

```bash
cat conn.log | grep '#fields'
```

```bash
cat conn.log | zeek-cut orig_pkts resp_pkts id.orig_p
```

![[zeek_t5q4.png]]

**Answer** : 20

#### Create the global rule shown in the task and investigate the **ftp.pcap** file. Investigate the **notice.log.** What is the number of unique events?

make the signature file

```bash
nano ftp-bruteforce.sig
```

```bash
signature ftp-username {
    ip-proto == tcp
    ftp /.*USER.*/
    event "FTP Username Input Found!"
}

signature ftp-brute {
    ip-proto == tcp
    payload /.*530.*Login.*incorrect.*/
    event "FTP Brute-force Attempt!"
}
```

make the log files

```bash
zeek -C -r ftp.pcap -s ftp-bruteforce.sig
```

check for the available fields

```bash
cat notice.log | grep '#fields'
```

list the sessions (uid) then sort it, make uniq events, count the lines and just output the last line

```bash
cat notice.log | zeek-cut uid | sort | uniq | nl | tail -1
```

![[zeek_t5q5.png]]

**Answer** : 1413

#### What is the number of **ftp-brute** signature matches?

```bash
cat notice.log | zeek-cut id.orig_p msg | sort | uniq -c
```

![[zeek_t5q6.png]]

**Answer** : 1410

---

## Zeek Scripts | Fundamentals

Zeek dispose d'un langage de scripting événementiel. Les scripts peuvent aussi servir de **policy scripts** (application de politiques).

### Chemins importants

| Chemin                                 | Usage                                     |
| -------------------------------------- | ----------------------------------------- |
| `/opt/zeek/share/zeek/base`            | Scripts par défaut -> **ne pas modifier** |
| `/opt/zeek/share/zeek/site`            | Scripts utilisateur/modifiés              |
| `/opt/zeek/share/zeek/policy`          | Policy scripts                            |
| `/opt/zeek/share/zeek/site/local.zeek` | Fichier de configuration                  |
| `/opt/zeek/share/zeek/base/bif`        | Built-In Functions                        |
| `/opt/zeek/share/zeek/base/protocols`  | Protocoles supportés                      |

Extension des scripts : `.zeek`

### Chargement d'un script

bash

```bash
# Utilisation ponctuelle (pcap)
zeek -C -r sample.pcap mon-script.zeek

# Chargement permanent (live)
# Ajouter dans local.zeek :
@load /script/path
# ou
@load script-name
```

### Zeek est orienté événements, pas paquets

Les scripts réagissent à des **événements** (connexion, message DHCP, requête HTTP...).

zeek

```zeek
event dhcp_message (c: connection, is_orig: bool, msg: DHCP::Msg, options: DHCP::Options)
{
    print options$host_name;
}
```

bash

```bash
zeek -C -r smallFlows.pcap dhcp-hostname.zeek
# student01-PC
# vinlap01
```

> Avantage vs tcpdump/tshark : extraction propre, sans pipeline non contrôlé, facilement réutilisable.

### Questions
#### Investigate the **smallFlows.pcap** file. Investigate the **dhcp.log** file. What is the domain value of the **"vinlap01"** host?

generate the log files

```bash
zeek -C -r smallFlows.pcap dhcp-hostname.zeek
```

check for the fields then type this command

```bash
cat dhcp.log | zeek-cut domain
```

**Answer** : astaro_vineyard

#### Investigate the **big**Flows.pcap file. Investigate the **dhcp.log** file. What is the number of identified unique hostnames?

```bash
zeek -C -r bigFlows.pcap dhcp-hostname.zeek 
```

```bash
cat dhcp.log | grep '#fields'
```

```bash
cat dhcp.log | zeek-cut host_name | sort -nr | uniq | nl
```

![[zeek_t6q3.png]]

**Answer** : 17

#### Investigate the dhcp.log file. What is the identified domain value?

```bash
cat dhcp.log | zeek-cut domain | sort | uniq -c  
```

**Answer** : jaalam.net

---

## Zeek Scripts | Scripts and Signatures

### Scripts 101 | Write Basic Scripts

Les scripts contiennent : opérateurs, types, attributs, déclarations, statements, directives.

**Événements de cycle de vie** (sans paramètres) :

zeek

```zeek
event zeek_init()
{
    print ("Started Zeek!");
}
event zeek_done()
{
    print ("Stopped Zeek!");
}
# zeek_init : exécuté au démarrage
# zeek_done : exécuté à l'arrêt
```

bash

```bash
zeek -C -r sample.pcap 101.zeek
# Started Zeek!
# Stopped Zeek!
```

**Afficher toutes les données brutes d'une connexion** (`new_connection` = déclenché automatiquement pour chaque nouvelle connexion) :

zeek

```zeek
event new_connection(c: connection)
{
    print c;
}
```

bash

```bash
zeek -C -r sample.pcap 102.zeek
# [id=[orig_h=192.168.121.40, orig_p=123/udp, resp_h=212.227.54.68, ...], ...]
```

La sortie brute contient tous les champs identiques à ceux des fichiers de logs. Pour filtrer, on utilise le tag principal (`c`), la valeur `id` et le nom du champ. Accès via `$` : `c$id$orig_h`.

**Extraire des champs spécifiques** :

zeek

```zeek
event new_connection(c: connection)
{
    print ("###########################################################");
    print ("");
    print ("New Connection Found!");
    print ("");
    print fmt ("Source Host: %s # %s --->", c$id$orig_h, c$id$orig_p);
    print fmt ("Destination Host: resp: %s # %s <---", c$id$resp_h, c$id$resp_p);
    print ("");
}
# %s : string output
# c$id : référence au champ identifiant de la connexion
```

bash

```bash
zeek -C -r sample.pcap 103.zeek
# New Connection Found!
# Source Host: 192.168.121.2 # 58304/udp --->
# Destination Host: resp: 192.168.120.22 # 53/udp <---
```

> Les logs sont toujours générés dans le répertoire courant, indépendamment des scripts.

### Scripts 201 | Use Scripts and Signatures Together

Scripts et signatures utilisables ensemble. Les scripts peuvent référencer des signatures et d'autres scripts.

**Événement `signature_match`** : déclenché sur chaque hit de signature.

zeek

```zeek
event signature_match (state: signature_state, msg: string, data: string)
{
    if (state$sig_id == "ftp-admin")
    {
        print ("Signature hit! --> #FTP-Admin");
    }
}
```

Signature associée :

```
signature ftp-admin {
    ip-proto == tcp
    ftp /.*USER.*admin.*/
    event "FTP Username Input Found!"
}
```

bash

```bash
zeek -C -r ftp.pcap -s ftp-admin.sig 201.zeek
# Signature hit! --> #FTP-Admin
# Signature hit! --> #FTP-Admin
```

### Scripts 202 | Load Local Scripts

**Charger tous les scripts locaux** définis dans `local.zeek` :

bash

```bash
zeek -C -r ftp.pcap local
```

Logs supplémentaires générés : `loaded_scripts.log`, `capture_loss.log`, `notice.log`, `stats.log`. 465 scripts chargés dans l'exemple. Zeek ne génère pas de log pour les scripts sans hit.

**Charger un script spécifique** :

bash

```bash
zeek -C -r ftp.pcap /opt/zeek/share/zeek/policy/protocols/ftp/detect-bruteforcing.zeek

cat notice.log | zeek-cut ts note msg
# 1024380732.223481  FTP::Bruteforcing  10.234.125.254 had 20 failed logins on 1 FTP server in 0m1s
```

> Le script built-in est plus complet que les règles manuelles : une seule ligne de résumé avec comptage des échecs.

### Questions
#### Investigate the sample.pcap file with **103.zeek script**. Investigate the **terminal output**. What is the number of the detected new connections?

investigate de pcap with the script 

```bash
zeek -C -r sample.pcap 103.zeek
```

the output is to verbose so i use grep and uniq

```bash
zeek -C -r sample.pcap 103.zeek | grep 'New Connection Found!' | uniq -c
```

**Answer** : 87

#### Investigate the ftp.pcap file with **ftp-admin.sig** signature and  **201.zeek** script. Investigate the signatures.log file. What **i**s the number of signature hits?

```bash
zeek -C -r ftp.pcap 201.zeek -s ftp-admin.sig | uniq -c
   
   #1401 Signature hit! --> #FTP-Admin 

```

**Answer** : 1401

#### Investigate the signatures.log file. What is the total number of "administrator" username detections?

```bash
cat signatures.log | zeek-cut sub_msg | sort | uniq -c
    #670 USER admin
    #731 USER administrator

```

**Answer** : 731

#### Investigate the **ftp.pcap** file with **all local scripts**, and investigate the loaded_scripts.log file. What is the total number of loaded scripts?

```bash
zeek -C -r ftp.pcap local
```

search for fields as always then type this command

```bash
cat loaded_scripts.log | zeek-cut name | sort | uniq -c | nl
```

**Answer** : 731

#### Investigate the ftp-brute.pcap file with "/opt/zeek/share/zeek/policy/protocols/ftp/detect-bruteforcing.zeek" script. Investigate the notice.log file. What is the total number of brute-force detections?

```bash
zeek -C -r ftp-brute.pcap /opt/zeek/share/zeek/policy/protocols/ftp/detect-bruteforcing.zeek
```

```bash
cat notice.log | zeek-cut msg | sort | uniq -c
      #1 10.234.125.254 had 20 failed logins on 1 FTP server in 0m1s
      #1 192.168.56.1 had 20 failed logins on 1 FTP server in 0m37s

```

**Answer** : 2

---

## Zeek Scripts | Frameworks

### Scripts 203 | Load Frameworks

Chargement d'un framework dans un script :

zeek

```zeek
@load /opt/zeek/share/zeek/policy/frameworks/framework-name
```

### File Framework | Hashes

Calcul MD5, SHA1, SHA256 de tous les fichiers détectés :

bash

```bash
# Via script custom
cat hash-demo.zeek
# @load /opt/zeek/share/zeek/policy/frameworks/files/hash-all-files.zeek

# Exécution (deux formes équivalentes)
zeek -C -r case1.pcap hash-demo.zeek
zeek -C -r case1.pcap /opt/zeek/share/zeek/policy/frameworks/files/hash-all-files.zeek

cat files.log | zeek-cut md5 sha1 sha256
```

Script interne de `hash-all-files.zeek` :

zeek

```zeek
@load base/files/hash
event file_new(f: fa_file)
{
    Files::add_analyzer(f, Files::ANALYZER_MD5);
    Files::add_analyzer(f, Files::ANALYZER_SHA1);
    Files::add_analyzer(f, Files::ANALYZER_SHA256);
}
```

### File Framework | Extract Files

Extraire tous les fichiers transférés :

bash

```bash
zeek -C -r case1.pcap /opt/zeek/share/zeek/policy/frameworks/files/extract-all-files.zeek
```

Les fichiers sont extraits dans le dossier `extract_files/` créé automatiquement.

Format du nom extrait : `extract-<ts>-<protocole>-<conn_uid>`

bash

```bash
ls extract_files | nl
file * | nl          # identifier le type réel des fichiers extraits
```

Corrélation via `files.log` :

bash

```bash
cat files.log | zeek-cut fuid conn_uids tx_hosts rx_hosts mime_type extracted | nl
```

Recherche d'un `conn_uid` dans tous les logs :

bash

```bash
grep -rin CZruIO2cqspVhLuAO9 * | column -t | nl | less -S
```

> Un même `conn_uid` relie `conn.log`, `files.log` et `http.log` → permet de corréler un fichier suspect avec sa connexion HTTP.

### Notice Framework | Intelligence

Le framework Intelligence corrèle le trafic réseau avec un fichier de threat intel.

Emplacement du fichier intel : `/opt/zeek/intel/zeek_intel.txt`

Contraintes :

- Fichier **tab-delimited** obligatoire
- Ajout de lignes : pas de redéploiement nécessaire
- Suppression de lignes : redéploiement requis

Format du fichier intel :

```
#fields  indicator       indicator_type   meta.source          meta.desc
smart-fax.com            Intel::DOMAIN    zeek-intel-test      Zeek-Intelligence-Framework-Test
```

Script d'activation :

zeek

```zeek
@load policy/frameworks/intel/seen
@load policy/frameworks/intel/do_notice
redef Intel::read_files += { "/opt/zeek/intel/zeek_intel.txt" };
```

bash

```bash
zeek -C -r case1.pcap intelligence-demo.zeek

cat intel.log | zeek-cut uid id.orig_h id.resp_h seen.indicator matched
# CZ1jLe2nHENdGQX377  10.6.27.102  10.6.27.1      smart-fax.com  Intel::DOMAIN
# C044Ot1OxBt8qCk7f2  10.6.27.102  107.180.50.162  smart-fax.com  Intel::DOMAIN
```

> Un match génère `intel.log` avec les détails de la connexion et l'indicateur correspondant.

### Questions
#### Investigate the **case1**.pcap file with **intelligence-demo.zeek** script. Investigate the **intel.log** file. Look at the second finding, where was the intel info found?

```bash
zeek -C -r case1.pcap intelligence-demo.zeek
```

```bash
cat intel.log | zeek-cut seen.where
	#DNS::IN_REQUEST
	#HTTP::IN_HOST_HEADER
```

**Answer** : IN_HOST_HEADER

#### Investigate the **http.log** file. What is the name of the downloaded .exe file?

```bash
cat http.log | zeek-cut uri                    
	#/ncsi.txt
	#/Documents/Invoice&MSO-Request.doc
	#/knr.exe
```

**Answer** : knr.exe

#### Investigate the case1.pcap file with **hash-demo.zeek** script. Investigate the files.log file. What is the MD5 hash of the downloaded .exe file?

```bash
cat files.log | zeek-cut md5 tx_hosts
#cd5a4d3fdd5bffc16bf959ef75cf37bc	23.63.254.163
#b5243ec1df7d1d5304189e7db2744128	107.180.50.162
#cc28e40b46237ab6d5282199ef78c464	107.180.50.162
```

```bash
cat http.log | zeek-cut uri id.resp_h
#/ncsi.txt	23.63.254.163
#/Documents/Invoice&MSO-Request.doc	107.180.50.162
#/knr.exe	107.180.50.162
```

**Answer** : cc28e40b46237ab6d5282199ef78c464

#### Investigate the case1.pcap file with **file-extract-demo.zeek** script. Investigate the "extract_files" folder. Review the contents of the text file. What is written in the file?

```bash
ls -l extract_files/

extract-1561667874.743959-HTTP-Fpgan59p6uvNzLFja
extract-1561667889.703239-HTTP-FB5o2Hcauv7vpQ8y3
extract-1561667899.060086-HTTP-FOghls3WpIjKpvXaEl
```

```bash
cat http.log | zeek-cut uri resp_fuids
/ncsi.txt	Fpgan59p6uvNzLFja
/Documents/Invoice&MSO-Request.doc	FB5o2Hcauv7vpQ8y3
/knr.exe	FOghls3WpIjKpvXaEl
```

```bash
cat extract_files/extract-1561667874.743959-HTTP-Fpgan59p6uvNzLFja 
Microsoft NCSI

Microsoft NCSI
```

**Answer** : Microsoft NCSI

---

## Zeek Scripts | Packages

### Scripts 204 | Package Manager

Outil : `zkg` (nécessite les privilèges root)

|Commande|Description|
|---|---|
|`zkg install zeek/auteur/package`|Installer via chemin|
|`zkg install https://github.com/...`|Installer via URL git|
|`zkg list`|Lister les packages installés|
|`zkg remove`|Supprimer un package|
|`zkg refresh`|Vérifier les mises à jour|
|`zkg upgrade`|Mettre à jour les packages|

**3 façons de charger un package** :

bash

```bash
# 1. Via script avec @load
cat sniff-demo.zeek
# @load /opt/zeek/share/zeek/site/zeek-sniffpass
zeek -Cr http.pcap sniff-demo.zeek

# 2. Via chemin direct
zeek -Cr http.pcap /opt/zeek/share/zeek/site/zeek-sniffpass

# 3. Via nom du package (zkg uniquement)
zeek -Cr http.pcap zeek-sniffpass
```

### Packages | Cleartext Submission of Password

Package : `zeek/cybera/zeek-sniffpass` — détecte les mots de passe en clair dans les HTTP POST.

bash

```bash
zkg install zeek/cybera/zeek-sniffpass

zeek -Cr http.pcap zeek-sniffpass
cat notice.log | zeek-cut id.orig_h id.resp_h proto note msg
# SNIFFPASS::HTTP_POST_Password_Seen  Password found for user BroZeek
# SNIFFPASS::HTTP_POST_Password_Seen  Password found for user ZeekBro
```

### Packages | Geolocation Data

Package : `geoip-conn` — ajoute des données de géolocalisation dans `conn.log` (base MaxMind GeoLite2-City).

bash

```bash
zeek -Cr case1.pcap geoip-conn

cat conn.log | zeek-cut uid id.orig_h id.resp_h geo.orig.country_code geo.orig.region geo.orig.city geo.orig.latitude geo.orig.longitude geo.resp.country_code geo.resp.region geo.resp.city
# Cbk46G2zXi2i73FOU6  10.6.27.102  23.63.254.163  -  -  -  -  -  US  CA  Los Angeles
```

> Géolocalisation disponible uniquement pour les IP présentes dans la base de données interne.

### Questions
#### Investigate the http.pcap file with the **zeek-sniffpass** module. Investigate the notice.log file. Which username has more module hits?

```bash
zeek -Cr http.pcap zeek-sniffpass
```

```bash
cat notice.log | zeek-cut msg | uniq -c
      3 Password found for user BroZeek
      2 Password found for user ZeekBro

```

**Answer** : BroZeek

#### Investigate the case2.pcap file with **geoip-conn** module. Investigate the conn.log file. What is the name of the identified City?

```bash
zeek -Cr case2.pcap geoip-conn
```

```bash
cat conn.log | zeek-cut geo.resp.city |sort| uniq -c
     20 -
     12 Chicago
```

**Answer** : Chicago

#### Which IP address is associated with the identified City?

```bash
cat conn.log | zeek-cut geo.resp.city id.resp_h |sort| uniq -c
[...]
12 Chicago	23.77.86.54
```

**Answer** : 23.77.86.54

#### Investigate the case2.pcap file with **sumstats-counttable.zeek** script. How many types of status codes are there in the given traffic capture?

```bash
zeek -Cr case2.pcap sumstats-counttable.zeek 
Host: 116.203.71.114
status code: 404, count: 6
status code: 301, count: 4
status code: 200, count: 26
status code: 302, count: 4
Host: 23.77.86.54
status code: 301, count: 4
```

**Answer** : 4

---

## Zeek — Points clés

### C'est quoi

Analyseur de trafic réseau passif. Produit 50+ logs structurés en 7 catégories. Orienté **événements**, pas paquets. Plus puissant que Snort pour l'investigation, moins simple pour la détection basique.

### Commandes essentielles

bash

```bash
zeek -v                            # version
zeek -C -r sample.pcap             # analyser un pcap
zeek -C -r sample.pcap -s sig.sig  # avec signature
zeek -C -r sample.pcap script.zeek # avec script
zeek -C -r sample.pcap local       # tous les scripts locaux
zeekctl start|stop|status          # mode service (live)
```

### Logs prioritaires

`conn.log` → vue globale des connexions `notice.log` → anomalies et alertes `intel.log` → hits threat intel `signatures.log` → hits de signatures `files.log` → fichiers transférés `http/dns/ftp/ssh.log` → investigation protocolaire

### Signatures `.sig`

Pattern matching bas niveau. Structure : `id + conditions (header/content) + action (event)`. Ne sont pas le point de détection principal — complétées par les scripts.

### Scripts `.zeek`

Langage événementiel. Accès aux champs via `$` : `c$id$orig_h`. Événements clés : `zeek_init`, `zeek_done`, `new_connection`, `signature_match`. Scripts + signatures combinables dans une même commande.

### Frameworks utiles

|Framework|Usage|
|---|---|
|`files/hash-all-files`|MD5/SHA1/SHA256 des fichiers|
|`files/extract-all-files`|Extraction des fichiers → `extract_files/`|
|`intel/seen` + `intel/do_notice`|Threat intel → `intel.log`|

### Packages `zkg`

`zeek-sniffpass` → mots de passe en clair (HTTP POST) `geoip-conn` → géolocalisation des IP dans `conn.log`

### Workflow d'investigation

1. `conn.log` + `files.log` → vue globale
2. Log protocolaire ciblé (`http`, `dns`, `ftp`...)
3. `notice.log` + `signatures.log` → détection
4. `known_hosts/services.log` + `weird.log` → observation
5. Corrélation par **UID** entre les logs

### Outil zeek-cut

bash

```bash
cat conn.log | zeek-cut uid id.orig_h id.resp_h proto
```

Extraire uniquement les champs utiles. Utiliser les noms de **fields**, pas les types.

---
## SUITE

Now, we invite you to complete the Zeek Exercise room: [**ZeekExercises**](https://tryhackme.com/room/zeekbroexercises)