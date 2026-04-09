- Processus de :
    - **capturer**
    - **inspecter**
    - **analyser** le trafic réseau
- Objectif :  
    👉 **visibilité complète** des communications (interne + externe)

⚠️ Important :

- NTA ≠ outil comme Wireshark
- NTA = combinaison de :
    - corrélation de logs
    - **Deep Packet Inspection (DPI)**
    - statistiques de flux réseau
    - objectifs précis d’analyse

---

## Purpose of Network Traffic Analysis

👉 Objectif principal :  
**Comprendre + détecter ce qui est caché derrière les communications réseau**

### e.g DNS tunneling and beaconing

You are an SOC analyst, and you receive an alert stating that an unusual number of DNS queries are coming from a host named WIN-016 with IP 192.168.1.16. The DNS logs on the firewall show multiple DNS queries going to the same TLD, each time using a different subdomain.

```bash
2025-10-03 09:15:23    SRC=192.168.1.16      QUERY=aj39skdm.malicious-tld.com    QTYPE=A      
2025-10-03 09:15:31    SRC=192.168.1.16      QUERY=msd91azx.malicious-tld.com    QTYPE=A     
2025-10-03 09:15:45    SRC=192.168.1.16      QUERY=cmd01.malicious-tld.com       QTYPE=TXT     
2025-10-03 09:15:45    SRC=192.168.1.16      QUERY=cmd01.malicious-tld.com       QTYPE=TXT 
```

#### Indices suspects

- Beaucoup de requêtes DNS
- Même domaine (TLD)
- Sous-domaines aléatoires
- Requêtes répétées

```json
Domain Name System (response)
    Transaction ID: 0x4a2b
    Flags: 0x8180 Standard query response, No error
        1... .... .... .... = Response: Message is a response
        .... .... .... 0000 = RCODE: No error (0)
    Questions: 1
    Answer RRs: 1
    Authority RRs: 0
    Additional RRs: 0
    Queries
        cmd1.evilc2.com: type TXT, class IN
    Answers
        cmd1.evilc2.com: type TXT, class IN, TTL 60, TXT length: 20
            TXT: "SSBsb3ZlIHlvdXIgY3VyaW91c2l0eQ=="
```

#### Ce que cache réellement le trafic

- Utilisation de **TXT records**
- Transmission de **commandes C2 (Command & Control)**
- Données encodées (ex: Base64)

👉 Exemple :

TXT: "SSBsb3ZlIHlvdXIgY3VyaW91c2l0eQ=="

---

### Rôle clé du NTA & Utilisations

- Inspecter le **contenu réel des paquets**
- Comprendre le **contexte et l’intention**
- Détecter des techniques avancées :
    - DNS tunneling
    - exfiltration de données
    - C2

| Usage                 | Exemple                           |
| --------------------- | --------------------------------- |
| Performance réseau    | Détection de pics / lenteurs      |
| Détection d’anomalies | Trafic inhabituel                 |
| Analyse de contenu    | ZIP malveillant, C2, exfiltration |

**Pour un SOC**

| Fonction      | Description              |
| ------------- | ------------------------ |
| Détection     | Activités malveillantes  |
| Investigation | Reconstruction d’attaque |
| Validation    | Vérification des alertes |

---

## What Network Traffic can we Observe

👉 On observe le trafic via le **modèle TCP/IP (4 couches)**  
➡️ Chaque couche ajoute des **headers + données**

⚠️ Important :

- Logs = **partiel**
- NTA = **packet complet (full visibilité)**

![[Network Traffic (TCP-IP).png]]

|Couche|Ce qu’on voit|Limite des logs|Intérêt sécurité|
|---|---|---|---|
|Application|Requêtes + données (payload)|❌ pas de contenu|Malware, exfiltration|
|Transport|Ports, flags, séquences|❌ champs incomplets|Hijacking|
|Internet|IP, fragmentation|❌ détails fragments|Evasion IDS|
|Link|MAC, ARP|❌ contexte réseau|ARP spoofing|

