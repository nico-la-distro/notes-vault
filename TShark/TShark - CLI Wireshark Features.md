## Command-Line Wireshark Features I | Statistics

**3 points clés :**

- Options appliquées à tous les paquets sauf si display filter fourni
- `-q` supprime l'affichage des paquets → stats uniquement
- TShark affiche le paramètre utilisé en début de sortie

|Paramètre|Usage|
|---|---|
|`--color`|Output colorisé style Wireshark|
|`-z <filter>`|Statistiques (voir options : `tshark -z help`)|
|`-q`|Supprime les paquets, affiche uniquement les stats|

### Colourised Output

bash

```bash
tshark -r file.pcap --color
```

### Statistics | Protocol Hierarchy

bash

```bash
tshark -r demo.pcapng -z io,phs -q          # Toute la hiérarchie
tshark -r demo.pcapng -z io,phs,udp -q      # Filtré sur UDP
```

### Statistics | Packet Lengths Tree

bash

```bash
tshark -r demo.pcapng -z plen,tree -q
```

Distribution des paquets par taille — détecte les paquets anormalement grands/petits.

### Statistics | Endpoints

bash

```bash
tshark -r demo.pcapng -z endpoints,ip -q
```

|Filtre|Protocole|
|---|---|
|`eth`|Ethernet|
|`ip`|IPv4|
|`ipv6`|IPv6|
|`tcp`|TCP (v4/v6)|
|`udp`|UDP (v4/v6)|
|`wlan`|802.11|

### Statistics | Conversations

bash

```bash
tshark -r demo.pcapng -z conv,ip -q
```

Mêmes filtres que Endpoints. Affiche le flux entre deux points (frames, bytes, durée).

### Statistics | Expert Info

bash

```bash
tshark -r demo.pcapng -z expert -q
```

Commentaires automatiques Wireshark : retransmissions, duplicates ACK, SYN/FIN, requêtes HTTP, etc.

---

## Command-Line Wireshark Features II | Statistics II

### Statistics | IPv4 and IPv6

|Paramètre|Description|
|---|---|
|`-z ptype,tree -q`|Distribution des protocoles IPv4 (TCP/UDP/...)|
|`-z ip_hosts,tree -q`|Tous les hosts IPv4|
|`-z ipv6_hosts,tree -q`|Tous les hosts IPv6|
|`-z ip_srcdst,tree -q`|Src & dst IPv4|
|`-z ipv6_srcdst,tree -q`|Src & dst IPv6|
|`-z dests,tree -q`|Destinations + ports IPv4|
|`-z ipv6_dests,tree -q`|Destinations + ports IPv6|

### Statistics | DNS

bash

```bash
tshark -r demo.pcapng -z dns,tree -q
```

Résumé des paquets DNS : opcodes, rcodes, types de requêtes.

### Statistics | HTTP

|Paramètre|Description|
|---|---|
|`-z http,tree -q`|Compteur paquets/statuts HTTP|
|`-z http2,tree -q`|Compteur paquets/statuts HTTP2|
|`-z http_srv,tree -q`|Load distribution|
|`-z http_req,tree -q`|Requêtes|
|`-z http_seq,tree -q`|Requêtes + réponses|

---

## Command-Line Wireshark Features III | Streams, Objects and Credentials

### Follow Stream

Structure : `-z follow,<protocol>,<mode>,<stream_id> -q`

|Protocole|Exemple|
|---|---|
|TCP|`-z follow,tcp,ascii,0 -q`|
|UDP|`-z follow,udp,ascii,0 -q`|
|HTTP|`-z follow,http,ascii,0 -q`|

- Modes dispo : `ascii`, `hex`
- Streams indexés à partir de `0`

### Export Objects

Structure : `--export-objects <protocol>,<dossier_cible> -q`

Protocoles supportés : `dicom`, `http`, `imf`, `smb`, `tftp`

bash

```bash
tshark -r demo.pcapng --export-objects http,/home/ubuntu/Desktop/extracted-by-tshark -q
```

### Credentials

Détecte les credentials en clair sur : FTP, HTTP, IMAP, POP, SMTP.

bash

```bash
tshark -r credentials.pcap -z credentials -q
```

Sortie : numéro de paquet, protocole, username, référence au paquet source.

---

## Advanced Filtering Options | Contains, Matches and Fields

### Contains / Matches

|Filtre|Case|Regex|Usage|
|---|---|---|---|
|`contains`|Sensible|Non|Cherche une valeur dans un champ|
|`matches`|Insensible|Oui|Cherche un pattern regex|

- Non utilisables sur des champs de type **integer**
- Préférer HEX ou regex plutôt qu'ASCII pour de meilleurs résultats

### Extract Fields

Structure : `-T fields -e <champ> [-e <champ>...] -E header=y`

bash

```bash
tshark -r demo.pcapng -T fields -e ip.src -e ip.dst -E header=y
```

Un `-e` par champ souhaité.

### Filter: "contains"

bash

```bash
tshark -r demo.pcapng -Y 'http.server contains "Apache"'

# Avec extraction de champs
tshark -r demo.pcapng -Y 'http.server contains "Apache"' -T fields -e ip.src -e ip.dst -e http.server -E header=y
```

### Filter: "matches"

bash

```bash
tshark -r demo.pcapng -Y 'http.request.method matches "(GET|POST)"'

# Avec extraction de champs
tshark -r demo.pcapng -Y 'http.request.method matches "(GET|POST)"' -T fields -e ip.src -e ip.dst -e http.request.method -E header=y
```

---

## Use Cases | Extract Information

A skilled analyst should know how to use native Linux tools/utilities to manage and organise the command line output

Pipeline de base pour tous les cas : `| awk NF | sort -r | uniq -c | sort -r`

|Étape|Rôle|
|---|---|
|`awk NF`|Supprime les lignes vides|
|`sort -r`|Trie avant déduplification|
|`uniq -c`|Déduplique + compte les occurrences|
|`sort -r`|Trie du plus fréquent au moins fréquent|

### Extract Hostnames

bash

```bash
tshark -r hostnames.pcapng -T fields -e dhcp.option.hostname | awk NF | sort -r | uniq -c | sort -r
```

### Extract DNS Queries

bash

```bash
tshark -r dns-queries.pcap -T fields -e dns.qry.name | awk NF | sort -r | uniq -c | sort -r
```

### Extract User Agents

bash

```bash
tshark -r user-agents.pcap -T fields -e http.user_agent | awk NF | sort -r | uniq -c | sort -r
```

User agents suspects à repérer : `sqlmap`, `Wfuzz`, `Nmap Scripting Engine`.

---

## Challenges

- [TShark Challenge I: Teamwork](https://tryhackme.com/r/room/tsharkchallengesone)
- [TShark Challenge II: Directory](https://tryhackme.com/r/room/tsharkchallengestwo)

