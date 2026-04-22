## Nmap Scans

- Identifier les **patterns réseau générés par Nmap**
- Types principaux :
    - TCP Connect (`-sT`)
    - SYN Scan (`-sS`)
    - UDP Scan (`-sU`)

### 🧩 TCP Flags (Wireshark)

| Flag    | Description     | Filter Wireshark       |
| ------- | --------------- | ---------------------- |
| SYN     | Init connexion  | `tcp.flags.syn == 1`   |
| ACK     | Ack réception   | `tcp.flags.ack == 1`   |
| SYN+ACK | Réponse serveur | `tcp.flags == 18`      |
| RST     | Reset connexion | `tcp.flags.reset == 1` |
| RST+ACK | Refus connexion | `tcp.flags == 20`      |
| FIN     | Fermeture       | `tcp.flags.fin == 1`   |

👉 Filtre global :

tcp OR udp

### 🔗 TCP Connect Scan (`-sT`)

⚙️ Caractéristiques

- Utilise **3-way handshake complet**
- Accessible sans privilèges (non-root)
- **Window size > 1024**

🔍 Comportement

|État port|Séquence|
|---|---|
|Open|SYN → SYN/ACK → ACK|
|Closed|SYN → RST/ACK|

![[wireshark traffic analysis tcp connect scans open&closed tcp port.png]]

🎯 Filtre détection

tcp.flags.syn==1 and tcp.flags.ack==0 and tcp.window_size > 1024

![[wireshark traffic analysis show tcp connect scan patterns.png]]

### ⚡ SYN Scan (`-sS`)

⚙️ Caractéristiques

- **Pas de handshake complet** (stealth)
- Nécessite privilèges (root)
- **Window size ≤ 1024**

🔍 Comportement

|État port|Séquence|
|---|---|
|Open|SYN → SYN/ACK → RST|
|Closed|SYN → RST/ACK|

![[wireshark traffic analysis syn scans open&closed tcp port.png]]

🎯 Filtre détection

tcp.flags.syn==1 and tcp.flags.ack==0 and tcp.window_size <= 1024

![[wireshark traffic analysis tcp syn scan patterns.png]]

### 📡 UDP Scan (`-sU`)

⚙️ Caractéristiques

- **Pas de handshake**
- Pas de réponse = port possiblement open
- Port fermé → **ICMP error**

🔍 Comportement

|État port|Réponse|
|---|---|
|Open|Pas de réponse|
|Closed|ICMP Type 3 Code 3|

![[wireshark traffic analysis udp scan closed&open port.png]]

👉 ICMP encapsule la requête originale

The ICMP error message uses the original request as encapsulated data to show the source/reason of the packet. Once you expand the ICMP section in the packet details pane, you will see the encapsulated data and the original request, as shown in the below image.

![[wireshark traffic analysis udp scan error msg icmp encapsulated original request.png]]

🎯 Filtre détection

icmp.type==3 and icmp.code==3

![[wireshark traffic analysis show udp scan patterns.png]]

### 🧪 Astuce Analyse

1. Utiliser filtres génériques → repérer anomalies
2. Zoom sur flux spécifique
3. Vérifier patterns (flags + taille fenêtre + réponses)

### ⚡ Points clés à retenir

- **Window size = indicateur clé** (Connect vs SYN)
- **SYN scan = furtif**
- **UDP = silence ou ICMP**
- Toujours analyser **flags + séquence + contexte**

---

## ARP Poisoning / Spoofing (AKA Man In The Middle Attack)

- Manipuler la table **IP ↔ MAC**
- Intercepter le trafic (**Man In The Middle**)
- Cible principale : **gateway (routeur)**

### ⚙️ ARP — Rappels

|Caractéristique|Détail|
|---|---|
|Portée|Réseau local uniquement|
|Rôle|Résolution IP → MAC|
|Sécurité|❌ Aucune authentification|
|Routable|❌ Non|
|Patterns|Request / Reply / Gratuitous|

