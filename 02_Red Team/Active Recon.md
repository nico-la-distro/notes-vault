## Introduction

Active recon = contact direct avec la cible -> laisse des traces dans les logs (IP, heure, durée). Nécessite une autorisation légale signée avant de commencer.

Outils couverts : navigateur web, `ping`, `traceroute`, `telnet`, `nc`

---
## Web Browser

Ports par défaut : TCP 80 (HTTP), TCP 443 (HTTPS). Accès port custom : `https://IP:PORT` (ex: `https://127.0.0.1:8834/`)

DevTools : `Ctrl+Shift+I` (PC) / `Option+Cmd+I` (Mac) -> inspecter JS, cookies, structure du site, requêtes/réponses

|Extension|Usage|
|---|---|
|[FoxyProxy](https://addons.mozilla.org/en-US/firefox/addon/foxyproxy-standard)|Changer rapidement de proxy (Burp Suite, etc.)|
|[User-Agent Switcher](https://addons.mozilla.org/en-US/firefox/addon/user-agent-string-switcher)|Usurper l'OS/navigateur|
|[Wappalyzer](https://addons.mozilla.org/en-US/firefox/addon/wappalyzer)|Identifier les technos du site visité|

---

## Ping

Protocole : ICMP Echo (type 8) / Echo Reply (type 0). Header ICMP = **8 bytes**.

bash

```bash
ping MACHINE_IP               # Linux : Ctrl+C pour arrêter
ping -c 10 MACHINE_IP         # Linux : limiter à 10 paquets
ping -n 10 MACHINE_IP         # Windows : 10 paquets
ping -s SIZE MACHINE_IP       # Changer la taille des données
```

Absence de réponse -> machine éteinte, firewall bloque ICMP, ou réseau coupé. Note : le firewall Windows bloque le ping par défaut (Y).

---

## Traceroute

Trace le chemin des paquets et révèle les IP des routeurs intermédiaires.

Mécanisme : envoie des paquets avec TTL croissant (1, 2, 3...) -> chaque routeur décrémente TTL de 1 -> TTL=0 -> routeur renvoie ICMP "Time-to-Live exceeded" -> son IP est révélée.

`*` dans la sortie = routeur configuré pour ne pas répondre aux ICMP. La route peut varier entre deux exécutions (routage dynamique).

bash

```bash
traceroute MACHINE_IP    # Linux / macOS
tracert MACHINE_IP       # Windows
```

---

## Telnet

Port 23, données transmises **en clair** (non chiffré). Utile pour le **banner grabbing**.

bash

```bash
telnet MACHINE_IP PORT
```

Exemple banner grabbing HTTP (port 80) :

```
telnet MACHINE_IP 80
GET / HTTP/1.1
host: telnet
[Entrée x2]
```

-> la réponse contient le nom et la version du serveur web.

---

## Netcat

Fonctionne comme telnet mais plus polyvalent : peut agir en client **ou** en serveur.

bash

```bash
nc MACHINE_IP PORT    # Client (connexion sortante)
nc -lvnp PORT         # Serveur (écoute sur un port)
```

|Option|Usage|
|---|---|
|`-l`|Mode écoute (listen)|
|`-v`|Verbose|
|`-n`|Pas de résolution DNS|
|`-p`|Spécifier le port|

Banner grabbing : même principe que telnet, envoyer la requête manuellement après connexion.

---

## Putting It All Together

|But|Outil|Commande|
|---|---|---|
|Vérifier si hôte en ligne|ping|`ping -c 10 MACHINE_IP`|
|Tracer la route / compter les hops|traceroute|`traceroute MACHINE_IP`|
|Banner grabbing / test port|telnet|`telnet MACHINE_IP PORT`|
|Connexion ou écoute sur un port|netcat|`nc MACHINE_IP PORT`|
|Inspecter JS, cookies, headers|navigateur|`Ctrl+Shift+I`|
