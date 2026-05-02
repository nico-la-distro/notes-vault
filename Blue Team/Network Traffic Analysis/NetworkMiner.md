## NetworkMiner in Forensics

Objectif : détecter activités malveillantes, violations de sécurité, anomalies réseau via le trafic.

Apports de NetworkMiner :

- Contexte des hôtes : IP, MAC, hostname, OS
- Indicateurs d’attaque/anomalies : pics de trafic, scans de ports
- Outils utilisés : ex. Nmap
### Supported Data Types

3 types analysés :

- Live Traffic
- Traffic Captures (PCAP)
- Log Files

NetworkMiner :

- supporte PCAP + trafic live
- focus ici : live + captures

---

## What is NetworkMiner?

### NetworkMiner in a Nutshell

| Capability                 | Description                                  |
| -------------------------- | -------------------------------------------- |
| Traffic sniffing           | Interception + capture + logging des paquets |
| Parsing PCAP files         | Analyse détaillée des paquets                |
| Protocol analysis          | Identification des protocoles                |
| OS fingerprinting          | Détection OS via PCAP (Satori, p0f)          |
| File Extraction            | Extraction images, HTML, emails              |
| Credential grabbing        | Extraction identifiants                      |
| Clear text keyword parsing | Extraction chaînes/keywords en clair         |

Version utilisée : Free (moins de features que Pro)

### Operating Modes

**Sniffer Mode**

- Sniffing dispo uniquement Windows
- Peu fiable → non recommandé
- Pas un sniffer dédié (≠ Wireshark, tcpdump)

**Packet Parsing/Processing**

- Analyse PCAP pour vue rapide
- Objectif : récupérer infos rapides ("low hanging fruit")

### Pros and Cons

**Pros**

- OS fingerprinting
- File extraction simple
- Credential grabbing
- Cleartext parsing
- Vue globale rapide

**Cons**

- Mauvais en sniffing actif
- Inefficace sur gros PCAP
- Filtrage limité
- Pas adapté à l’analyse manuelle fine

### Differences Between Wireshark and NetworkMiner

Bonne pratique :  
→ Capture trafic → aperçu rapide avec NetworkMiner → analyse approfondie avec Wireshark

|Feature|NetworkMiner|Wireshark|
|---|---|---|
|Purpose|Vue rapide, mapping, extraction|Analyse approfondie|
|GUI|✅|✅|
|Sniffing|✅|✅|
|PCAP|✅|✅|
|OS Fingerprinting|✅|❌|
|Keyword Discovery|✅|Manuel|
|Credential Discovery|✅|✅|
|File Extraction|✅|✅|
|Filtering|Limité|✅|
|Packet Decoding|Limité|✅|
|Protocol Analysis|❌|✅|
|Payload Analysis|❌|✅|
|Statistical Analysis|❌|✅|
|Cross-Platform|✅|✅|
|Host Categorisation|✅|❌|
|Ease of Management|✅|✅|

---

## Tool Overview 1

### Landing Page

Écran initial au lancement de NetworkMiner

![[NetworkMiner landing page.png]]

### File Menu

- Charger PCAP ou drag & drop
- Réception PCAP over IP (non utilisé ici)
- Usage recommandé : aperçu rapide + low hanging fruit

![[NetworkMiner file menu.png]]

### Tools Menu

- Nettoyer dashboard
- Supprimer données capturées

![[NetworkMiner tools menu.png]]

### Help Menu

- Infos version + mises à jour

![[NetworkMiner help menu.png]]

### Case Panel

- Liste des PCAP chargés
- Actions : reload, metadata, suppression

![[NetworkMiner case panel.png]]

**Viewing metadata of loaded files**

- Affichage des métadonnées des fichiers chargés

![[NetworkMiner viewing metadata.png]]

### Hosts

Infos par hôte :

- IP, MAC, OS
- Ports ouverts
- Paquets envoyés/reçus
- Sessions entrantes/sortantes

Notes :