👉 Vulnérabilité clé : **aucune vérification → spoof facile**

### 🔍 Filtres Wireshark utiles

| Objectif     | Filtre                                                   |
| ------------ | -------------------------------------------------------- |
| ARP global   | `arp`                                                    |
| ARP request  | `arp.opcode == 1`                                        |
| ARP reply    | `arp.opcode == 2`                                        |
| Scan ARP     | `arp.dst.hw_mac==00:00:00:00:00:00`                      |
| Duplicate IP | `arp.duplicate-address-detected`                         |
| Flooding     | `((arp)&&(arp.opcode==1)) && (arp.src.hw_mac == target)` |

**ARP Request**

![[wireshark traffic analysis arp request.png]]

**ARP Response**

![[wireshark traffic analysis arp response.png]]

### 🚨 Indicateurs d’attaque

#### 1. Conflit IP (Spoofing)

|Situation|Interprétation|
|---|---|
|2 MAC pour 1 IP|ARP spoofing|
|IP critique (ex: gateway)|Très suspect|

![[wireshark traffic analysis conflit ip.png]]

| Notes                         | Detection Notes                                                     | Findings                                            |
| ----------------------------- | ------------------------------------------------------------------- | --------------------------------------------------- |
| Possible IP address match     | 1 IP address announced from a MAC address                           | MAC: 00:0c:29:e2:18:b4 <br>IP: 192.168.1.25         |
| Possible ARP spoofing attempt | 2 MAC addresses claimed the same IP (192.168.1.1)→ possible gateway | MAC1: 50:78:b3:f3:cd:f4 <br>MAC2: 00:0c:29:e2:18:b4 |
| Possible ARP flooding attempt | Same MAC claims a new/different IP                                  | MAC: 00:0c:29:e2:18:b4<br>IP: 192.168.1.1           |

#### 2. ARP Flooding

|Pattern|Interprétation|
|---|---|
|Beaucoup de requêtes ARP|Scan ou attaque|
|Range d’IP ciblé|Reconnaissance réseau|

![[wireshark traffic analysis flooding.png]]

| Notes                         | Detection Notes                                                     | Findings                                           |
| ----------------------------- | ------------------------------------------------------------------- | -------------------------------------------------- |
| Possible IP address match     | 1 IP address announced from a MAC address                           | MAC: 00:0c:29:e2:18:b4<br>IP: 192.168.1.25         |
| Possible ARP spoofing attempt | 2 MAC addresses claimed the same IP (192.168.1.1)→ possible gateway | MAC1: 50:78:b3:f3:cd:f4<br>MAC2: 00:0c:29:e2:18:b4 |
| Possible ARP spoofing attempt | Same MAC claims a different/new IP                                  | MAC: 00:0c:29:e2:18:b4<br>IP: 192.168.1.1          |
| Possible ARP flooding attempt | Multiple ARP requests on IP range                                   | MAC: 00:0c:29:e2:18:b4<br>IP: 192.168.1.xxx        |

#### 3. Comportement suspect MAC

|MAC suspecte|Activité|
|---|---|
|même MAC → plusieurs IP|Spoofing|
|claims gateway IP|MITM probable|
|spam ARP requests|Flooding|

### 🧪 Cas analysé (synthèse)

📊 Mapping normal

|MAC|IP|
|---|---|
|b4|192.168.1.25 (attaquant)|
|f4|192.168.1.1 (gateway)|
|a8|192.168.1.12 (victime)|

🚨 Anomalies détectées

|Type|Observation|
|---|---|
|Spoofing|2 MAC → 192.168.1.1|
|Spoofing|MAC b4 prétend être gateway|
|Flooding|ARP requests massives|
|Scan|Range 192.168.1.xxx|

### 🌐 Confirmation MITM

🔍 Indice clé

- Trafic HTTP **normal côté IP**
- MAIS :  
    👉 tous les paquets passent par **MAC b4**

➡️ Donc :

