## Introduction

### Nmap

- Créé par Gordon Lyon (Fyodor), open-source, licence GPL
- Usages : mapping réseau, identification d'hôtes, découverte de services, scripting (fingerprinting, exploitation)

### Learning Objectives

Deux questions cibles :

1. Quels systèmes sont up ?
2. Quels services tournent ?

Cette room -> question 1 (host discovery). Les suivantes -> question 2 (port scanning).

Points clés :

- Host discovery avant port scan = évite le bruit et le gaspillage de temps sur hosts offline
- ARP scan -> link layer, broadcast, hosts sur le même subnet
- ICMP (echo, timestamp, address mask) -> hosts sur segments différents
- TCP SYN/ACK ping -> transport layer, comportement différent selon privilèges
- UDP ping -> déclenche réponse ICMP port-unreachable si port fermé = host online

---

## Subnetworks

### Concepts clés

- **Network segment** -> groupe de machines reliées par un medium physique (switch Ethernet, Wi-Fi AP)
- **Subnet** -> connexion logique, range IP propre, reliée au réseau via un router
- Un firewall peut exister entre segments pour enforcer des policies

|Notation|Masque|Hosts max|
|---|---|---|
|`/16`|`255.255.0.0`|~65 000|
|`/24`|`255.255.255.0`|~250|

