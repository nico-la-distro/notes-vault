## A Network - Overview

Un réseau = structure organisée d’actifs interconnectés → communication, partage de ressources, connectivité.

Comprendre composants + rôles = identifier activité suspecte.

### Network Components - Building Blocks of a Network

### Small Enterprise Network

### User Workstations (Endpoints)

Postes utilisateurs (PC, laptop).

Point d’entrée principal (phishing, malware).

Importance

- Foothold initial → mouvement latéral
- Moins monitorés
- Logs endpoint → processus malveillants
- Logs réseau → connexions C2

### File & Database Servers

Stockage des données critiques.

- File server → documents partagés
- DB server → données structurées (clients, RH, finance)

Importance

- Cible principale (data sensible)
- Ransomware → file servers
- Exfiltration → extraction discrète

### Application Servers (Web, Email, VPN, etc.)

- Web → apps/sites
- Email → communication
- VPN → accès distant sécurisé

Importance

- Exposés Internet → surface d’attaque élevée
- Scan constant (vulnérabilités, mauvaise config)
- Foothold interne possible

Surveillance

- Logs applicatifs
- Firewall / IDS

Détection

- Exploits (ex: SQLi)
- Brute-force (email/VPN)
- IP externes suspectes

### Active Directory (AD) / Authentication Servers

Gestion identités + accès (users, groupes, machines).

Importance

- Contrôle total du réseau
- Cible pour :
    - Privilege escalation
    - Persistence
    - Lateral movement
- Compromission Domain Admin = compromission totale

Surveillance

- Logs authentification

Détection

- Échecs répétés (password spraying)
- Logins anormaux (heure/IP)
- Accès inhabituels

### Routers & Switches (Network Infrastructure)

- Router → interconnexion réseaux / Internet
- Switch → communication interne

Importance

- Si compromis :
    - MITM (interception/modification trafic)
    - Reroutage (backdoor)
    - Canaux cachés Internet

### Firewalls / Perimeter Devices

Contrôle trafic interne ↔ Internet.

Fonctions

- Filtrage (rules)
- Inspection avancée
- IPS / détection malware

Importance

- Protection périmètre
- Bloque accès non autorisés (DB, RDP)
- Logs = premiers indicateurs d’attaque

Détection

- Port scans
- Brute-force
- Exploitation attempts

---

## Network Visibility

Visibilité réseau = capacité à surveiller/comprendre activité réseau.

Principe clé :

> You can't defend what you can't see.

Objectifs

- Threat detection
- Incident investigation
- Threat hunting
- Compliance

Sources principales

- Host-centric logs
- Network-centric logs

Corrélation des deux = timeline d’attaque complète.

### Why is Visibility Crucial?

Sans visibilité :

- Malware invisible
- Accès non autorisés invisibles
- Exfiltration invisible

Permet de :

- Détecter anomalies
- Investiguer incidents
- Chasser menaces
- Assurer conformité

Logs = enregistrements des événements réseau/système.

### Host-Centric Logs

Logs générés par les machines :

- Serveurs
- Workstations
- Laptops

Vue locale détaillée d’un host.

Utilité

- Comprendre impact direct d’une attaque

### Key Host-Centric Log Sources

|Source|Contenu|
|---|---|
|OS Logs|Logons, process, services, failed logins|
|Application Logs|Apache, Nginx, MySQL, MSSQL, apps|
|Security Tool Logs|AV, EDR, HIDS|

### Network-Centric Logs

Logs générés par équipements réseau.

Vue trafic entre machines.

Contenu

- Source/Destination IP
- Ports
- Protocoles
- Allowed/Blocked

Utilité

- Reconnaissance
- Lateral movement
- Exfiltration

Analogie

- Host logs = activité dans une pièce
- Network logs = entrées/sorties du bâtiment

### Key Network-Centric Log Sources

|Source|Utilité|
|---|---|
|Firewalls|Connexions autorisées/bloquées|
|IDS/IPS|Détection signatures/anomalies|
|Routers/Switches|Flow data (qui parle à qui)|
|Web Proxies|Historique web utilisateur|
|VPN|Connexions distantes|

### Importance of Network-Centric Logs

Détection :

- Unauthorized access
- Active attacks
- Data exfiltration
- Suspicious remote access

Combiner host + network logs = vision complète + timeline précise.

---

## Network Perimeter

Périmètre réseau = frontière entre :

- Internal network (trusted)
- Internet (untrusted)

Point d’entrée/sortie de tout le trafic.

= première ligne de défense.

- Internal network → systèmes critiques
- External Internet → menaces potentielles
- Perimeter → point de contrôle du trafic

Comprendre le périmètre = essentiel pour analyste sécurité.

