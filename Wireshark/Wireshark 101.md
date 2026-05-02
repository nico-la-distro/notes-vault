DisplayFilters
https://wiki.wireshark.org/DisplayFilters
Filtering Documentation
https://www.wireshark.org/docs/wsug_html_chunked/ChWorkBuildDisplayFilterSection.html


## Collection Methods

Comprendre les méthodes de collecte de PCAP avant analyse.  
Capture Wireshark simple, mais obtention du trafic = difficulté principale.

Méthodes :

- taps
- port mirroring
- MAC floods
- ARP Poisoning

### Network Taps

Implant physique sur câble → sniff/capture trafic  
Utilisé par Threat Hunting / DFIR / Red Team

Deux types :

|Type|Principe|
|---|---|
|Hardware tap|Intercepte trafic directement (ex: vampire tap)|
|Inline tap|Placé entre deux équipements, réplique les paquets (ex: Throwing Star LAN Tap)|

![[Wireshark types of tap.png]]

### MAC Floods

Remplit la table CAM du switch → saturation

Effet :

- switch ne peut plus apprendre de nouvelles MAC
- envoie trafic sur tous les ports

⚠️ Utilisation :

- intrusive
- nécessite autorisation explicite

### ARP Poisoning

Redirige le trafic vers l’attaquant

Caractéristiques :

- sniff actif
- moins stressant pour le réseau que MAC Flooding

⚠️ Utilisation :

- prudence
- seulement si taps indisponibles

---

## Filtering Captures

Filtrage essentiel (captures volumineuses, 100k+ paquets)

Deux types :

- capture filters
- display filters (plus puissants, via onglet Analyze ou barre de filtre)

### Filtering Operators

Opérateurs logiques :