---

### 1. Application Layer (HTTP, DNS...)

**_request_**

```json
GET /downloads/suspicious_package.zip HTTP/1.1
Host: www.tryhackrne.thn
User-Agent: curl/7.85.0
Accept: */*
Connection: close
```

**_response_**

```json
HTTP/1.1 200 OK
Date: Mon, 29 Sep 2025 10:15:30 GMT
Server: nginx/1.18.0
Content-Type: application/zip
Content-Length: 10485760
Content-Disposition: attachment; filename="suspicious_package.zip"
Last-Modified: Mon, 29 Sep 2025 09:54:00 GMT
ETag: "5d8c72-9f8a1c-3a2b4c"
Accept-Ranges: bytes
Connection: close

[binary ZIP file bytes follow — 10,485,760 bytes]
```

**✅ Ce qu’on voit**

- Headers (GET, status code…)
- **Payload (contenu réel)**

**❌ Logs**

- voient :
    - fichier demandé (`.zip`)
- ne voient pas :
    - contenu du fichier

👉 Exemple :

- Logs : `suspicious_package.zip`
- NTA : **le vrai contenu du ZIP**

---

### 2.  Transport Layer (TCP/UDP)

**_Firewall logs_**

```json
2025-10-13 09:15:32 ACCEPT TCP src=192.168.1.45 dst=172.217.22.14 sport=51432 dport=443 flags=SYN len=60
2025-10-13 09:15:32 ACCEPT TCP src=172.217.22.14 dst=192.168.1.45 sport=443 dport=51432 flags=SYN,ACK len=60
```

**_Wireshark Capture_**

```json
No.     Time        Source          Destination     Protocol Length  Info
1       0.000000    192.168.1.45    172.217.22.14   TCP      74      51432 → 80 [SYN] Seq=0 Win=64240 Len=0 MSS=1460
2       0.000120    172.217.22.14   192.168.1.45    TCP      74      80 → 51432 [SYN, ACK] Seq=0 Ack=1 Win=65535 Len=0 MSS=1460
3       0.000220    192.168.1.45    172.217.22.14   TCP      66      51432 → 80 [ACK] Seq=1 Ack=1 Win=64240 Len=0
4       0.010500    192.168.1.45    172.217.22.14   TCP      1514    51432 → 80 [PSH, ACK] Seq=1 Ack=1 Win=64240 Len=1460
5       0.010620    172.217.22.14   192.168.1.45    TCP      66      80 → 51432 [ACK] Seq=1 Ack=1461 Win=65535 Len=0
6       0.020100    192.168.99.200  172.217.22.14   TCP      74      51432 → 80 [PSH, ACK] Seq=34567232 Ack=1 Win=64240 Len=20  
```

- The first 3 lines show a normal TCP 3-way handshake
- Lines 4 and 5 show legitimate data transfer
- Line 6 shows a packet from another source trying to inject itself into the session. Note the massive jump in the sequence number

**✅ Ce qu’on voit**

- Ports (443, etc.)
- Flags (SYN, ACK…)
- **Sequence numbers**

**⚠️ Exemple attaque : Session Hijacking**

- paquet injecté avec :
    - mauvais sequence number

👉 Indice clé :  
➡️ **gros saut dans les sequence numbers**

---

### 3. Internet Layer (IP)

```json
No.   Time       Source        Destination   Protocol Length Info
1     0.000000   203.0.113.45  192.168.1.10  UDP      1514    Fragmented IP protocol (UDP) (id=0x1a2b) [MF] Offset=0, Len=1480
2     0.000015   203.0.113.45  192.168.1.10  UDP      1514    Fragmented IP protocol (UDP) (id=0x1a2b) [MF] Offset=1480, Len=1480
3     0.000030   203.0.113.45  192.168.1.10  UDP       600    Fragmented IP protocol (UDP) (id=0x1a2b) Offset=1480, Len=64   <-- Overlap
4     0.000045   192.168.1.10  203.0.113.45  ICMP      98     Destination unreachable (Fragment reassembly time exceeded)
```

