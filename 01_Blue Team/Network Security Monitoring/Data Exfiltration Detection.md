## Data Exfil: Overview, techniques, and Indicators

### Why Adversaries Perform Data Exfiltration

| Motif                  | Détail                                     |
| ---------------------- | ------------------------------------------ |
| Financial Gain         | Vente dark web, fraude                     |
| Espionage              | PI (intellectual property), secrets d'État |
| Ransomware & Extortion | Double extorsion : vol + chiffrement       |
| Disruption & Sabotage  | Fuite publique pour nuire                  |
| Persistence & Recon    | Comprendre l'env pour attaques futures     |

### Threat Actors & Their Exfiltration Techniques

| Acteur                                                                                                                                                         | Technique                          | Description                                                                        |
| -------------------------------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------- | ---------------------------------------------------------------------------------- |
| [**APT29 (Cozy Bear)**](https://www.picussecurity.com/resource/blog/apt29-cozy-bear-evolution-techniques)                                                      | HTTPS via domaines légitimes       | Encrypted HTTPS channels were used to exfiltrate data from government networks.    |
| [**FIN7**](https://www.fbi.gov/contact-us/field-offices/seattle/news/stories/how-cyber-crime-group-fin7-attacked-and-stole-data-from-hundreds-of-us-companies) | HTTP POST vers C2                  | Embedded stolen data in HTTP POST requests to evade detection.                     |
| [**Lunar Spider (Zloader)**](https://thedfirreport.com/2025/09/29/from-a-single-click-how-lunar-spider-enabled-a-near-two-month-intrusion/)                    | C2 chiffré + exfil stagée          | Maintained a two-month intrusion using encrypted channels and staged exfiltration. |
| [**DarkSide Ransomware**](https://www.varonis.com/blog/darkside-ransomware)                                                                                    | Dual extortion (vol + chiffrement) | Stole data before encrypting systems, then threatened public leaks.                |
| [**APT10 (Cloud Hopper)**](https://www.sei.cmu.edu/blog/operation-cloud-hopper-case-study/)                                                                    | Cloud-to-cloud via APIs MSP        | Exfiltrated data from managed service providers using cloud APIs.                  |

### Common phases related to exfiltration

1. **Discovery / Collection** — localisation des fichiers sensibles
2. **Staging / Compression** — ZIP, RAR, 7z, tar, base64, stéganographie
3. **Exfiltration transport** — réseau, média amovible, cloud, canaux couverts
4. **C2 coordination** — orchestration + confirmation de réception

### Techniques and Indicators

| Technique                     | Exemples                                                                                                                                                                                                    | Où chercher                                                                                                                         |
| ----------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------- |
| Network-based                 | HTTP/S uploads, FTP/SCP, DNS tunnelling, ICMP, TCP/UDP custom                                                                                                                                               | Proxy (gros POSTs), firewall flows (haut volume vers 1 IP), netflow (spikes/outbound flows), DNS logs (long hostnames, TXT queries) |
| Host-based                    | PowerShell/Invoke-WebRequest, rclone, curl, zip/rar, USB, ADS                                                                                                                                               | Sysmon/EDR (Event 1/3/11), Windows Sec (4663/4656), auditd, shell history                                                           |
| Cloud exfiltration            | S3 PutObject, Azure Blob upload, GDrive/SharePoint external share                                                                                                                                           | CloudTrail, Azure Activity, GCP Audit, cloud storage logs                                                                           |
| Covert & encoding             | DNS tunnelling, base64, stéganographie, low-and-slow chunking (splitting files into many small requests)                                                                                                    | DNS logs, proxy logs (nombreux petits POSTs), corrélation uploads + proc suspects                                                   |
| Insider & collab tools        | Slack/Teams/Dropbox/GDrive vers users externes, comptes compromis                                                                                                                                           | Audit logs (share events, downloads), mail logs                                                                                     |
| General IoAs & triage signals | A large outbound volume to external IPs/domains, unknown destination domains, suspicious processes/command lines, many file read events followed by an outbound connection, and multipart/streamed uploads. | Correlate: Proxy/Firewall/Netflow, DNS, Sysmon/EDR (EventID 1/3/11), mail server logs.                                              |

**Triage signals clés :** gros volume sortant, domaines inconnus, process suspects, nombreux reads fichiers + connexion sortante, uploads multipart/streamés.

> Détection efficace = corrélation host + network + cloud — pas une alerte isolée. Questions clés : **qui** a accédé, **quoi** transféré, **comment** stagé, **où** envoyé.

---

## Detection: Data Exfil through DNS Tunneling

DNS tunneling encode des données dans les requêtes/réponses DNS. Contourne firewalls et proxies car le port 53 est quasi toujours autorisé.

### Indicators of Attack

- Nombre élevé de requêtes vers un seul domaine externe
- Sous-domaines longs (> 60–100 chars)
- Entropie élevée / patterns Base32/Base64 dans le nom de requête
- Record types rares : TXT, NULL
- Nombreux NXDOMAIN ou fragments TCP/UDP larges
- Requêtes à intervalles réguliers (beaconing)

### Detecting through Wireshark

|Objectif|Filtre|
|---|---|
|Trafic DNS global|`dns`|
|Requêtes sans réponse|`dns.flags.response == 0`|
|Requêtes longues (suspicious)|`dns && frame.len > 70`|
|Filtrer sur domaine suspect|`dns && dns.qry.name contains <domain>`|

### Investigating with Splunk

spl

```spl
# Logs DNS bruts
index=data_exfil sourcetype=DNS_logs

# Requêtes par source IP
index="data_exfil" sourcetype="DNS_logs" | stats count by src_ip

# Top queries par volume
index="data_exfil" sourcetype="dns_logs" | stats count by query | sort -count

# Filtrer queries longues (> 30 chars)
index="data_exfil" sourcetype="DNS_logs" | where len(query) > 30
```

**Indicateurs confirmés :**

- Grand nombre de requêtes DNS sans réponse
- Longueur anormalement élevée des queries DNS

---

## Detection: Data Exfil through FTP

FTP = protocole TCP/IP en clair. Utilisé par les attaquants via credentials compromis, serveurs mal configurés, ou comptes éphémères.

### How adversaries use FTP for exfiltration

- Serveurs FTP légitimes ou internes mal configurés
- Credentials compromis (comptes service, users)
- Ports non-standard ou tunneling

### Indicators of Attack

- Commandes `USER` / `PASS` en clair
- `STOR` (upload) / `RETR` (download) répétés ou volumineux
- Grosses connexions vers IPs externes inhabituelles, hors heures ouvrées
- Ouvertures de canaux data sur ports éphémères (PASV) + gros payloads

### Detecting through Wireshark

| Objectif                       | Filtre                                                             |
| ------------------------------ | ------------------------------------------------------------------ |
| Sessions FTP (contrôle + data) | `ftp \|\| ftp-data`                                                |
| Tentatives de login            | `ftp.request.command == "USER" \|\| ftp.request.command == "PASS"` |
| Uploads                        | `ftp contains "STOR"`                                              |
| Fichiers CSV suspects          | `ftp contains "csv"`                                               |
| Gros payloads                  | `ftp && frame.len > 90`                                            |

> Pour inspecter le contenu : clic droit sur paquet → **Follow → TCP Stream**

**Indicateurs confirmés :**

- Compte Guest connecté sur IP externe suspecte
- Transfert de fichiers CSV sensibles
- Documents sensibles dans les streams à large payload

---

## Detection: Data Exfil via HTTP

HTTP abusé car il se fond dans le trafic légitime, traverse firewalls/proxies, et supporte obfuscation/chiffrement.

### How adversaries use HTTP for data exfiltration

- POST vers serveurs externes (bulk data)
- GET avec data encodée dans query strings (low-and-slow)
- Headers custom (`X-Data: <base64>`) pour bypass DLP
- Chunked/multipart pour éviter les seuils de taille
- HTTPS/TLS tunneling (détection via SNI _Server Name Indication_ ou metadata)
- Staging via Dropbox/GitHub/Gist

### Indicators of Attack (IoAs)

- Gros POST vers hosts externes/inconnus
- Domaines à faible réputation ou absents du baseline
- Beaconing (nombreuses petites requêtes) suivi de gros uploads
- Transferts chunked/multipart composant un fichier plus large

### Analyzing Logs in Splunk

spl

```spl
# Logs HTTP bruts
index="data_exfil" sourcetype="http_logs"

# Filtrer sur méthode POST
index="data_exfil" sourcetype="http_logs" method=POST

# Stats bytes par domaine
index="data_exfil" sourcetype="http_logs" method=POST
| stats count avg(bytes_sent) max(bytes_sent) min(bytes_sent) by domain
| sort - count

# Isoler gros payloads POST (> 600 bytes)
index="data_exfil" sourcetype="http_logs" method=POST bytes_sent > 600
| table _time src_ip uri domain dst_ip bytes_sent
| sort - bytes_sent
```

### Network Traffic Analysis (Wireshark)

|Objectif|Filtre|
|---|---|
|Trafic HTTP global|`http`|
|Requêtes POST|`http.request.method == "POST"`|
|POST + payload > 500|`http.request.method == "POST" and frame.len > 500`|
|POST + payload > 750|`http.request.method == "POST" and frame.len > 750`|

> Inspecter le contenu : **Follow → HTTP Stream** sur le paquet isolé

**Workflow :** Splunk (identifier domaine/IP suspect + volume) → Wireshark (corréler avec pcap + inspecter payload)

---

## Detection: Data Exfiltration via ICMP

ICMP (ping, TTL exceeded) est peu inspecté par les firewalls → attrayant pour tunneler des données en encodant des chunks dans les payloads echo request/reply.

### How adversaries use ICMP for exfiltration

- Echo request (type 8) / reply (type 0) avec payload encodé (base64, hex)
- ICMP types/codes non-standard pour éviter les signatures
- Fragmentation de gros payloads sur plusieurs paquets
- Chiffrement/obfuscation du contenu

### Indicators of Attack

- Sessions ICMP persistantes vers un host externe non légitime
- Payloads ICMP anormalement grands (> taille normale d'un ping)
- Entropie élevée ou patterns base64/hex dans le payload
- Bursts ICMP sans autre trafic applicatif depuis le même host
- Espacement régulier des paquets (beaconing)

### Traffic Analysis (Wireshark)

|Objectif|Filtre|
|---|---|
|Tout le trafic ICMP|`icmp`|
|Echo Requests uniquement|`icmp.type == 8`|
|Gros payloads suspects|`icmp.type == 8 and frame.len > 100`|

> Ping normal ≈ 74 bytes. Tout ce qui dépasse 100 bytes est suspect.

---