- OS fingerprinting : Satori (GitHub repo) + p0f
- MAC DB : mac-ages (GitHub repo)
- Tri + coloration possibles
- OSINT lookup → premium
- Right-click → copier valeurs

![[NetworkMiner hosts.png]]

### Sessions

Infos sessions :

- Frame number
- Client / server
- Ports source/destination
- Protocole
- Start time

![[NetworkMiner sessions.png]]

Filtrage :

- Barre de recherche + colonnes
- Types :
    - `"ExactPhrase"`
    - `"AllWords"`
    - `"AnyWord"`
    - `"RegExe"`

### DNS

Infos requêtes DNS :

- Frame, timestamp
- Client / server
- Ports
- TTL
- DNS time
- Transaction ID + type
- Query + answer

Notes :

- Alexa Top 1M → premium
- Barre de recherche dispo

![[NetworkMiner dns.png]]

### Credentials

Extraction creds + hashes :

- Kerberos
- NTLM
- RDP cookies
- HTTP cookies / requests
- IMAP, FTP, SMTP
- MS SQL

Outils de crack :

- Hashcat https://tryhackme.com/room/crackthehashlevel2
- John the Ripper 

Notes :

- Right-click → copier username/password

![[NetworkMiner credentials.png]]

---

### Lab t4

#### What is the total number of frames?

![[NetworkMiner t6q1.png]]

#### How many IP addresses use the same MAC address with host 145.253.2.203?

![[NetworkMiner t6q2.png]]

#### How many packets were sent from host 65.208.228.223?

![[NetworkMiner t6q3.png]]

#### What is the name of the webserver banner under host 65.208.228.223?

![[NetworkMiner t6q4.png]]

#### What is the extracted username for the 02694W-WIN10 host?

![[NetworkMiner t6q5.png]]

#### What is the extracted password for the user logged into the 02694W-WIN10 host? Enter the full NTLM hash.

![[NetworkMiner t6q6.png]]

---

## Tool Overview 2

### Files

Fichiers extraits du PCAP

| Champ                | Contenu              |
| -------------------- | -------------------- |
| Frame number         | Numéro de trame      |
| Filename             | Nom fichier          |
| Extension            | Type                 |
| Size                 | Taille               |
| Source / Destination | IP                   |
| Ports                | Source / destination |
| Protocol             | Protocole            |
| Timestamp            | Horodatage           |
| Reconstructed path   | Chemin reconstruit   |
| Details              | Infos complètes      |

Notes :

- OSINT hash lookup + sample submission → premium
- Search bar dispo
- Right-click : ouvrir fichiers/dossiers + détails

![[NetworkMiner files.png]]

### Images

Images extraites du PCAP

- Open + zoom in/out via clic droit

![[NetworkMiner images.png]]

Hover image :

- Source / destination
- File path

![[NetworkMiner hover image.png]]

### Parameters

Paramètres extraits

| Champ                     | Contenu              |
| ------------------------- | -------------------- |
| Parameter name            | Nom                  |
| Parameter value           | Valeur               |
| Frame number              | Trame                |
| Source / destination host | Hôtes                |
| Ports                     | Source / destination |
| Timestamp                 | Temps                |
| Details                   | Infos                |

- Right-click : copier nom/valeur

![[NetworkMiner parameters.png]]

### Keywords

Mots-clés extraits

|Champ|Contenu|
|---|---|
|Frame number|Trame|
|Timestamp|Temps|
|Keyword|Mot-clé|
|Context|Contexte|
|Source / destination|Hôtes + ports|

Filtrage :

- Ajout keywords
- Reload case files obligatoire
- Multi-keywords support
- Scan global PCAP après reload

![[NetworkMiner keywords.png]]

### Messages

Extraction communications :

- Emails
- Chats
- Messages

|Champ|Contenu|
|---|---|
|Frame number|Trame|
|Source / destination|Hôtes|
|Protocol|Protocole|
|From|Expéditeur|
|To|Destinataire|
|Timestamp|Temps|
|Size|Taille|

Notes :

- Détails : attachments + attributs
- Viewer intégré
- Open file pour pièces jointes
- Search bar + right-click dispo