- attaquant intercepte trafic
- redirige vers vraie destination

**http ip traffic**

![[wireshark traffic analysis http ip traffic.png]]

**http mac traffic**

![[wireshark traffic analysis http mac traffic.png]]

### ⚡ Méthodologie analyste

1. Repérer anomalies ARP
2. Identifier conflits IP/MAC
3. Détecter flooding / scan
4. Corréler avec autres protocoles (HTTP, etc.)
5. Ajouter colonnes MAC dans Wireshark ⚠️ (clé pour MITM)

---

## Identifying Hosts: DHCP, NetBIOS and Kerberos

Identifier **hosts + utilisateurs** via :

- DHCP
- NetBIOS (NBNS)
- Kerberos

👉 Complémentaire à IP ↔ MAC

### 🧩 Protocoles clés

|Protocole|Rôle|
|---|---|
|DHCP|Attribution IP + hostname|
|NBNS|Résolution nom ↔ IP|
|Kerberos|Authentification utilisateur|

### 🌐 DHCP Analysis

⚙️ Rôle

- Attribution automatique :
    - IP
    - hostname
    - paramètres réseau

🔍 Filtres

| Type    | Filtre                  |
| ------- | ----------------------- |
| Global  | `dhcp or bootp`         |
| Request | `dhcp.option.dhcp == 3` |
| ACK     | `dhcp.option.dhcp == 5` |
| NAK     | `dhcp.option.dhcp == 6` |

![[wireshark traffic analysis dhcp.png]]

#### DHCP Request

|Option|Info|
|---|---|
|12|Hostname|
|50|IP demandée|
|51|Lease time|
|61|MAC client|

👉 Filtre :

dhcp.option.hostname contains "keyword"

#### DHCP ACK

|Option|Info|
|---|---|
|15|Domain name|
|51|Lease time|

#### DHCP NAK

|Option|Info|
|---|---|
|56|Raison refus|

👉 ⚠️ Lire le message (pas juste filtrer)

### 🖥️ NetBIOS (NBNS)

⚙️ Rôle

- Communication entre hôtes
- Résolution nom ↔ IP

🔍 Filtres

|Type|Filtre|
|---|---|
|Global|`nbns`|
|Recherche nom|`nbns.name contains "keyword"`|

📊 Infos utiles

- Nom machine
- IP
- TTL

![[wireshark traffic analysis netbios.png]]

### 🔐 Kerberos Analysis

⚙️ Rôle

- Authentification (Windows domain)

🔍 Filtres

| Type             | Filtre                                    |
| ---------------- | ----------------------------------------- |
| Global           | `kerberos`                                |
| Username         | `kerberos.CNameString contains "keyword"` |
| Exclure machines | `!(kerberos.CNameString contains "$")`    |

📊 Infos utiles

|Champ|Description|
|---|---|
|CNameString|Username ⚠️|
|(avec $)|Hostname|
|pvno|Version protocole|
|realm|Domaine|
|sname|Service|
|addresses|IP client + NetBIOS|

👉 Exemples :

kerberos.pvno == 5  
kerberos.realm contains ".org"

![[wireshark traffic analysis kerberos.png]]

### ⚡ Points clés

- DHCP → **associer IP + hostname**
- NBNS → **résolution noms réseau**
- Kerberos → **identifier utilisateurs**
- ⚠️ Différencier :
    - user → sans `$`
    - machine → avec `$`

### 🧪 Méthodo analyste

1. DHCP → identifier machines
2. NBNS → confirmer noms
3. Kerberos → identifier utilisateurs
4. Corréler avec trafic suspect

### LAB thm

#### What is the MAC address of the host "Galaxy A30"?

![[wireshark traffic analysis t4 q1.png]]

réponse : 9a:81:41:cb:96:6c

#### How many NetBIOS registration requests does the "LIVALJM" workstation have?

![[wireshark traffic analysis t4 q2.png]]

réponse : 16

#### Which host requested the IP address "172.16.13.85"?

