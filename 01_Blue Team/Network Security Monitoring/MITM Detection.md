## MITM Attack: An Overview

Attaquant intercepte/modifie silencieusement les communications entre deux parties. Cible : credentials, données bancaires, injection de contenu malveillant.

### How MITM Attacks Work

1. **Interception** : insertion dans le flux (ARP/DNS/IP spoofing)
2. **Manipulation/Decryption** : déchiffrement ou injection de contenu malveillant

### Common Types of MITM Attacks

|Technique|Description|
|---|---|
|Packet sniffing|Capture de paquets non chiffrés (Wi-Fi ouvert)|
|Session hijacking|Vol de tokens de session|
|SSL stripping|Downgrade HTTPS → HTTP|
|DNS spoofing|Redirection vers domaine frauduleux|
|IP spoofing|Paquets forgés depuis une IP de confiance|
|Rogue Wi-Fi AP|Faux point d'accès pour intercepter le trafic|

### MITM and Cyber Kill Chain

| Phase            | Rôle du MITM                                                                                                                 |
| ---------------- | ---------------------------------------------------------------------------------------------------------------------------- |
| **Exploitation** | Exploite les failles de confiance ARP/DNS → interception du canal, foothold initial                                          |
| **Installation** | Position MITM = vecteur de livraison → injection de malware/RAT (Remote Access Trojan) dans des téléchargements non chiffrés |

Détecter un MITM = adversaire actif en phase intermédiaire → fenêtre d'intervention SOC avant atteinte des objectifs finaux.

---

## Detecting ARP Spoofing

### What Is the ARP Protocol

Mappe IP → MAC sur le réseau local. Fonctionne par broadcast ("Who has X?") + réponse ("X is at MAC").

### ARP Spoofing

L'attaquant envoie de fausses réponses ARP pour associer son MAC à l'IP du gateway → tout le trafic victime passe par lui (MITM). Possible car **ARP n'a aucune authentification**.

### Indicators of the Attack

- **Duplicate MAC-to-IP Mappings**: Multiple MAC addresses claiming the same IP address. Indicates impersonation.
- **Unsolicited ARP Replies**: High number of ARP replies without matching requests ("gratuitous ARP").
- **Abnormal ARP Traffic Volume:** A Large number of ARP packets in short intervals.
- **Unusual Traffic Routing**: Traffic rerouted through the attacker’s MAC.
- **Gateway Redirection Patterns:** Multiple destination MACs for the same gateway IP.
- **ARP Probe / Reply Loops**: Many ARP requests with `Who has 192.168.1.x? Tell 192.168.1.y` patterns.

### Network Traffic Analysis

**Contexte réseau du lab :**

|Rôle|IP|MAC|
|---|---|---|
|Gateway|192.168.10.1|02:aa:bb:cc:00:01|

#### Filtres Wireshark

| Objectif                                | Filtre                                                                      |
| --------------------------------------- | --------------------------------------------------------------------------- |
| Tout le trafic ARP                      | `arp`                                                                       |
| Requêtes ARP uniquement                 | `arp.opcode == 1`                                                           |
| Réponses ARP uniquement                 | `arp.opcode == 2`                                                           |
| Gratuitous ARP                          | `arp.isgratuitous`                                                          |
| ARP légit du gateway                    | `arp && arp.src.proto_ipv4 == 192.168.10.1 && eth.src == 02:aa:bb:cc:00:01` |
| Tous les MAC revendiquant le gateway IP | `arp.opcode == 2 && arp.src.proto_ipv4 == 192.168.10.1`                     |
| Confirmation spoofing gateway           | `arp.opcode == 2 && _ws.col.info contains "192.168.10.1 is at"`             |
| Duplicate IP-to-MAC mappings            | `arp.duplicate-address-detected \| arp.duplicate-address-frame`             |

**Logique d'investigation :**

1. Isoler le trafic ARP
2. Repérer les gratuitous replies (réponses sans requête)
3. Filtrer sur l'IP du gateway → identifier les MACs qui la revendiquent
4. Confirmer avec le filtre duplicate-address

---

## Unmasking DNS Spoofing

### DNS Protocol Simplified

DNS = annuaire IP/domaine. Le DNS spoofing consiste à répondre à une requête DNS avec une fausse IP → la victime se connecte au serveur de l'attaquant sans le savoir.

**Chaîne d'attaque typique :** ARP spoofing (position MITM) → interception requête DNS → fausse réponse → victime redirigée vers serveur clone.

### Indicators of the Attack

- Plusieurs réponses DNS pour la même requête (légitime + forgée)
- Réponse DNS depuis une IP qui n'est pas le resolver configuré
- TTL anormalement court (1–30s) → l'attaquant ré-empoisonne régulièrement
- Réponse DNS sans requête correspondante

### Network Traffic Analysis

|Objectif|Filtre|
|---|---|
|Tout le trafic DNS|`dns`|
|Toutes les réponses DNS|`dns.flags.response == 1`|
|Réponses légitimes (Google DNS)|`dns.flags.response == 1 && ip.src == 8.8.8.8`|
|Trafic DNS pour un domaine précis|`dns && dns.qry.name == "corp-login.acme-corp.local"`|
|Réponses légitimes pour ce domaine|`dns.flags.response == 1 && ip.src == 8.8.8.8 && dns.qry.name == "corp-login.acme-corp.local"`|
|**Réponses forgées** (pas du DNS server)|`dns.flags.response == 1 && ip.src != 8.8.8.8 && dns.qry.name == "corp-login.acme-corp.local"`|

### Analysis Summary

Attaque multi-étapes confirmée :

1. ARP poisoning → gateway `192.168.10.1` redirigé vers MAC attaquant
2. DNS spoofing → `corp-login.acme-corp.local` redirigé vers IP attaquant

---

## Spotting SSL Stripping in Action

### How It Works

L'attaquant maintient une session HTTPS avec le serveur mais relaie le trafic en HTTP vers la victime → données en clair interceptables.

1. Victime initie HTTPS
2. Attaquant intercepte (via ARP spoofing)
3. Attaquant ↔ serveur : HTTPS
4. Attaquant → victime : HTTP

### Indicators of SSL stripping

- Requête initiale HTTPS (port 443) → bascule immédiatement sur HTTP (port 80)
- Redirects 301/302 forçant HTTP sur une ressource initialement HTTPS
- Échec du handshake TLS ou certificat auto-signé

### Network Traffic Analysis

| Objectif                                          | Filtre                                                                                               |
| ------------------------------------------------- | ---------------------------------------------------------------------------------------------------- |
| Tout le trafic SSL/TLS                            | `tls \| ssl`                                                                                         |
| Handshakes TLS vers le serveur cible              | `tls.handshake.type == 1 && tls.handshake.extensions_server_name == "corp-login.acme-corp.local"`    |
| Réponses DNS forgées de l'attaquant               | `dns.flags.response == 1 && ip.src == 192.168.10.55 && dns.qry.name == "corp-login.acme-corp.local"` |
| Trafic HTTP victime → attaquant (preuve du strip) | `http && ip.src == 192.168.10.10 && ip.dst == 192.168.10.55`                                         |

Le dernier filtre révèle la session HTTP en clair + credentials en plaintext dans un POST.

---
## Summary

|Étape|Action|
|---|---|
|ARP Spoofing|Attaquant associe son MAC au gateway IP via gratuitous ARP|
|DNS Spoofing|Requête DNS victime → fausse réponse → `corp-login.acme-corp.local` → `192.168.10.55`|
|SSL Stripping|Connexion HTTP vers attaquant → credentials capturés en clair|