|Opérateur|Syntaxe|
|---|---|
|and|`and` / `&&`|
|or|`or` / `|
|equals|`eq` / `==`|
|not equal|`ne` / `!=`|
|greater than|`gt` / `>`|
|less than|`lt` / `<`|

Autres :

- contains
- matches
- bitwise_and

### Basic Filtering

|Cas|Syntaxe|
|---|---|
|Syntaxe générale|`[protocole].[champ] [opérateur] [valeur]`|
|Filtering by IP|`ip.addr == [IP Address]`|
|Filtering by SRC and DST|`ip.src == [SRC IP Address] and ip.dst == [DST IP Address]`|
|Filtering by TCP (port)|`tcp.port eq [Port #]`|
|Filtering by TCP (protocole)|`tcp.port eq [Protocol Name]`|
|Filtering by UDP (port)|`udp.port eq [Port #]`|
|Filtering by UDP (protocole)|`udp.port eq [Protocol Name]`|

---

## Packet Dissection

Utilisation du modèle OSI pour analyser les paquets (connaissance préalable requise)

### Packet Details

Double clic sur un paquet → détails  
Un paquet contient 5 à 7 couches (OSI)

|Couche|Contenu|
|---|---|
|Frame (Layer 1)|Infos paquet + couche physique|
|Source [MAC] (Layer 2)|MAC source + destination|
|Source [IP] (Layer 3)|IP source + destination|
|Protocol (Layer 4)|TCP/UDP + ports source/destination|
|Protocol Errors|Segments TCP à réassembler|
|Application Protocol (Layer 5)|Protocole (HTTP, FTP, SMB…)|
|Application Data|Données applicatives|

Structure typique visible :

- frame/packet
- MAC
- IP
- protocole
- erreurs protocole
- protocole applicatif
- données applicatives

---

## ARP Traffic

ARP (Layer 2) → mapping IP ↔ MAC

Types :

- Request (opcode 1)
- Reply (opcode 2)

Identifier :

- champ Opcode

Notes :

- appareils souvent identifiés (ex: Intel_78)
- trafic suspect : nombreuses requêtes source inconnue

Activer résolution MAC :  
`View > Name Resolution > Resolve Physical Addresses`

---

## ICMP Traffic

You should already be familiar with how ICMP works; however, if you need a refresher, read the [IETF documentation(opens in new tab)](https://tools.ietf.org/html/rfc792).

ICMP → analyse réseau (ping, traceroute)

Flux typique :

- Request → serveur
- Reply → serveur → client

### ICMP request:

Points clés :

|Champ|Valeur|
|---|---|
|Type|8 = Request|
|Code|indication complémentaire|
|Timestamp|heure du ping|
|Data|données (souvent aléatoires)|

⚠️ Type/code anormaux → activité suspecte

### ICMP Reply:

Points clés :

|Champ|Valeur|
|---|---|
|Type|0 = Reply|
|Code|confirme réponse|

Différence principale :

- Type = 0 (reply)

Analyse identique au request

---

## TCP Traffic

You should already have an understanding of how TCP works, if you need a refresher check out the [IETF TCP Documentation(opens in new tab)](https://tools.ietf.org/html/rfc793).

TCP → livraison des paquets (séquencement, erreurs)

Exemple :

- scan Nmap → RST, ACK ⇒ port fermé

Wireshark :

- colorisation selon niveau

Limite :

- grand volume de paquets → analyse complexe
- outils complémentaires : RSA NetWitness, NetworkMiner

### TCP Handshake

Séquence :

- SYN
- SYN-ACK
- ACK

→ établissement connexion

⚠️ Anomalies :

- ordre incorrect
- présence de RST

⇒ activité suspecte

### TCP Packet Analysis

Points clés :

- sequence number
- acknowledgment number

SYN packet :

- acknowledgment number = 0

Afficher sequence number réel :

Edit > Preferences > Protocols > TCP > Relative sequence numbers (disable)

Note :

- analyse globale (flux), pas paquet isolé

---

## DNS Traffic

you should be familiar with DNS; however, if you're not you can refresh with the [IETF DNS Documentation(opens in new tab)](https://www.ietf.org/rfc/rfc1035.txt).

DNS → résolution nom ↔ IP

Caractéristiques normales :

- Query ↔ Response
- DNS servers uniquement
- UDP

⚠️ Anomalies → suspect :

- autre protocole (ex: TCP)
- comportement hors norme

Observation :

- domaine query visible directement → détection rapide

### DNS Query:

Points clés :

- port : UDP 53 (normal)
- TCP 53 → suspect
- origine de la requête
- domaine demandé

Note :

- dépend du contexte réseau

### DNS Response:

Points clés :

- similaire à Query
- contient :
    - réponse (IP)

→ permet validation de la requête

---

## HTTP Traffic

if you need a refresher you can read the official paper by the [IETF on HTTP methods(opens in new tab)](https://www.ietf.org/rfc/rfc2616.txt).

HTTP → requêtes web non chiffrées

Utilisation :

- GET / POST
- pages web
- serveurs web

Analyse utile pour :

- SQLi
- web shells
- attaques web

Note :

- HTTPS = version chiffrée (plus courant)

#### HTTP Traffic Overview

Caractéristiques :

- pas de handshake complexe
- protocole direct
- données lisibles (non chiffrées)

Informations visibles :

- Request URI
- File Data
- Server

### Protocol Hierarchy

Statistics > Protocol Hierarchy

→ vue des protocoles utilisés

Utilité :

- détection anomalies
- threat hunting

### Export HTTP Object

File > Export Objects > HTTP

→ liste des URIs / objets HTTP

Utilité :

- identification rapide contenu web

### Endpoints

Statistics > Endpoints

→ liste IPs / endpoints

Utilité :

- repérer sources / destinations suspectes

---

## HTTPS Traffic

HTTPS = HTTP chiffré → nécessite handshake + tunnel sécurisé

Étapes :

- accord version protocole
- choix algorithme cryptographique
- authentification (optionnelle)
- création tunnel avec clé publique

### HTTPS Handshake

| Étape               | Contenu / Rôle                                                                  |
| ------------------- | ------------------------------------------------------------------------------- |
| Client Hello        | SSL version<br>SSL record layer<br>paramètres de handshake                      |
| Server Hello        | session details<br>certificat SSL<br>paramètres serveur                         |
| Client Key Exchange | définition clé publique<br>base du chiffrement futur                            |
| Fin handshake       | serveur confirme clé publique<br>tunnel sécurisé créé<br>trafic suivant chiffré |

### HTTPS Encryption

Après handshake :

- Application Data = chiffré
- impossible à lire sans clé RSA

### Déchiffrement RSA (Wireshark)

Configuration :

Edit > Preferences > Protocols > TLS
  
(ou SSL selon version)

Paramètres :

- IP Address : 127.0.0.1
- Port : start_tls
- Protocol : http
- Keyfile : clé RSA

Résultat :

- trafic déchiffré
- HTTP visible

Données visibles après déchiffrement :

- Request URI
- User-Agent

Utilité :

- threat hunting
- administration réseau

### Export HTTP Object

File > Export Objects > HTTP

→ extraction des objets HTTP depuis trafic déchiffré


---

## Analyzing Exploit PCAPs

check task 13 -> https://tryhackme.com/room/wireshark
### Zerologon PCAP Overview

Contexte :

- Exploit Windows AD : CVE-2020-1472 (Zerologon)
- Domain Controller : `192.168.100.6`
- Attaquant : `192.168.100.128`

### Identifying the Attacker

Observations initiales :

- trafic normal : OpenVPN, ARP
- trafic suspect : DCERPC, EPM

Conclusion :

- `192.168.100.128` = source des requêtes  
    → identifié comme attaquant

Filtrage :

ip.src == 192.168.100.128

### Zerologon POC Connection Analysis

Indices (IOCs) :

- multiples connexions RPC
- requêtes DCERPC
- modification machine account password

Lien exploit :

- comportement typique Zerologon
- validation via analyse PCAP

### Secretsdump SMB Analysis

Observations :

- trafic SMB2/3
- trafic DRSUAPI

Interprétation :

- usage de secretsdump
- extraction de hashes
- abus SMB + DRSUAPI

### Conclusion d’analyse

- exploitation Zerologon confirmée via artefacts réseau
- séquence d’attaque identifiable dans PCAP
- identification attaquant via IP source + protocoles RPC/SMB

Étape suivante (hors scope) :

- isolation hôte
- reporting incident
- threat hunting / DFIR response

---

## Suite ⚠️

If you're looking to get more practice with Wireshark you can check out their [Wireshark Sample Captures(opens in new tab)](https://wiki.wireshark.org/SampleCaptures). Or if you're looking for a real would Threat Hunting challenge you can check out [Case: 001 PCAP Analysis(opens in new tab)](https://dfirmadness.com/case-001-pcap-analysis/) by DFIR Madness.

If you want to continue working on your analysis skills on Tryhackme, check out [Overpass 2 - Hacked](https://tryhackme.com/room/overpass2hacked).