![[wireshark traffic analysis t4 q3.png]]

réponse : Galaxy-A12

#### What is the IP address of the user "u5"? (Enter the address in defanged format.)

![[wireshark traffic analsyis t4 q4.png]]

réponse : 10[.]1[.]12[.]2

#### What is the hostname of the available host in the Kerberos packets?

Right clicking the “CNameString” field in the bottom left and apply as column will help!

réponse : xp1$

---

## Tunneling Traffic: DNS and ICMP

- Détecter **tunneling / exfiltration / C2**
- Protocoles utilisés :
    - ICMP
    - DNS

👉 Pourquoi ?

- Protocoles **trusted → passent les firewalls**
- Permettent **encapsulation de données malveillantes**

**⚙️ Principe du tunneling**

|Élément|Description|
|---|---|
|Encapsulation|Données cachées dans trafic légitime|
|Objectif|Exfiltration / C2|
|Avantage attaquant|Discrétion + bypass sécurité|

### 📡 ICMP Analysis

**⚙️ Rôle normal**

- Diagnostic réseau (ping, erreurs)

**🚨 Usage malveillant**

- Exfiltration de données
- C2 communication
- DoS

**🔍 Indicateurs**

|Indicateur|Description|
|---|---|
|Volume élevé ICMP|Activité anormale|
|Taille paquet élevée|Payload suspect|
|Payload|Contient données encapsulées|

👉 Filtre :

data.len > 64 and icmp

**⚠️ Limite**

- Attaquant peut imiter taille normale (~64 bytes)

![[wireshark traffic analysis icmp.png]]

### 🌐 DNS Analysis

**⚙️ Rôle normal**

- Résolution nom → IP

**🚨 Usage malveillant**

- DNS tunneling (C2)
- Exfiltration via requêtes DNS

**🔍 Indicateurs**

|Indicateur|Description|
|---|---|
|Longueur requête élevée|Anormal|
|Sous-domaines longs|Données encodées|
|Domaine suspect|C2|
|Volume élevé|Beaconing|

👉 Exemple :

encoded-data.maliciousdomain.com

**🔍 Filtres**

|Objectif|Filtre|
|---|---|
|Global|`dns`|
|Pattern connu|`dns contains "dnscat"`|
|Long requêtes|`dns.qry.name.len > 15 and !mdns`|

**!mdns:** Disable local link device queries.


![[wireshark traffic analysis dns.png]]

Detecting suspicious activities in chunked files is easy and a great way to learn how to focus on the details
### ⚡ Comparaison ICMP vs DNS

|Critère|ICMP|DNS|
|---|---|---|
|Usage normal|Diagnostic|Résolution|
|Détection|Taille / volume|Longueur / nom|
|Discrétion|Moyenne|Élevée|
|Payload|Direct|Encodé|

### 🧪 Méthodologie analyste

1. Identifier trafic inhabituel
2. Vérifier taille paquets
3. Analyser contenu / structure
4. Repérer patterns (long domain, data ICMP)
5. Corréler avec infection (timing ⚠️)

### Lab thm

#### Use the "Desktop/exercise-pcaps/dns-icmp/icmp-tunnel.pcap" file. Investigate the anomalous packets. Which protocol is used in ICMP tunnelling?

![[wireshark traffic analysis t5 q1.png]]

réponse : SSH

#### Use the "Desktop/exercise-pcaps/dns-icmp/dns.pcap" file.Investigate the anomalous packets. What is the suspicious main domain address that receives anomalous DNS queries? (Enter the address in defanged format.)

![[wireshark traffic analysis t5 q2.png]]

réponse : dataexfil[.]com

---

## Cleartext Protocol Analysis : FTP

Analyser FTP pour détecter :

- credentials en clair
- activités suspectes (bruteforce, exfiltration, etc.)

**⚙️ FTP — Rappels**

|Caractéristique|Détail|
|---|---|
|Type|Cleartext ❌|
|Sécurité|Faible|
|Usage|Transfert de fichiers|

