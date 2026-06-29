## Introduction

### Learning Objectives

Étapes Nmap vues jusqu'ici : enumerate targets -> discover live hosts -> reverse-DNS lookup.

Cette room couvre : TCP connect scan, TCP SYN scan, UDP scan + options ports / scan rate / parallel probes.

---

## TCP and UDP Ports

Un port identifie un service réseau sur un hôte. Un seul service peut écouter par port/IP.

Les 6 états de port selon Nmap :

|État|Signification|
|---|---|
|`open`|Un service écoute sur ce port|
|`closed`|Aucun service, mais port accessible (non bloqué)|
|`filtered`|Nmap ne peut pas déterminer l'état -> firewall bloque les paquets|
|`unfiltered`|Port accessible mais état indéterminable -> rencontré avec ACK scan (`-sA`)|
|`open\|filtered`|Nmap ne peut pas distinguer entre open et filtered|
|`closed\|filtered`|Nmap ne peut pas distinguer entre closed et filtered|

---

## TCP and UDP Ports

Un port identifie un service réseau sur un hôte. Un seul service peut écouter par port/IP.

Les 6 états de port selon Nmap :

|État|Signification|
|---|---|
|`open`|Un service écoute sur ce port|
|`closed`|Aucun service, mais port accessible (non bloqué)|
|`filtered`|Nmap ne peut pas déterminer l'état -> firewall bloque les paquets|
|`unfiltered`|Port accessible mais état indéterminable -> rencontré avec ACK scan (`-sA`)|
|`open\|filtered`|Nmap ne peut pas distinguer entre open et filtered|
|`closed\|filtered`|Nmap ne peut pas distinguer entre closed et filtered|

---

## TCP Flags

Les flags TCP occupent les 24 premiers octets du header TCP. Référence : [RFC 793](https://datatracker.ietf.org/doc/html/rfc793.html).

![[TCP header.png]]

|Flag|Rôle|
|---|---|
|`URG`|Données urgentes -> traitement immédiat|
|`ACK`|Accusé de réception d'un segment TCP|
|`PSH`|Transmet les données immédiatement à l'application|
|`RST`|Réinitialise la connexion (firewall, service absent)|
|`SYN`|Initie le 3-way handshake, synchronise les numéros de séquence|
|`FIN`|Indique que l'émetteur n'a plus de données à envoyer|

---

## TCP Connect Scan

Complète le 3-way handshake (SYN -> SYN/ACK -> ACK), puis coupe avec RST/ACK dès que l'état est confirmé.

Seule option disponible sans privilèges root/sudo.

bash

```bash
nmap -sT <target>
```

|Option|Effet|
|---|---|
|`-sT`|TCP connect scan|
|`-F`|Fast mode -> 100 ports au lieu de 1000|
|`-r`|Scan en ordre consécutif au lieu d'aléatoire|

Comportement :

- Port **open** -> cible répond SYN/ACK
- Port **closed** -> cible répond RST/ACK

Par défaut : 1000 ports les plus communs.

---

## TCP SYN Scan

Scan par défaut quand exécuté en root/sudo. Envoie un SYN, reçoit SYN/ACK, puis envoie RST sans compléter le handshake -> moins susceptible d'être loggé.

bash

```bash
nmap -sS <target>
```

| |`-sT` (Connect)|`-sS` (SYN)|
|---|---|---|
|Privilèges|Non requis|root/sudo requis|
|Handshake|Complet|Incomplet (RST après SYN/ACK)|
|Logging|Plus visible|Moins visible|
|Défaut|Non|Oui (si root)|

- Port **open** -> cible répond SYN/ACK -> Nmap envoie RST
- Port **closed** -> cible répond RST/ACK

---

## UDP Scan

Protocole sans connexion -> pas de handshake. Comportement :

- Port **open** -> aucune réponse (état supposé open)
- Port **closed** -> ICMP type 3, code 3 (port unreachable)

bash

```bash
nmap -sU <target>
nmap -sU --top-ports 10 <target>   # top 10 ports UDP
nmap -sU -sS <target>              # combinaison UDP + SYN
```

---

## Fine-Tuning Scope and Performance

**Sélection des ports :**

|Option|Effet|
|---|---|
|`-p22,80,443`|Ports spécifiques|
|`-p1-1023`|Plage de ports|
|`-p-`|Tous les 65535 ports|
|`-F`|100 ports les plus communs|
|`--top-ports 10`|10 ports les plus communs|

**Timing (`-T<0-5>`) :**

|Template|Usage typique|
|---|---|
|`-T0` paranoid|Stealth max, 1 port à la fois, 5 min entre chaque probe|
|`-T1` sneaky|Engagements réels (stealth)|
|`-T2` polite|-|
|`-T3` normal|Défaut|
|`-T4` aggressive|CTF / cibles d'entraînement|
|`-T5` insane|Vitesse max, risque de perte de paquets|

**Contrôle du débit :**

bash

```bash
--min-rate <n>        # paquets/sec minimum
--max-rate <n>        # paquets/sec maximum
```

**Parallélisation des probes :**

bash

```bash
--min-parallelism <n>   # probes parallèles minimum
--max-parallelism <n>   # probes parallèles maximum
```

---

## Summary

|Scan|Commande|
|---|---|
|TCP Connect|`nmap -sT <target>`|
|TCP SYN|`sudo nmap -sS <target>`|
|UDP|`sudo nmap -sU <target>`|

|Option|Effet|
|---|---|
|`-p-`|Tous les ports|
|`-p1-1023`|Ports 1 à 1023|
|`-F`|100 ports les plus communs|
|`-r`|Ordre consécutif|
|`-T<0-5>`|Timing (0 = lent, 5 = rapide)|
|`--max-rate 50`|<= 50 paquets/sec|
|`--min-rate 15`|>= 15 paquets/sec|
|`--min-parallelism 100`|>= 100 probes en parallèle|