### The Perimeter

Défini par :

- Hardware edge devices
- Virtual gateways
- Cloud connections
- Remote access points

#### Common components of a network perimeter include:

|Composant|Rôle|
|---|---|
|Firewalls|Filtrage trafic|
|Routers/Gateways|Routage + règles accès|
|DMZ|Zone tampon services publics|
|VPN Gateways|Accès distant sécurisé|

### Importance of Network Perimeter

Le périmètre :

- Contrôle trafic entrant/sortant
- Protège assets internes
- Premier point observé par SOC

Si faible/mal configuré :

- Exploitation services exposés (RDP, MySQL, SMB)
- Scanning / reconnaissance
- Brute-force
- Exfiltration données

### Network Perimeter in a Small Enterprise

Architecture typique :

- Firewall entre Internet et LAN
- Web server dans DMZ
- AD/File/DB derrière firewall
- Accès externe via VPN

Objectif :

- Isolation services publics
- Contrôle accès interne

#### Commonly includes:

|Composant|Fonction|
|---|---|
|Routers|Direction trafic|
|Firewalls|Inspection/filtrage|
|DMZ|Hébergement services publics|
|VPN Gateways|Accès distant sécurisé|
### Role of a Security Analyst

Surveillance :

- Firewall logs
- Allowed/blocked connections
- Scanning attempts
- Brute-force
- Outbound traffic anormal
- Beaconing / exfiltration

Comprendre :

- Ce qui doit être exposé
- Ce qui ne doit pas l’être

---

# Network Perimeters: Monitoring and Protecting

Monitoring du périmètre = inspection + limitation exposition via :

- Firewalls
- IDS/IPS
- Access control

Objectifs :

- Détecter attaques précoces (scan, brute-force)
- Identifier mauvaises configurations
- Repérer trafic sortant suspect (exfiltration)

Logs périmètre : `/Perimeter_logs/task6/`

---
## Monitoring the Perimeter in Action

### Scenario 1: Probing for Ports (Port Scanning)

Firewall Log

```
2025-09-22 08:30:04 ALLOW TCP 198.51.100.45:49876 -> 10.0.0.51:80
2025-09-22 08:30:05 BLOCK TCP 203.0.113.10:50001 -> 10.0.0.20:21
2025-09-22 08:30:06 BLOCK TCP 203.0.113.10:50002 -> 10.0.0.20:22
2025-09-22 08:30:07 ALLOW TCP 192.0.2.115:51235 -> 10.0.0.50:443
2025-09-22 08:30:08 BLOCK TCP 203.0.113.10:50003 -> 10.0.0.20:23
2025-09-22 08:30:09 BLOCK TCP 203.0.113.10:50004 -> 10.0.0.20:25
2025-09-22 08:30:10 ALLOW TCP 198.51.100.92:51111 -> 10.0.0.50:443
2025-09-22 08:30:11 BLOCK TCP 203.0.113.10:50005 -> 10.0.0.20:53
```

Log Breakdown

- IP unique (203.0.113.10) teste plusieurs ports rapidement
- Même cible interne, ports différents

Verdict : port scan (reconnaissance services ouverts)

### Scenario 2: Attacking the Web Server (SQL Injection)

WAF Logs

```
timestamp=2025-09-22T09:14:44Z src_ip=192.0.2.130 action=ALLOW request="GET /index.html"
timestamp=2025-09-22T09:14:45Z src_ip=198.51.100.92 action=ALLOW request="GET /products.php?id=9"
timestamp=2025-09-22T09:14:46Z src_ip=[REDACTED] action=BLOCK request="GET /search.php?q=<script>alert('XSS')</script>" rule_id=941100 attack_type="XSS"
timestamp=2025-09-22T09:14:47Z src_ip=192.0.2.140 action=ALLOW request="GET /css/style.css"
timestamp=2025-09-22T09:15:42Z src_ip=[REDACTED] action=BLOCK request="GET /../../../../etc/passwd" rule_id=930120 attack_type="Directory Traversal"
```

Log Breakdown

- Filtrer `action=BLOCK` → identification attaques
- `XSS` → injection script
- `Directory Traversal` → accès fichiers sensibles
- IDS/WAF précise le type d’attaque

Verdict : attaques web actives multi-vector (haute confiance)

### Scenario 3: Guessing the Password (VPN Brute-Force)

VPN Gateway Log