**🚨 Risques principaux**

- MITM
- Vol de credentials
- Accès non autorisé
- Malware upload
- Data exfiltration

**🔍 Filtres Wireshark**

|Objectif|Filtre|
|---|---|
|Global|`ftp`|

### 📊 Codes FTP importants

- **x1x series:** Information request responses.
- **x2x series:** Connection messages.
- **x3x series:** Authentication messages.

**Note:** "200" means command successful.

#### 🔹 x1x (Info)

| Code | Signification    |
| ---- | ---------------- |
| 211  | System status    |
| 212  | Directory status |
| 213  | File status      |

👉 Filtre :

ftp.response.code == 211

#### 🔹 x2x (Connexion)

|Code|Signification|
|---|---|
|220|Service ready|
|227|Passive mode|
|228|Long passive|
|229|Extended passive|

👉 Filtre :

ftp.response.code == 227

#### 🔹 x3x (Auth)

|Code|Signification|
|---|---|
|230|Login OK ✅|
|331|Username OK|
|430|Invalid user|
|530|Login failed ❌|

👉 Filtre :

ftp.response.code == 230

### 🔑 Commandes importantes

|Commande|Description|
|---|---|
|USER|Username|
|PASS|Password|
|CWD|Change directory|
|LIST|List files|

👉 Filtres :

ftp.request.command == "USER"

ftp.request.command == "PASS"

### 🚨 Détection d’attaques

**🔥 Bruteforce**

|Indicateur|Filtre|
|---|---|
|Échecs répétés|`ftp.response.code == 530`|
|User ciblé|`(ftp.response.code == 530) and (ftp.response.arg contains "username")`|

**🎯 Password Spray**

|Indicateur|Filtre|
|---|---|
|Même password|`(ftp.request.command == "PASS") and (ftp.request.arg == "password")`|

![[wireshark traffic analysis ftp.png]]

### 🧪 Méthodologie analyste

1. Identifier logins (USER/PASS)
2. Vérifier succès/échec
3. Détecter patterns répétitifs
4. Analyser transferts fichiers
5. Corréler avec autres activités

### Lab thm

#### How many incorrect login attempts are there?

![[wireshark traffic analysis t6 q1.png]]

réponse : 737

#### What is the size of the file accessed by the "ftp" account?

![[wireshark traffic analysis t6 q2.png]]

réponse : 39424

#### The adversary uploaded a document to the FTP server. What is the filename?

ftp.response.code = 230 to see if there is a login. then follow the tcp stream of this login conduct us here :

![[wireshark traffic analysis t6 q3.png]]

réponse : resume.doc

#### The adversary tried to assign special flags to change the executing permissions of the uploaded file. What is the command used by the adversary?

always on the same stream :

![[wireshark traffic analysis t6 q4.png]]

réponse : chmod 777

---

## Cleartext Protocol Analysis : HTTP

Hypertext Transfer Protocol (HTTP) is a cleartext-based, request-response and client-server protocol. It is the standard type of network activity to request/serve web pages, and by default, it is not blocked by any network perimeter. As a result of being unencrypted and the backbone of web traffic, HTTP is one of the must-to-know protocols in traffic analysis. Following attacks could be detected with the help of HTTP analysis:

Détecter :

- Phishing
- Web attacks
- Exfiltration
- C2

👉 HTTP = **très utilisé + non chiffré → cible idéale**

**🔍 Filtres de base**

|Objectif|Filtre|
|---|---|
|HTTP|`http`|
|HTTP2|`http2`|

**📊 Méthodes HTTP**

|Méthode|Usage|
|---|---|
|GET|Récupération|
|POST|Envoi données|

👉 Filtres :

http.request.method == "GET"  
http.request.method == "POST"  
http.request

**📊 Codes HTTP importants**