**✅ Ce qu’on voit**
- IP source/destination
- TTL
- fragmentation (offset, length)

**⚠️ Exemple attaque : Fragmentation attack**

👉 Technique :
- fragments qui se **chevauchent (overlap)**

➡️ But :
- tromper IDS
- contourner détection

---

### 4. Link Layer (MAC/ARP)

```json
No.   Time       Source           Destination      Protocol Length Info
1     0.000000   192.168.1.1      Broadcast        ARP      60     Who has 192.168.1.10? Tell 192.168.1.1
2     0.000025   192.168.1.10     192.168.1.1      ARP      60     192.168.1.10 is at 00:11:22:33:44:55
3     1.002010   192.168.1.200    192.168.1.1      ARP      60     192.168.1.10 is at aa:bb:cc:dd:ee:ff  <-- Attacker spoof
4     1.002015   192.168.1.200    192.168.1.10     ARP      60     192.168.1.1 is at aa:bb:cc:dd:ee:ff  <-- Attacker spoof
5     1.100000   192.168.1.10     172.217.22.14    TCP      74     54433 → 80 [SYN] Seq=0 Win=64240 Len=0
6     1.100120   192.168.1.200    172.217.22.14    TCP      74     54433 → 80 [SYN] Seq=0 Win=64240 Len=0  <-- Relayed via attacker
```

**✅ Ce qu’on voit

- adresses MAC
- ARP requests/replies

**⚠️ Exemple attaque : ARP Poisoning

👉 Attaquant :

- envoie fausses réponses ARP
- associe **sa MAC à d’autres IP**

➡️ Résultat :

- trafic redirigé vers lui (MITM)

---

## Network Traffic Sources & Flows

### 1️⃣ Sources de trafic

|Type|Exemple de périphériques|Caractéristiques / Protocoles|
|---|---|---|
|**Intermédiaires**|Firewalls, switches, web proxies, IDS/IPS, routeurs, AP, WLC|Peu de trafic généré. Protocoles : EIGRP, OSPF, BGP, SNMP, PING, SYSLOG, ARP, STP, DHCP|
|**Endpoints**|Serveurs, postes de travail, IoT, imprimantes, VM, ressources cloud, téléphones, tablettes|Génèrent la majorité du trafic. Origine et destination du flux principal|

---

### 2️⃣ Flows de trafic

|Type|Description|Exemples|
|---|---|---|
|**North-South (NS)**|Trafic entrant ou sortant du LAN via le firewall|HTTPS, DNS, SSH, VPN, SMTP, RDP|
|**East-West (EW)**|Trafic interne au LAN (ou LAN-cloud)|Active Directory, SMB, Print/File Services, Router/Infrastructure, App Comm, Backup/Replication, Monitoring/Management|

> ⚠️ EW souvent moins surveillé mais critique pour détecter les mouvements latéraux d’attaquants.

---

### 3️⃣ Exemples de flows concrets

### 🔹 HTTPS Flow (avec inspection TLS via web proxy)

1. Client → Web Proxy (proxy agit comme serveur)
2. Proxy → Web Server (nouvelle session TCP)
3. Proxy inspecte réponse → si safe, renvoie au client

> Résultat : 2 sessions TCP mais le client voit **une seule session avec le serveur**.

![[Network Traffic http flow.png]]

---

#### 🔹 External DNS Flow

1. Host → DNS interne (port 53)
2. DNS interne : check cache
3. Si réponse absente : DNS interne → routeur → firewall → DNS externe
4. Réponse → même chemin → DNS interne → Host

> Permet de suivre le flux DNS et d’identifier les requêtes anormales (exfiltration, tunneling).