```
2025-09-22 10:12:11 FAILED_AUTH TCP [REDACTED]:31245 -> 10.0.0.1:443 (user 'admin')
2025-09-22 10:12:15 FAILED_AUTH TCP [REDACTED]:31248 -> 10.0.0.1:443 (user 'admin')
2025-09-22 10:12:21 SUCCESS_AUTH TCP 198.51.100.88:41233 -> 10.0.0.1:443 (user 'b.jones')
2025-09-22 10:12:08 FAILED_AUTH TCP [REDACTED]:31249 -> 10.0.0.1:443 (user 'guest')
2025-09-22 10:12:09 FAILED_AUTH TCP [REDACTED]:31250 -> 10.0.0.1:443 (user 'user')
```

Log Breakdown

- Volume élevé `FAILED_AUTH`
- Analyse par source IP → activité concentrée
- Tentatives multiples sur comptes communs

Verdict : brute-force VPN

### Key Takeaways

- Distinguer trafic normal vs suspect
- Patterns clés :
    - One-to-many → scanning
    - One-to-one répétitif → brute-force
    - Intervalles réguliers → malware beaconing
- IDS context > simple firewall block
- Corrélation = clé de détection
- Monitoring périmètre = première défense sécurité

---

## Incident Scenario

Initech Corp, a mid-sized financial services company, has recently deployed a new firewall and intrusion detection system (IDS) to monitor its network perimeter. Over the past month, security analysts have noticed abnormal traffic patterns, but the SOC team has been overwhelmed and missed deeper analysis.

As a new security analyst, you have been tasked with reviewing one month of perimeter logs to determine what techniques the adversary used, and whether they succeeded in breaching the perimeter.

You have been given three sets of logs from the time of the incident. The logs can be found in the `Perimeter_logs/challenge` directory on the Desktop.

- **Firewall Logs:**`firewall.log`
- **WAF Logs:**`ids_alerts.log` 
- **VPN Logs:**`vpn_auth.log` 

