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

