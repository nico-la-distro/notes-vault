## Nmap Scans

- Identifier les **patterns réseau générés par Nmap**
- Types principaux :
    - TCP Connect (`-sT`)
    - SYN Scan (`-sS`)
    - UDP Scan (`-sU`)

### 🧩 TCP Flags (Wireshark)

| Flag    | Description     | Filter Wireshark       |
| ------- | --------------- | ---------------------- |
| SYN     | Init connexion  | `tcp.flags.syn == 1`   |
| ACK     | Ack réception   | `tcp.flags.ack == 1`   |
| SYN+ACK | Réponse serveur | `tcp.flags == 18`      |
| RST     | Reset connexion | `tcp.flags.reset == 1` |
| RST+ACK | Refus connexion | `tcp.flags == 20`      |
| FIN     | Fermeture       | `tcp.flags.fin == 1`   |

👉 Filtre global :

tcp OR udp

### 🔗 TCP Connect Scan (`-sT`)

⚙️ Caractéristiques

- Utilise **3-way handshake complet**
- Accessible sans privilèges (non-root)
- **Window size > 1024**

🔍 Comportement

|État port|Séquence|
|---|---|
|Open|SYN → SYN/ACK → ACK|
|Closed|SYN → RST/ACK|

![[wireshark traffic analysis tcp connect scans open&closed tcp port.png]]

🎯 Filtre détection

tcp.flags.syn==1 and tcp.flags.ack==0 and tcp.window_size > 1024

![[wireshark traffic analysis show tcp connect scan patterns.png]]

### ⚡ SYN Scan (`-sS`)

⚙️ Caractéristiques

- **Pas de handshake complet** (stealth)
- Nécessite privilèges (root)
- **Window size ≤ 1024**

🔍 Comportement

|État port|Séquence|
|---|---|
|Open|SYN → SYN/ACK → RST|
|Closed|SYN → RST/ACK|

![[wireshark traffic analysis syn scans open&closed tcp port.png]]

🎯 Filtre détection

tcp.flags.syn==1 and tcp.flags.ack==0 and tcp.window_size <= 1024

![[wireshark traffic analysis tcp syn scan patterns.png]]

### 📡 UDP Scan (`-sU`)

⚙️ Caractéristiques

- **Pas de handshake**
- Pas de réponse = port possiblement open
- Port fermé → **ICMP error**

🔍 Comportement

|État port|Réponse|
|---|---|
|Open|Pas de réponse|
|Closed|ICMP Type 3 Code 3|

![[wireshark traffic analysis udp scan closed&open port.png]]

👉 ICMP encapsule la requête originale

The ICMP error message uses the original request as encapsulated data to show the source/reason of the packet. Once you expand the ICMP section in the packet details pane, you will see the encapsulated data and the original request, as shown in the below image.

![[wireshark traffic analysis udp scan error msg icmp encapsulated original request.png]]

🎯 Filtre détection

icmp.type==3 and icmp.code==3

![[wireshark traffic analysis show udp scan patterns.png]]

### 🧪 Astuce Analyse

1. Utiliser filtres génériques → repérer anomalies
2. Zoom sur flux spécifique
3. Vérifier patterns (flags + taille fenêtre + réponses)

### ⚡ Points clés à retenir

- **Window size = indicateur clé** (Connect vs SYN)
- **SYN scan = furtif**
- **UDP = silence ou ICMP**
- Toujours analyser **flags + séquence + contexte**

---