|Code|Signification|
|---|---|
|200|OK|
|301|Redirect permanent|
|302|Redirect temporaire|
|400|Bad request|
|401|Unauthorized|
|403|Forbidden|
|404|Not found|
|405|Method not allowed|
|408|Timeout|
|500|Server error|
|503|Service down|

👉 Exemple :

http.response.code == 404

**🔑 Paramètres importants**

|Champ|Description|
|---|---|
|user-agent|navigateur / OS|
|URI|ressource demandée|
|full URI|URL complète|
|host|serveur|
|server|type serveur|
|data-text-lines|contenu clair|
User Agent : https://explore.whatismybrowser.com/useragents/explore/
**URI:** Uniform Resource Identifier

👉 Exemples :

http.user_agent contains "nmap"  
http.request.uri contains "admin"  
http.host contains "keyword"  
data-text-lines contains "keyword"

![[wireshark traffic analysis http user agent.png]]

### 🚨 User-Agent Analysis

**🔍 Indicateurs**

|Indicateur|Exemple|
|---|---|
|Outils pentest|nmap, sqlmap, wfuzz, nikto|
|Différents UA même host|suspect|
|Fautes typo|Mozlila|
|UA custom|anormal|
|Payload dans UA|très suspect|

👉 Filtre :

(http.user_agent contains "sqlmap") or (http.user_agent contains "Nmap")

⚠️ Ne jamais se fier uniquement au user-agent

### 💥 Log4j Detection

**🔍 Indicateurs clés**

|Indicateur|Description|
|---|---|
|POST request|vecteur attaque|
|"jndi:ldap"|signature|
|"Exploit.class"|payload|
|caractères suspects|`$`, `==`|

**🎯 Filtres**

http.request.method == "POST"  
(ip contains "jndi") or ( ip contains "Exploit")
(frame contains "jndi") or (frame contains "Exploit")  
(http.user_agent contains "$") or (http.user_agent contains "==")

### 🧪 Méthodologie analyste

1. Identifier requêtes (GET/POST)
2. Vérifier réponses (codes HTTP)
3. Analyser URI / paramètres
4. Inspecter user-agent
5. Chercher patterns connus (ex: Log4j)
6. Corréler avec comportement global

### ⚡ Points clés

- HTTP = **mine d’or en analyse (cleartext)**
- POST = souvent critique ⚠️
- User-Agent = **indice, pas preuve**
- Long URI / contenu suspect = red flag 🚨

### Lab thm

#### Investigate the user agents. What is the number of anomalous  "user-agent" types?

- Windows 6.4 (because of the hint lol)
- Mozilla 5.0 with sus URI/query
- Mozilla 5.0 with nmap script
- Mozlila 5.0
- Wfuzz 2.4
- sqlmap 1.4

réponse : 6

#### What is the packet number with a subtle spelling difference in the user agent field?

![[wireshark traffic analysis t7 q2.png]]

réponse : 52

#### Locate the "Log4j" attack starting phase. What is the packet number?

![[wireshark traffic analysis t7 q3.png]]

réponse : 444

#### Locate the "Log4j" attack starting phase and decode the base64 command. What is the IP address contacted by the adversary? (Enter the address in defanged format and exclude "{}".)

![[wireshark traffic analysis t7 q4.png]]

réponse : 62[.]210[.]130[.]250

---

## Encrypted Protocol Analysis : Decrypting HTTPS

- Analyser trafic **chiffré HTTPS**
- Accéder au contenu via **clé de déchiffrement**

### 🔐 HTTPS — Rappels

|Élément|Détail|
|---|---|
|Protocole|HTTPS|
|Chiffrement|TLS|
|Objectif|Confidentialité + intégrité|

👉 Sans clé → **contenu illisible**

**⚠️ Point important**

- HTTPS protège…
- MAIS aussi utilisé par attaquants (C2, malware)

### 🔍 Filtres Wireshark

|Objectif|Filtre|
|---|---|
|TLS global|`tls`|
|HTTP requests|`http.request`|
|Client Hello|`tls.handshake.type == 1`|
|Server Hello|`tls.handshake.type == 2`|
|Exclure SSDP|`!(ssdp)`|

