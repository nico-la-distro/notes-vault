## Command-Line Packet Analysis Hints

TShark s'utilise en combinaison avec des outils CLI standards pour l'analyse, le filtrage et l'automatisation.

|Outil|Utilité|
|---|---|
|`capinfos`|Résumé d'un fichier pcap (à lancer en premier)|
|`grep`|Recherche dans du texte|
|`cut`|Extraction de champs|
|`uniq`|Suppression des doublons|
|`nl`|Numérotation des lignes|
|`sed`|Édition de flux texte|
|`awk`|Recherche et traitement par pattern|

Note: Sample usage of these tools is covered in the [Zeek](https://tryhackme.com/room/zeekbro) room.

Répertoire de travail : `cd Desktop/exercise-files/`

### Capinfos

bash

```bash
capinfos demo.pcapng
```

Retourne : nom de fichier, type, encapsulation, nombre de paquets, taille, durée de capture, timestamps, checksums (SHA256, SHA1...), ordre temporel, interfaces.

---

## TShark Fundamentals I | Main Parameters I

### Command-Line Interface and Parameters

Superuser requis pour sniffer du trafic live et lister les interfaces.

|Paramètre|Usage|
|---|---|
|`-h`|Aide|
|`-v`|Version|
|`-D`|Liste les interfaces disponibles|
|`-i <id/nom>`|Choisit l'interface de capture|
|_(aucun)_|Sniffe comme tcpdump (interface 1 par défaut)|

bash

```bash
sudo tshark -D        # liste les interfaces
tshark -i 1           # capture sur interface 1
tshark -i ens55       # capture par nom
tshark                # capture sur l'interface par défaut (alias -i 1)
```

### Sniffing

- Sans `-i` → utilise la première interface listée (équivalent `-i 1`)
- TShark affiche toujours le nom de l'interface utilisée au démarrage
- Format de sortie : `N° | timestamp | src → dst | proto | infos`

---

## TShark Fundamentals I | Main Parameters II

### Command-Line Interface and Parameters II

|Paramètre|Usage|
|---|---|
|`-r <file>`|Lire un fichier pcap|
|`-c <n>`|Limiter à n paquets|
|`-w <file>`|Écrire les paquets dans un fichier|
|`-V`|Mode verbeux (équivalent Packet Details de Wireshark)|
|`-q`|Mode silencieux (supprime l'affichage terminal)|
|`-x`|Affiche les paquets en hex + ASCII|

bash

```bash
tshark -r demo.pcapng                        # lire un fichier
tshark -r demo.pcapng -c 2                   # lire les 2 premiers paquets
tshark -r demo.pcapng -c 1 -w output.pcap    # extraire le 1er paquet dans un fichier
tshark -r output.pcap -x                     # afficher en hex/ASCII
tshark -r demo.pcapng -c 1 -V               # vue verbeuse d'un paquet
```

#### Notes d'usage

- `-x` → à combiner avec `-c` pour éviter d'être noyé dans l'output
- `-V` → à utiliser **après filtrage**, pas sur l'ensemble du trafic
- `-w` → utile pour isoler des paquets suspects et les partager


![[tshark cli vs wireshark gui.png]]

---

## TShark Fundamentals II | Capture Conditions

### Capture Condition Parameters

Fonctionne uniquement en mode capture live (pas sur fichier pcap).

|Paramètre|Comportement|Sous-options|
|---|---|---|
|`-a`|Autostop — arrêt après condition|`duration:X` · `filesize:X` · `files:X`|
|`-b`|Ring buffer — boucle infinie, rotation des fichiers|`duration:X` · `filesize:X` · `files:X`|

- `-a files:X` → s'arrête après X fichiers
- `-b files:X` → écrase le fichier le plus ancien après X fichiers (rotation)
- `-a` + `-b` combinables — **obligatoire d'inclure un `-a` pour stopper une boucle `-b`**

bash

```bash
# Autostop : 2 sec, max 5 fichiers de 5 KB
tshark -w capture.pcap -a duration:2 -a filesize:5 -a files:5

# Ring buffer : fichiers de 10 KB, rotation sur 3 fichiers
tshark -w capture.pcap -b filesize:10 -b files:3
```

---

## TShark Fundamentals III | Packet Filtering Options: Capture vs. Display Filters

### Packet Filtering Parameters | Capture & Display Filters

A quick recap from the [Wireshark: Packet Operations](https://tryhackme.com/r/room/wiresharkpacketoperations) room:

| **Capture Filters** | Live filtering options. The purpose is to **save** only a specific part of the traffic. It is set before capturing traffic and is not changeable during live capture. |
| ------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Display Filters** | Post-capture filtering options. The purpose is to investigate packets by **reducing the number of visible packets**, which is changeable during the investigation.    |

| Paramètre | Type                                           | Moment d'application                      |
| --------- | ---------------------------------------------- | ----------------------------------------- |
| `-f`      | Capture filter (BPF = Berkeley Packet Filters) | Avant/pendant la capture — non modifiable |
| `-Y`      | Display filter (Wireshark)                     | Post-capture — modifiable                 |

- `-f` → réduit ce qui est **enregistré** (scope brut : plage, protocole, direction)
- `-Y` → réduit ce qui est **affiché** sans modifier le fichier

Check out the [**Wireshark: Packet Operations**](https://tryhackme.com/room/wiresharkpacketoperations) room (Task 4 & 5) if you want to review the principles of packet filtering.

bash

```bash
tshark -f "tcp port 80"          # capture uniquement le trafic TCP port 80
tshark -r demo.pcapng -Y "http"  # affiche uniquement les paquets HTTP
```

---

## TShark Fundamentals IV | Packet Filtering Options: Capture Filters

### Capture Filters

You can read more on capture filter syntax [here(opens in new tab)](https://www.wireshark.org/docs/man-pages/pcap-filter.html) and [here(opens in new tab)](https://gitlab.com/wireshark/wireshark/-/wikis/CaptureFilters#useful-filters).

Syntaxe BPF : `[direction] [type] [valeur]`

|Qualifier|Options|
|---|---|
|Type|`host` · `net` · `port` · `portrange` (défaut : `host`)|
|Direction|`src` · `dst` (défaut : les deux)|
|Protocole|`arp` · `ether` · `icmp` · `ip` · `ip6` · `tcp` · `udp`|

bash

```bash
tshark -f "host 10.10.10.10"
tshark -f "net 10.10.10.0/24"
tshark -f "port 80"
tshark -f "portrange 80-100"
tshark -f "src host 10.10.10.10"
tshark -f "dst host 10.10.10.10"
tshark -f "tcp"
tshark -f "udp"
tshark -f "ether host F8:DB:C5:A2:5D:81"
tshark -f "ip proto 1"          # ICMP via numéro de protocole IANA
```

Opérateurs booléens supportés : `and` · `or` · `not`

---

## TShark Fundamentals V | Packet Filtering Options: Display Filters

### Display Filters

You can use the official [**Display Filter Reference**(opens in new tab)](https://www.wireshark.org/docs/dfref/) to find the protocol breakdown for filtering.

Utiliser des guillemets simples pour éviter les problèmes bash. 

you can check the [Wireshark: Packet Operations](https://tryhackme.com/room/wiresharkpacketoperations) room (Task 4 & 5) if you want to review the principles of packet filtering.

bash

```bash
# IP
tshark -r demo.pcapng -Y 'ip.addr == 10.10.10.10'
tshark -r demo.pcapng -Y 'ip.addr == 10.10.10.0/24'
tshark -r demo.pcapng -Y 'ip.src == 10.10.10.10'
tshark -r demo.pcapng -Y 'ip.dst == 10.10.10.10'

# TCP
tshark -r demo.pcapng -Y 'tcp.port == 80'
tshark -r demo.pcapng -Y 'tcp.srcport == 80'

# HTTP
tshark -r demo.pcapng -Y 'http'
tshark -r demo.pcapng -Y 'http.response.code == 200'

# DNS
tshark -r demo.pcapng -Y 'dns'
tshark -r demo.pcapng -Y 'dns.qry.type == 1'   # requêtes de type A
```

Les numéros affichés correspondent à la position dans le fichier original, pas au résultat filtré. Utiliser `nl` pour compter les paquets filtrés :

bash

```bash
tshark -r demo.pcapng -Y 'http' | nl
```