-> Référence subnetting : [Intro to LAN - Task 2](https://tryhackme.com/room/introtolan)

### ARP et limites

- Sur le **même subnet** -> scanner utilise ARP pour découvrir les hosts live (récupère MAC = host online)
- Sur un **subnet différent** -> paquets routés via default gateway, mais **ARP n'est pas routable**
- ARP = protocole link-layer -> limité à son subnet, ne traverse pas le router

---

## Understanding Hosts Discovery through TCP/IP Layer

### Protocoles utilisables par couche

|Couche|Protocole|
|---|---|
|Link|ARP|
|Network|ICMP|
|Transport|TCP / UDP|

### ARP

- Envoie une frame en broadcast sur le segment
- Demande à la machine avec une IP cible de répondre avec son MAC
- Usage unique : résolution IP -> MAC

### ICMP

- Nombreux types -> [référence IANA](https://www.iana.org/assignments/icmp-parameters/icmp-parameters.xhtml)
- Ping = Type 8 (Echo) + Type 0 (Echo Reply)
- Sur le même subnet : ARP query précède l'ICMP Echo

### TCP / UDP

- Scanner envoie un paquet crafté sur des ports TCP/UDP communs
- Vérifie si la cible répond
- Utile quand ICMP Echo est bloqué

---

## Enumerating Targets

### Spécification des cibles

|Format|Exemple|Résultat|
|---|---|---|
|Liste|`MACHINE_IP scanme.nmap.org example.com`|3 IPs scannées|
|Range|`10.11.12.15-20`|6 IPs scannées|
|Subnet|`MACHINE_IP/30`|4 IPs scannées|
|Fichier|`nmap -iL list_of_hosts.txt`|IPs depuis fichier|

### Options utiles

bash

```bash
nmap -sL TARGETS   # Liste les hosts sans scanner (+ reverse-DNS)
nmap -sL -n TARGETS  # Idem sans résolution DNS
```

- `-sL` -> liste détaillée des cibles, 0 paquet envoyé
- Reverse-DNS activé par défaut -> peut révéler des infos utiles
- `-n` -> désactive le DNS

---

## Nmap Host Discovery Using ARP

### Comportement par défaut de Nmap

|Contexte|Méthode|
|---|---|
|Privileged + réseau local|ARP requests|
|Privileged + réseau distant|ICMP echo + TCP ACK:80 + TCP SYN:443 + ICMP timestamp|
|Unprivileged + réseau distant|TCP 3-way handshake (SYN sur ports 80 et 443)|

- Par défaut : ping scan pour trouver les hosts live, puis scan uniquement ces hosts
- `-sn` -> host discovery sans port scan

### ARP Scan

- Uniquement sur le **même subnet** (Ethernet/Wi-Fi)
- Principe : Nmap envoie ARP requests en broadcast -> host online = répond avec son MAC
- ARP opère au Layer 2 -> souvent **non filtré par les firewalls**

bash

```bash
nmap -PR -sn TARGETS    # ARP scan only, pas de port scan
```

- Destination MAC = broadcast (MAC cible inconnue au départ)
- Requêtes envoyées séquentiellement à chaque IP du subnet

### Cas d'usage

- Post-exploitation et énumération réseau interne
- Identifier rapidement les hosts live sur un segment local
- Efficace pour red-team / pentest interne car ARP rarement filtré

---

## Nmap Host Discovery Using ICMP

### ICMP Echo (`-PE`)

- Envoie ICMP Type 8 (Echo Request), attend Type 0 (Echo Reply)
- Souvent bloqué par firewalls et Windows (bloque ICMP echo par défaut)
- Si cible sur même subnet -> ARP précède l'ICMP

bash

```bash
nmap -PE -sn TARGETS
```

### ICMP Timestamp (`-PP`)

- Envoie ICMP Type 13 (Timestamp Request), attend Type 14 (Timestamp Reply)
- Alternative quand ICMP echo est bloqué

bash

```bash
nmap -PP -sn TARGETS
```

### ICMP Address Mask (`-PM`)

- Envoie ICMP Type 17, attend Type 18
- Souvent bloqué -> peut retourner 0 hosts même si des machines sont up
- Chaque requête envoyée deux fois

bash

```bash
nmap -PM -sn TARGETS
```

### Résumé

|Option|Type ICMP envoyé|Réponse attendue|
|---|---|---|
|`-PE`|Type 8 (Echo)|Type 0 (Echo Reply)|
|`-PP`|Type 13 (Timestamp)|Type 14 (Timestamp Reply)|
|`-PM`|Type 17 (Address Mask)|Type 18 (Address Mask Reply)|

-> Si un type est bloqué, essayer un autre. Toujours avoir plusieurs approches.

---

## Nmap Host Discovery Using TCP and UDP

### TCP SYN Ping (`-PS`)

- Envoie SYN sur port 80 (défaut) -> attend SYN/ACK (port open) ou RST (port closed)
- Seul le fait de recevoir une réponse compte -> host is up
- Privileged : envoie SYN sans compléter le 3-way handshake
- Unprivileged : forcé de compléter le 3-way handshake

bash

```bash
nmap -PS -sn TARGETS          # port 80 par défaut
nmap -PS21 -sn TARGETS        # port 21
nmap -PS21-25 -sn TARGETS     # ports 21 à 25
nmap -PS80,443,8080 -sn TARGETS
```

### TCP ACK Ping (`-PA`)

- Envoie ACK sur port 80 (défaut) -> cible répond RST (ACK hors connexion = réponse RST systématique)
- Requiert privilèges (sinon -> 3-way handshake)
- Nmap envoie chaque paquet deux fois

bash

```bash
nmap -PA -sn TARGETS
nmap -PA21,443 -sn TARGETS
```

### UDP Ping (`-PU`)

- UDP vers port ouvert -> pas de réponse
- UDP vers port **fermé** -> réponse ICMP port-unreachable = host is up
- Nmap cible des ports UDP probablement fermés pour déclencher cette réponse

bash

```bash
nmap -PU -sn TARGETS
```

### Résumé

|Option|Protocole|Mécanisme|Privilèges requis|
|---|---|---|---|
|`-PS`|TCP SYN|SYN -> SYN/ACK ou RST|Non (mais comportement différent)|
|`-PA`|TCP ACK|ACK -> RST|Oui|
|`-PU`|UDP|UDP fermé -> ICMP port-unreachable|Oui|

### Masscan

Alternative rapide à Nmap, approche similaire mais très agressive en rate de paquets.

bash

```bash
masscan 10.200.6.0/24 -p443
masscan 10.200.6.0/24 -p80,443
masscan 10.200.6.0/24 -p22-25
# Installation : apt install masscan
```

---

## Using Reverse-DNS Lookup

Reverse DNS : IP -> hostname (inverse du DNS classique). Utile pour identifier les rôles des machines (`mail.company.local`, `dc01.domain.com`).

|Option|Comportement|
|---|---|
|`-n`|Désactive le reverse-DNS|
|`-R`|Force le reverse-DNS pour tous les hosts (online ET offline)|
|`--dns-servers DNS_SERVER`|Utilise un DNS spécifique|

Par défaut : Nmap résout uniquement les hosts online.

Limites : records rDNS pas toujours configurés ou fiables, ralentit légèrement le scan.

---

## Summary

### Commandes

|Scan Type|Commande|
|---|---|
|ARP|`sudo nmap -PR -sn 10.200.6.0/24`|
|ICMP Echo|`sudo nmap -PE -sn 10.200.6.0/24`|
|ICMP Timestamp|`sudo nmap -PP -sn 10.200.6.0/24`|
|ICMP Address Mask|`sudo nmap -PM -sn 10.200.6.0/24`|
|TCP SYN Ping|`sudo nmap -PS22,80,443 -sn 10.200.6.0/30`|
|TCP ACK Ping|`sudo nmap -PA22,80,443 -sn 10.200.6.0/30`|
|UDP Ping|`sudo nmap -PU53,161,162 -sn 10.200.6.0/30`|

### Options générales

|Option|Rôle|
|---|---|
|`-sn`|Host discovery uniquement, pas de port scan|
|`-n`|Pas de DNS lookup|
|`-R`|Reverse-DNS pour tous les hosts|

-> Sans `-sn` : Nmap enchaîne automatiquement sur un port scan des hosts live.