![[wireshark traffic analysis https.png]]

### 🤝 TLS Handshake

|Étape|Description|
|---|---|
|Client Hello|Client initie connexion|
|Server Hello|Serveur répond|

👉 Filtres combinés :

(http.request or tls.handshake.type == 1) and !(ssdp)

![[wireshark traffic analysis tls.handshake.type == 1.png]]

(http.request or tls.handshake.type == 2) and !(ssdp)

![[wireshark traffic analysis tls.handshake.type == 2.png]]

### 🔑 Déchiffrement HTTPS

**⚙️ Principe**

- Utiliser **key log file (SSLKEYLOGFILE)**
- Contient clés TLS par session

**📊 Conditions**

|Condition|Importance|
|---|---|
|Capture + génération clé simultanée|✅ obligatoire|
|Navigateur compatible|Chrome / Firefox|
|Session active|Clé unique par session|

#### 🛠️ Configuration

1. Définir variable :
    
    SSLKEYLOGFILE
    
2. Naviguer → clés générées
3. Importer dans Wireshark :
    - Preferences → TLS → key log file
    - or right-click menu

**right click menu**

![[wireshark traffic analysis import key right click.png]]

**Preferences → TLS → key log file**

![[wireshark traffic analysis import key preference.png]]

### 🔍 Résultat après déchiffrement

|Avant|Après|
|---|---|
|Données illisibles|Contenu HTTP visible|
|TLS uniquement|HTTP/HTTP2 détaillé|
|Pas d’URL|URL + payload|
**Après déchiffrement**

![[wireshark trafic analysis déchiffrement.png]]


**📦 Données accessibles**

- Decrypted TLS
- HTTP headers
- Reassembled TCP
- Reassembled SSL
- Decompressed headers

### 🧪 Méthodologie analyste

1. Identifier flux TLS
2. Repérer Client/Server Hello
3. Charger key log file
4. Déchiffrer trafic
5. Analyser HTTP caché

### ⚡ Points clés

- Sans clé → **analyse limitée**
- Clé = générée **au moment de la session ⚠️**
- HTTPS ≠ trafic sûr
- Déchiffrement = **game changer en analyse**

### Lab thm

#### What is the frame number of the "Client Hello" message sent to "accounts.google.com"?

(http.request or tls.handshake.type == 1) and !(ssdp) and search for account.google.com in "Extension: server_name -> Server Name:"

![[wireshark traffic analysis t8 q1.png]]

réponse : 16

#### Decrypt the traffic with the "KeysLogFile.txt" file. What is the number of HTTP2 packets?

Decrypt in preference -> protocol -> tls and take the keyslogfile.txt then filter with http2

réponse : 115

#### Go to Frame 322. What is the authority header of the HTTP2 packet? (Enter the address in defanged format.)

ctrl + g "322"

![[wireshark traffic analysis t8 q3.png]]

réponse : safebrowsing[.]googleapis[.]com

#### Investigate the decrypted packets and find the flag! What is the flag?

Méthode 1 : Find a Packet "flag.txt" search for http code 200 and check the payload
Méthode 2 : File > export object > http... then "flag" in text filter then save and open the file or go to No. of the packet and check payload

![[wireshark traffic analysis t8 q4.png]]

réponse : FLAG{THM-PACKETMASTER}

---

## Bonus : Hunt Cleartext Credentials!

- Détecter credentials en clair = difficile (trafic ressemble au normal)
- Wireshark propose une aide :

**🔍 Feature**

**Tools → Credentials**

- Liste :
    - username
    - password
    - protocole
    - packet associé

**⚠️ Limites**

- Supporte seulement :
    - FTP, HTTP, IMAP, POP, SMTP
- Pas fiable à 100% → toujours vérifier manuellement

**🎯 Objectif**

👉 Visualiser rapidement les credentials → accélérer détection brute force / fuite

### Lab thm