![[Network Traffic external dns flow.png]]

---

#### 🔹 SMB avec Kerberos

1. Host connecté → authentification Kerberos avec Domain Controller
    - Obtention Ticket Granting Ticket (TGT)
2. Host demande un **Service Ticket** via TGT
3. Host → SMB Server → utilise service ticket
4. SMB session établie → accès share \FILESERVER\MARKETING

> Permet de comprendre comment le trafic interne est authentifié et sécurisé.

![[Network Traffic smb with kerberos.png]]

---

### 4️⃣ Points clés

- **Intermédiaire vs Endpoint** : qui génère le trafic et qui le relaye
- **North-South vs East-West** : direction critique pour surveillance
- **Flows typiques** : HTTPS, DNS, SMB/Kerberos
- **Importance** : surveiller EW pour détecter latéral movement / compromission interne

---

## How can we Observe Network Traffic ?

Network Traffic Analysis = combiner plusieurs sources pour **analyser, trouver des patterns et agir**.

---

### 1️⃣ Logs

**Exemples

```bash
# Auth log (Linux)  
Oct  8 11:20:15 web01 sshd[2145]: Accepted password for gensane from 192.168.1.50 port 52234 ssh2  
  
# Apache access log  
192.168.1.50 - - [08/Oct/2025:11:20:18 +0200] "GET /index.html HTTP/1.1" 200 2326 "-" "Mozilla/5.0"
```

**Points clés

- Logs = **première source d’information**
- Chaque vendor/log a sa propre norme
- Souvent seulement **IP source/destination, ports**, pas le **contenu complet**
- Protocoles standardisés pour logs : **Syslog**, **SNMP**

---

### 2️⃣ Full Packet Capture (FPC)

**Méthodes

|Méthode|Description|Exemple|
|---|---|---|
|**Network TAP**|Appareil physique en ligne → copie tout le trafic vers monitoring|Opère sur **link layer**, pas d’IP/MAC nécessaires|
|**Port Mirroring / SPAN**|Logiciel → copie des paquets d’un port vers un autre|Cisco : `monitor session 1 source interface fastEthernet0/1`|
_A network tap : is a physical device you place inline in your network._

_port morroring :_

![[Network Traffic port mirroring.png]]

- Peut aussi être fait sur **vSwitchs** ou cloud (ex : AWS VPC Traffic Mirroring)

**Best Practices

- **Placement** : là où le trafic important passe
- **Durée** : capture 1 Gbps → ~10,8 TB/jour
- **TAP vs Mirroring** : TAP = quasi zéro impact | Mirroring = peut ralentir si trafic élevé

**Outils d’analyse

- **Wireshark**, **TCPdump**, **Snort**, **Suricata**, **Zeek**

---

### 3️⃣ Network Statistics (metadata / flux)

- Comptage d’événements, ex : nombre de requêtes DNS par host
- Protocoles clés :
    - **NetFlow (Cisco)** : collecte métadonnées sur flux réseau → détecter C2, exfiltration, lateral movement
        
        Sample: srcIP=12.1.1.1 → dstIP=13.1.1.2
        
    - **IPFIX** : successeur de NetFlow, standard vendor-neutral, plus flexible

**Points clés

- Pas besoin de serveur dédié
- Beaucoup de NGFW, IPS, IDS incluent NetFlow/IPFIX
- Capture **métadonnées**, pas tous les paquets → moins de stockage requis

### Résumé

|Source|Exemple / Log|Info capturée|Usage|
|---|---|---|---|
|**Logs**|Auth, Apache|IP, port, statut|Surveillance rapide, corrélation|
|**Full Packet Capture**|TAP, SPAN|Contenu complet des paquets|Analyse détaillée, reconstruction, détection d’attaques|
|**Network Statistics**|NetFlow, IPFIX|Métadonnées de flux|Détection anomalies, C2, exfiltration|

---