![[NetworkMiner messages.png]]

### Anomalies

- Détection anomalies PCAP
- Pas un IDS
- Détections limitées :
    - EternalBlue exploit
    - Spoofing attempts

![[NetworkMiner anomalies.png]]

---

### Lab t5

#### What is the name of the Linux distro mentioned in the file associated with frame 63075?

![[NetworkMiner t5q1.png]]

#### What is the header of the page associated with frame 75942?

![[NetworkMiner t5q2.png]]


#### What is the source address of the image "ads.bmp.2E5F0FD9[1].bmp"?

![[NetworkMiner t5q3.png]]

#### What is the frame number of the possible TLS anomaly?

![[NetworkMiner t5q4.png]]

#### Look at the messages. Which platform sent an email with the subject starting with "You have more"?

![[NetworkMiner t5q5.png]]

#### What is the email address of Branson Matheson?

![[NetworkMiner t5q6.png]]

---

## Version Differences

### Généralités

- Montée de version = stabilité + sécurité + nouvelles features + optimisations
- Changelog disponible
- VM contient 2 versions : **v1.6** et **v2.7**
- Les versions changent le périmètre d’analyse

### Mac Address Processing

**v2+ (après version 2)**

- Corrélation MAC address
- Détection conflits MAC

![[NetworkMiner Mac address processing.png]]

**v1.6**

- Non supporté

![[NetworkMiner mac address processing 2.png]]

### Packet / Frame Processing

**v1.6**

- Analyse détaillée des packets
- Infos frames + détails complets

**v2.7+**

- Fonction retirée / non disponible

### Frame Processing

**v1.6**

- Gestion des frames
- Nombre de frames + détails associés

**v2.7+**

- Non disponible

![[NetworkMiner Frame processing.png]]

### Parameter Processing

**v2.7+**

- Analyse paramètres plus complète
- Extraction plus large

**v1.6**

- Moins de paramètres détectés

![[NetworkMiner parameter processing.png]]

### Cleartext Data Processing

**v1.6**

- Tab unique cleartext data
- Vue globale du contenu en clair

Limite :

- Impossible de relier cleartext ↔ packets

**v2.7+**

- Fonction supprimée / non disponible

![[NetworkMiner cleardata processing.png]]

---

### Lab

![[NetworkMiner answer t6.png]]

---

## Exercices

#### What is the OS name of the host 131.151.37.122?

![[NetworkMiner t7q1.png]]

#### Investigate the hosts 131.151.37.122 and 131.151.32.91.  How many data bytes were received from host 131.151.32.91 to host 131.151.37.122 through port 1065?

![[NetworkMiner t7q2.png]]

#### Investigate the hosts 131.151.37.122 and 131.151.32.21. How many data bytes were received from hos 131.151.37.122 to host 131.151.32.21 through port 143?

![[NetworkMiner t7q3.png]]

#### What is the sequence number of frame 9?

![[NetworkMiner t7q4.png]]

#### What is the USB product's brand name?

![[NetworkMiner t7q5.png]]

#### What is the name of the phone model?

![[NetworkMiner t7q6.png]]

#### What is the source IP of the fish image? 

![[NetworkMiner t7q7.png]]


#### What is the password of the "homer.pwned[.]se@gmx[.]com"?

![[NetworkMiner t7q8.png]]

#### What is the DNS Query of frame 62001?

![[NetworkMiner t7q9.png]]

---
## Congratulations! You just finished the NetworkMiner room.

In this room, we covered NetworkMiner, what it is, how it operates, and how to investigate pcap files. As I mentioned in the tasks before, there are a few things to remember about the NetworkMiner:

- Don't use this tool as a primary sniffer.
- Use this tool to overview the traffic, then move forward with Wireshark and tcpdump for a more in-depth investigation.

If you like this content, make sure you visit the following rooms later on THM!

- [**Wireshark**](https://tryhackme.com/room/wireshark)
- [**Snort**](https://tryhackme.com/room/snort)
- [**Brim**](https://tryhackme.com/room/brim)