![](https://tryhackme-images.s3.amazonaws.com/user-uploads/5e8dd9a4a45e18443162feab/room-content/5e8dd9a4a45e18443162feab-1758693781878.png)

### Investigating the Logs

There are two ways to investigate the logs: manually using command-line tools or using Splunk. Instructions on how to access the Splunk instance are mentioned at the end.

#### Method 1: Manual Log Analysis

Let's start by exploring the logs, as shown below:

**Commandline:** `head firewall.log`

```bash
ubuntu@tryhackme:~/Desktop/Perimeter_logs/challenge$ head firewall.log 
2025-08-25 00:47:46 ALLOW TCP [REDACTED]:60317 -> 10.0.0.50:443
2025-08-25 01:29:33 ALLOW TCP 203.0.113.100:62718 -> [REDACTED]:443
2025-08-25 01:42:12 ALLOW TCP 203.0.113.100:55875 -> [REDACTED]:80
2025-08-25 03:30:47 ALLOW TCP [REDACTED]:63035 -> [REDACTED]:80
2025-08-25 04:06:58 ALLOW TCP 192.0.2.115:65458 -> [REDACTED]:25
2025-08-25 05:51:36 ALLOW TCP 203.0.113.100:56035 -> [REDACTED]:53
2025-08-25 06:09:50 ALLOW TCP 198.51.100.92:63418 -> [REDACTED]:8080
2025-08-25 07:39:29 ALLOW TCP [REDACTED]:55955 -> [REDACTED]:8080
2025-08-25 08:24:34 ALLOW TCP 198.51.100.92:63475 -> [REDACTED]:8080
2025-08-25 08:57:21 ALLOW TCP 198.51.100.92:58636 -> 10.0.0.50:53
```

Explore ids logs, as shown below:

**Commandline:** `head ids_alerts.log`

```bash
ubuntu@tryhackme:~/Desktop/Perimeter_logs/challenge$ head ids_alerts.log 
2025-08-25 00:12:53 [**] [1:2003272:1] ET POLICY Suspicious HTTP [**] [Classification: Suspicious Activity] [Priority: 3] {TCP} 198.51.100.92:20127 -> [REDACTED]:22
2025-08-25 01:50:30 [**] [1:2003377:1] ET POLICY Suspicious HTTP [**] [Classification: Suspicious Activity] [Priority: 1] {TCP} 203.0.113.100:56603 -> [REDACTED]:25
2025-08-25 02:16:39 [**] [1:2003437:1] ET INFO Possible Benign Scan [**] [Classification: Suspicious Activity] [Priority: 3] {TCP} [REDACTED]:62546 -> [REDACTED]:21
2025-08-25 02:23:07 [**] [1:2003344:1] ET WEB_SERVER Possible SQL Injection [**] [Classification: Suspicious Activity] [Priority: 2] {TCP} 198.51.100.45:12396 -> [REDACTED]:22
2025-08-25 02:25:48 [**] [1:2003445:1] ET POLICY Suspicious HTTP [**] [Classification: Suspicious Activity] [Priority: 3] {TCP} 192.0.2.115:3952 -> [REDACTED]:22
2025-08-25 03:35:00 [**] [1:2003160:1] ET INFO Possible Benign Scan [**] [Classification: Suspicious Activity] [Priority: 1] {TCP} [REDACTED]:38760 -> [REDACTED]:443
2025-08-25 05:02:36 [**] [1:2003187:1] ET WEB_SERVER Possible SQL Injection [**] [Classification: Suspicious Activity] [Priority: 1] {TCP} 198.51.100.92:46776 -> [REDACTED]:3389
2025-08-25 06:04:26 [**] [1:2003179:1] ET INFO Possible Benign Scan [**] [Classification: Suspicious Activity] [Priority: 2] {TCP} 198.51.100.92:20632 -> 10.0.0.50:8080
2025-08-25 14:12:11 [**] [1:2003500:1] ET INFO Possible Benign Scan [**] [Classification: Suspicious Activity] [Priority: 2] {TCP} 192.0.2.115:30225 -> [REDACTED]:445
2025-08-25 15:30:03 [**] [1:2003354:1] ET POLICY Suspicious HTTP [**] [Classification: Suspicious Activity] [Priority: 3] {TCP} 203.0.113.100:27572 -> [REDACTED]:4444
```

Examine VPN Logs, as shown below:

**Commandline:** `head vpn_auth.log`

```bash
ubuntu@tryhackme:~/Desktop/Perimeter_logs/challenge$  head vpn_auth.log 
2025-08-25 08:25:10 [REDACTED] alice SUCCESS assigned_ip=10.8.0.143
2025-08-25 08:27:38 203.0.113.100 svc_[REDACTED] SUCCESS assigned_ip=10.8.0.131
2025-08-25 14:57:10 203.0.113.10 svc_[REDACTED] SUCCESS assigned_ip=10.8.0.116
2025-08-25 23:04:53 203.0.113.10 jsmith SUCCESS assigned_ip=10.8.0.31
2025-08-26 03:36:17 198.51.100.92 svc_[REDACTED] SUCCESS assigned_ip=10.8.0.62
2025-08-26 08:55:14 [REDACTED] bob SUCCESS assigned_ip=10.8.0.126
2025-08-26 10:02:45 198.51.100.92 svc_[REDACTED] SUCCESS assigned_ip=10.8.0.81
2025-08-27 03:11:33 198.51.100.45 bob SUCCESS assigned_ip=10.8.0.163
2025-08-28 02:52:16 192.0.2.115 alice SUCCESS assigned_ip=10.8.0.132
2025-08-28 03:20:33 [REDACTED] svc_[REDACTED] SUCCESS assigned_ip=10.8.0.193
```

### Reconnaissance attempt

Let's begin our analysis by analyzing the blocked requests in the firewall logs, as shown below:

**Commandline:** `cat firewall.log | grep "BLOCK" | head`

```bash
ubuntu@tryhackme:~/Desktop/Perimeter_logs/challenge$ cat firewall.log | grep "BLOCK" | head
2025-08-26 12:12:47 BLOCK TCP 203.0.113.10:64292 -> 10.0.0.50:21
2025-08-27 03:18:28 BLOCK TCP [REDACTED]:61701 -> [REDACTED]:23
2025-08-27 11:56:20 BLOCK TCP 203.0.113.10:64952 -> 10.0.0.50:22
2025-08-27 22:52:00 BLOCK TCP 203.0.113.10:63686 -> [REDACTED]:445
2025-08-28 10:00:00 BLOCK TCP [REDACTED]:50000 -> [REDACTED]:4444
2025-08-28 10:02:30 BLOCK TCP [REDACTED]:50005 -> [REDACTED]:22
```

Examining the blocked requests indicates that an external IP has been found probing against internal IPs using various ports (22,23,21,445,3389).

Let's use the following filtering to understand which IP is responsible for the maximum BLOCK requests.

**Commandline:** `cat firewall.log | grep "BLOCK" | cut -d' ' -f5 | cut -d: -f1 | sort -nr | uniq -c`

```bash
ubuntu@tryhackme:~/Desktop/Perimeter_logs/challenge$ cat firewall.log | grep "BLOCK" | cut -d' ' -f5 | cut -d: -f1 | sort -nr | uniq -c
279 [REDACTED]
46 203.0.113.10
26 [REDACTED]
```

We have identified a suspicious IP, which we can use to pivot, examine other log files, and understand the correlation.

Let's use the following query to find if our firewall has allowed any requests to the suspicious IP, as shown below:

**Commandline:** `cat firewall.log | grep [REDACTED] | grep "ALLOW"`

```bash
ubuntu@tryhackme:~/Desktop/Perimeter_logs/challenge$ cat firewall.log | grep [REDACTED] | grep "ALLOW"
2025-08-26 00:17:58 ALLOW TCP [REDACTED]:61009 -> [REDACTED]:4444
2025-08-26 22:04:34 ALLOW TCP [REDACTED]:55996 -> 10.0.0.50:445
2025-08-27 21:04:23 ALLOW TCP [REDACTED]:53944 -> 10.0.0.50:22
2025-08-28 15:50:50 ALLOW TCP [REDACTED]:56123 -> [REDACTED]:3389
2025-08-30 20:26:23 ALLOW TCP [REDACTED]:61685 -> [REDACTED]:4444
2025-09-02 09:25:06 ALLOW TCP [REDACTED]:50550 -> 10.0.0.50:22  -----------
2025-09-22 15:46:02 ALLOW TCP [REDACTED]:59771 -> [REDACTED]:23
2025-09-22 17:22:11 ALLOW TCP [REDACTED]:49360 -> 10.0.0.50:22
```

It seems that the attacker was able to gain access to the internal network through exploitation. Let's examine the VPN logs and see if we can find further footprints of the attack.

### VPN Brute-force / Credential Access

In the VPN Logs, let's first examine the number of failed attempts, using the following command, as shown below:

**Commandline:** `cat vpn_auth.log | grep FAIL | cut -d' ' -f3 | sort -nr | uniq -c`

```bash
ubuntu@tryhackme:~/Desktop/Perimeter_logs/challenge$ cat vpn_auth.log | grep FAIL | cut -d' ' -f3 | sort -nr | uniq -c
118 [REDACTED]
1 203.0.113.100
1 198.51.100.92
1 198.51.100.45
```

We can clearly see that the suspicious IP has multiple failed VPN login attempts. Let's now use the following command to narrow down on the suspicious IP, as shown below:

**Commandline:** `cat vpn_auth.log | grep [REDACTED]`

```bash
ubuntu@tryhackme:~/Desktop/Perimeter_logs/challenge$ cat vpn_auth.log | grep [REDACTED]
2025-09-03 02:19:00 [REDACTED] svc_[REDACTED] FAIL
2025-09-03 02:19:10 [REDACTED] svc_[REDACTED] FAIL
2025-09-03 02:19:20 [REDACTED] svc_[REDACTED] FAIL
2025-09-03 02:19:30 [REDACTED] svc_[REDACTED] FAIL
--------
-----------
2025-09-03 02:19:40 [REDACTED] svc_[REDACTED] SUCCESS assigned_ip=[REDACTED]
2025-09-03 02:19:50 [REDACTED] svc_[REDACTED] SUCCESS assigned_ip=[REDACTED]
2025-09-04 16:45:24 [REDACTED] svc_[REDACTED] SUCCESS assigned_ip=10.8.0.181
2025-09-05 13:21:52 [REDACTED] jsmith SUCCESS assigned_ip=10.8.0.94
2025-09-09 17:54:00 [REDACTED] jsmith SUCCESS assigned_ip=10.8.0.187
2025-09-09 19:15:51 [REDACTED] jsmith SUCCESS assigned_ip=10.8.0.134
2025-09-10 12:24:20 [REDACTED] bob SUCCESS assigned_ip=10.8.0.39
```

Above result shows, that, the multiple login attempt was made against a certain user `svc_REDACTED`, followed by a success login, resulting in the attacker being assigned a local IP address. We will pick the first assigned IP and extend our analysis to look for traces in the log files.

### Lateral Movement

By now, it is confirmed that, that attacker has successfully gained the initial access and got hold on to an internal IP address. Let's filter through the firewall logs and see if we can find the footprints of any lateral movement from the compromised host IP `REDACTED`.

**Commandline:** `cat firewall.log | grep [REDACTED] | grep "ALLOW" | head`  

```bash
ubuntu@tryhackme:~/Desktop/Perimeter_logs/challenge$ cat firewall.log | grep [REDACTED] | grep "ALLOW" | head 
2025-09-05 06:00:00 ALLOW TCP [REDACTED]:2000 -> [REDACTED]:22
2025-09-05 06:10:00 ALLOW TCP [REDACTED]:2001 -> [REDACTED]:445
2025-09-05 06:20:00 ALLOW TCP [REDACTED]:2002 -> [REDACTED]:22
2025-09-05 06:40:00 ALLOW TCP [REDACTED]:2004 -> [REDACTED]:3389
2025-09-05 07:30:00 ALLOW TCP [REDACTED]:2009 -> [REDACTED]:22
2025-09-05 08:00:00 ALLOW TCP [REDACTED]:2012 -> [REDACTED]:22
```

It is observed that, the compromised IP is probing internal machines `10.0.0.20`, `10.0.0.51`, `10.0.0.60` on various ports on three main ports `SMB/RDP/SSH (445/3389/22)`. 

Let's now pivot to `ids_alerts` logs, and filter through the compromised IP and see if we can find any intrusion rules triggered, as shown below:

**Commandline:** `cat ids_alerts.log | grep [REDACTED] | head`

```bash
ubuntu@tryhackme:~/Desktop/Perimeter_logs/challenge$ cat ids_alerts.log | grep [REDACTED] | head    
2025-09-05 06:00:00 [**] [1:2000200:1] ET SCAN Possible SSH Scan [**] [Classification: Attempted Unauthorized Access] [Priority: 1] {TCP} [REDACTED]:2000 -> [REDACTED]:22
2025-09-05 06:10:00 [**] [1:2000201:1] ET EXPLOIT Possible MS-SMB Lateral Movement [**] [Classification: Attempted Unauthorized Access] [Priority: 1] {TCP} [REDACTED]:2001 -> [REDACTED]:445
2025-09-05 06:20:00 [**] [1:2000202:1] ET SCAN Possible SSH Scan [**] [Classification: Attempted Unauthorized Access] [Priority: 1] {TCP} [REDACTED]:2002 -> [REDACTED]:22
2025-09-05 06:30:00 [**] [1:2000203:1] ET EXPLOIT Possible RDP Brute Force [**] [Classification: Attempted Unauthorized Access] [Priority: 1] {TCP} [REDACTED]:2003 -> [REDACTED]:3389
2025-09-05 07:10:00 [**] [1:2000207:1] ET SCAN Possible SSH Scan [**] [Classification: Attempted Unauthorized Access] [Priority: 1] {TCP} [REDACTED]:2007 -> [REDACTED]:22
2025-09-05 07:20:00 [**] [1:2000208:1] ET EXPLOIT Possible RDP Brute Force [**] [Classification: Attempted Unauthorized Access] [Priority: 1] {TCP} [REDACTED]:2008 -> [REDACTED]:3389
2025-09-05 07:30:00 [**] [1:2000209:1] ET SCAN Possible SSH Scan [**] [Classification: Attempted Unauthorized Access] [Priority: 1] {TCP} [REDACTED]:2009 -> [REDACTED]:22
```

It seems, the compromised host is trying to exploit various vulnerabilities against the services mentioned above on those hosts. One of the IDS alerts indicates SMB exploit, which look interesting. Let's narrow down our search by using the following search query:

**Commandline:** `cat ids_alerts.log | grep -n [REDACTED] | grep 'SMB' | cut -d' ' -f6,7,8,9,10,19,21 | head`

```bash
ubuntu@tryhackme:~/Desktop/Perimeter_logs/challenge$ cat ids_alerts.log | grep -n [REDACTED] | grep 'SMB' | cut -d' ' -f6,7,8,9,10,19,21 | head
EXPLOIT Possible MS-SMB Lateral Movement [REDACTED]:2001 [REDACTED]:445
EXPLOIT Possible MS-SMB Lateral Movement [REDACTED]:2006 [REDACTED]:445
EXPLOIT Possible MS-SMB Lateral Movement [REDACTED]:2010 [REDACTED]:445
EXPLOIT Possible MS-SMB Lateral Movement [REDACTED]:2016 [REDACTED]:445
EXPLOIT Possible MS-SMB Lateral Movement [REDACTED]:2033 [REDACTED]:445
EXPLOIT Possible MS-SMB Lateral Movement [REDACTED]:2035 [REDACTED]:445
```

Above results confirm that, the compromised host was able to exploit SMB service and was able to achieve lateral movement.

### C2 Beaconing

Now that, we have an evidence of the lateral movement of the attacker. Let's hunt for any indicator of C2 communication. If we look at the IDS alerts, we can find a specific alert related to C2 Beaconing, indicating possible C2 communication. Let's use the following search query to see the results:

**Commandline:** `cat ids_alerts.log | grep C2 | head`

```bash
ubuntu@tryhackme:~/Desktop/Perimeter_logs/challenge$ cat ids_alerts.log | grep C2 | head
2025-09-11 01:00:00 [**] [1:2001000:1] ET TROJAN Possible C2 Beaconing [**] [Classification: A network Trojan was detected] [Priority: 1] {TCP} [REDACTED]:30000 -> [REDACTED]:4444
2025-09-11 07:00:00 [**] [1:2001001:1] ET TROJAN Possible C2 Beaconing [**] [Classification: A network Trojan was detected] [Priority: 1] {TCP} [REDACTED]:30001 -> [REDACTED]:4444
2025-09-11 13:00:00 [**] [1:2001002:1] ET TROJAN Possible C2 Beaconing [**] [Classification: A network Trojan was detected] [Priority: 1] {TCP} [REDACTED]:30002 -> [REDACTED]:4444
2025-09-12 13:00:00 [**] [1:2001006:1] ET TROJAN Possible C2 Beaconing [**] [Classification: A network Trojan was detected] [Priority: 1] {TCP} [REDACTED]:30006 -> [REDACTED]:4444
2025-09-12 19:00:00 [**] [1:2001007:1] ET TROJAN Possible C2 Beaconing [**] [Classification: A network Trojan was detected] [Priority: 1] {TCP} [REDACTED]:30007 -> [REDACTED]:4444
2025-09-13 01:00:00 [**] [1:2001008:1] ET TROJAN Possible C2 Beaconing [**] [Classification: A network Trojan was detected] [Priority: 1] {TCP} [REDACTED]:30008 -> [REDACTED]:4444
2025-09-13 07:00:00 [**] [1:2001009:1] ET TROJAN Possible C2 Beaconing [**] [Classification: A network Trojan was detected] [Priority: 1] {TCP} [REDACTED]:30009 -> [REDACTED]:4444
```

It clearly confims our suspicion against one of the internal compromised host.

Let's slice and dice through the results, to filter against the compromised IP responsible for C2 beaconing, as shown below:

**Commandline:** `cat ids_alerts.log | grep -n [REDACTED]   | cut -d' ' -f6,7,8,9,10,19,22,23 | head -n 15`

```bash
ubuntu@tryhackme:~/Desktop/Perimeter_logs/challenge$ cat ids_alerts.log | grep -n [REDACTED]   | cut -d' ' -f6,7,8,9,10,19,22,23 | head -n 15
POLICY Suspicious HTTP [**] [Classification:
WEB_SERVER Possible SQL Injection [**] [REDACTED]:3389
POLICY Suspicious HTTP [**] [Classification:
WEB_SERVER Possible SQL Injection [**] [REDACTED]:25
POLICY Suspicious HTTP [**] [Classification:
POLICY Suspicious HTTP [**] [Classification:
POLICY Suspicious HTTP [**] [Classification:
POLICY Suspicious HTTP [**] [Classification:
INFO Possible Benign Scan [**] [REDACTED]:443
WEB_SERVER Possible SQL Injection [**] [REDACTED]:53
```

This further affirms that, the infected host is further performing susicious activities. We can use the following command to show the stats of the alerts triggered against infected host.

**Commandline:** `cat ids_alerts.log | grep -n [REDACTED] | cut -d' ' -f6,7,8,9,10,19,22,23 | uniq -c | sort -nr | head` 

```bash
ubuntu@tryhackme:~/Desktop/Perimeter_logs/challenge$ cat ids_alerts.log | grep -n [REDACTED] | cut -d' ' -f6,7,8,9,10,19,22,23 | uniq -c | sort -nr | head 
32 TROJAN Possible C2 Beaconing [**] {TCP} [REDACTED]:4444
5 POLICY Suspicious HTTP [**] [Classification:
3 TROJAN Possible C2 Beaconing [**] {TCP} [REDACTED]:4444
3 POLICY Suspicious HTTP [**] [Classification:
2 WEB_SERVER Possible SQL Injection [**] [REDACTED]:4444
2 TROJAN Possible C2 Beaconing [**] {TCP} [REDACTED]:4444
```

Above analysis clearly indicates that our internal network is fully compromised, and we now have the external IP address acting as a C2 server, recieving the C2 beacons from our compromised host.

**Commandline:** `cat ids_alerts.log | grep -n [REDACTED]   | cut -d' ' -f6,7,8,9,10,19,22,23 | uniq -c | sort -nr | head` 

```bash
ubuntu@tryhackme:~/Desktop/Perimeter_logs/challenge$ cat ids_alerts.log | grep -n [REDACTED]  | cut -d' ' -f6,7,8,9,10,19,22,23 | sort -nr | uniq -c | sort -nr       
80 TROJAN Possible C2 Beaconing [**] {TCP} [REDACTED]:4444
32 INFO Possible HTTP POST Large {TCP} [REDACTED]:80
28 INFO Possible HTTP POST Large {TCP} [REDACTED]:8080
23 POLICY Suspicious HTTP [**] [Classification:
2 WEB_SERVER Possible SQL Injection [**] 10.0.0.50:53
2 WEB_SERVER Possible SQL Injection [**] 10.0.0.50:3389
2 WEB_SERVER Possible SQL Injection [**] [REDACTED]:8080
2 WEB_SERVER Possible SQL Injection [**] [REDACTED]:445
2 INFO Possible Benign Scan [**] 10.0.0.50:21
1 WEB_SERVER Possible SQL Injection [**] [REDACTED]:53
1 WEB_SERVER Possible SQL Injection [**] [REDACTED]:443
```

### Data Exfiltration Attempt

Now that, we have identified the C2 communication and examined other alerts as well against suspicious Is, let's now investigate, if there are any indicators of data being exfiltrated out of our network. We will apply filter on the compromised hosts, and examine the traffic originating from those to an external destination IP, as shown below:

**Commandline:** `cat firewall.log | grep [REDACTED]  | cut -d' ' -f5,6,7  |   uniq  | sort` 

```bash
ubuntu@tryhackme:~/Desktop/Perimeter_logs/challenge$ cat firewall.log | grep [REDACTED]  | cut -d' ' -f5,6,7 | uniq | sort 
[REDACTED]:40000 -> [REDACTED]:8080
[REDACTED]:40001 -> [REDACTED]:8080
[REDACTED]:40002 -> [REDACTED]:8080
[REDACTED]:40003 -> [REDACTED]:80
[REDACTED]:40004 -> [REDACTED]:80
[REDACTED]:40005 -> [REDACTED]:80
[REDACTED]:40006 -> [REDACTED]:80
[REDACTED]:40007 -> [REDACTED]:80
[REDACTED]:40008 -> [REDACTED]:80
[REDACTED]:40009 -> [REDACTED]:8080
[REDACTED]:40010 -> [REDACTED]:80
[REDACTED]:40015 -> [REDACTED]:80
[REDACTED]:40016 -> [REDACTED]:80
[REDACTED]:40017 -> [REDACTED]:8080
[REDACTED]:40018 -> [REDACTED]:80
[REDACTED]:40019 -> [REDACTED]:8080
```

The output clearly shows, the compromised host `REDACTED` is sending extensive amound of traffic on external IP address. We can also filter on IDS logs, to see the alerts being triggered on these activities from the internal IP.

```bash
ubuntu@tryhackme:~/Desktop/Perimeter_logs/challenge$ cat ids_alerts.log | grep [REDACTED] | tail
2025-09-27 07:00:00 [**] [1:2002050:1] ET INFO Possible HTTP POST Large Upload [**] [Classification: Potential Data Exfiltration] [Priority: 2] {TCP} [REDACTED]:40050 -> [REDACTED]:8080
2025-09-27 11:00:00 [**] [1:2002051:1] ET INFO Possible HTTP POST Large Upload [**] [Classification: Potential Data Exfiltration] [Priority: 2] {TCP} [REDACTED]:40051 -> [REDACTED]:8080
2025-09-27 15:00:00 [**] [1:2002052:1] ET INFO Possible HTTP POST Large Upload [**] [Classification: Potential Data Exfiltration] [Priority: 2] {TCP} [REDACTED]:40052 -> [REDACTED]:80802025-09-28 07:00:00 [**] [1:2002056:1] ET INFO Possible HTTP POST Large Upload [**] [Classification: Potential Data Exfiltration] [Priority: 2] {TCP} [REDACTED]:40056 -> [REDACTED]:80
2025-09-28 11:00:00 [**] [1:2002057:1] ET INFO Possible HTTP POST Large Upload [**] [Classification: Potential Data Exfiltration] [Priority: 2] {TCP} [REDACTED]:40057 -> [REDACTED]:8080
2025-09-28 15:00:00 [**] [1:2002058:1] ET INFO Possible HTTP POST Large Upload [**] [Classification: Potential Data Exfiltration] [Priority: 2] {TCP} [REDACTED]:40058 -> [REDACTED]:80
2025-09-28 19:00:00 [**] [1:2002059:1] ET INFO Possible HTTP POST Large Upload [**] [Classification: Potential Data Exfiltration] [Priority: 2] {TCP} [REDACTED]:40059 -> [REDACTED]:8080
```

We have found evidence of data exfiltration attempts. If we dig deep into the ids alerts and pivot correlate the alerts through other log files, we can find more suspicious activities as well.

### Method 2: Analyzing Logs via Splunk

As a SOC Analyst, analyzing logs manually can become a tedious task if the log files are large. Therefore, a Splunk instance is also provided in the VM if you decide to use it for log Analysis. 

To proceed, open the link `localhost:8000` in the browser, click on the `Search & Reporting` tab on the left bar, and start analyzing the logs. Logs are pre-ingested into the `index="network_logs"`, as shown below:

![](https://tryhackme-images.s3.amazonaws.com/user-uploads/5e8dd9a4a45e18443162feab/room-content/5e8dd9a4a45e18443162feab-1758712688060.png)

