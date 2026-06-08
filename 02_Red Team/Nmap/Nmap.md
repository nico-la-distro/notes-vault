
## Flags/Switches

| Option                                                              | Explanation                                                                                        |
| ------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------- |
| `-sL`                                                               | List scan – list targets without scanning                                                          |
| **_Host Discovery_**                                                |                                                                                                    |
| `-sn`                                                               | Ping scan – host discovery only                                                                    |
| **_Port Scanning_**                                                 |                                                                                                    |
| `-sT`                                                               | TCP connect scan – complete three-way handshake                                                    |
| `-sS`                                                               | TCP SYN – only first step of the three-way handshake                                               |
| `-sU`                                                               | UDP Scan                                                                                           |
| `-F`                                                                | Fast mode – scans the 100 most common ports                                                        |
| `-p[range]`                                                         | Specifies a range of port numbers – `-p-` scans all the ports                                      |
| `-Pn`                                                               | Treat all hosts as online – scan hosts that appear to be down                                      |
| **_Service Detection_**                                             |                                                                                                    |
| `-O`                                                                | OS detection                                                                                       |
| `-sV`                                                               | Service version detection                                                                          |
| `-A`                                                                | OS detection, version detection, and other additions                                               |
| **_Timing_**                                                        |                                                                                                    |
| `-T<0-5>`                                                           | Timing template – paranoid (0), sneaky (1), polite (2), normal (3), aggressive (4), and insane (5) |
| `--min-parallelism <numprobes>` and `--max-parallelism <numprobes>` | Minimum and maximum number of parallel probes                                                      |
| `--min-rate <number>` and `--max-rate <number>`                     | Minimum and maximum rate (packets/second)                                                          |
| `--host-timeout`                                                    | Maximum amount of time to wait for a target host                                                   |
| **_Real-time output_**                                              |                                                                                                    |
| `-v`                                                                | Verbosity level – for example, `-vv` and `-v4`                                                     |
| `-d`                                                                | Debugging level – for example `-d` and `-d9`                                                       |
| **_Report_**                                                        |                                                                                                    |
| `-oN <filename>`                                                    | Normal output                                                                                      |
| `-oX <filename>`                                                    | XML output                                                                                         |
| `-oG <filename>`                                                    | `grep`-able output                                                                                 |
| `-oA <basename>`                                                    | Output in all major formats                                                                        |

---

## Exemple firewall qui renvoie `RST` (impact : faux “closed”)

```bash
iptables -I INPUT -p tcp --dport <port> -j REJECT --reject-with tcp-reset
```

➡️ Effet : au lieu de “filtered”, tu peux voir **closed**, donc **scan moins fiable**.

---
## SYN scan `-sS` (Half-open / “Stealth”) — l’essentiel

- Scanne des **ports TCP** comme `-sT`, mais **sans établir la connexion complète**.
    
- Sur **port ouvert** : Nmap envoie `SYN` → reçoit `SYN/ACK` → **répond `RST`** (au lieu de `ACK`) ⇒ connexion **jamais établie**.

---
### Séquences (open / closed / filtered)

|Cas|Échanges|Verdict Nmap|
|---|---|---|
|**Open**|`SYN` → `SYN/ACK` → **`RST`**|**open**|
|**Closed**|`SYN` → `RST`|**closed**|
|**Filtered**|`SYN` → _(rien)_ **ou** `RST` “spoofé” par firewall|**filtered** (souvent) / lecture parfois ambiguë|
Même logique que `-sT` pour **closed/filtered** ; la vraie différence = **gestion des ports open**

---
### Avantages (à retenir)

- **Peut contourner des IDS anciens** (car ils surveillent surtout le handshake complet) → “stealth” (moins vrai aujourd’hui).
    
- **Moins loggé par l’appli** (souvent log uniquement après connexion établie).
    
- **Plus rapide** que `-sT` (pas de handshake complet à terminer/fermer pour chaque port).    

---
### Inconvénients

- Nécessite en général **sudo/root** sous Linux (création de **raw packets**).
    
    - Alternative : capabilities `CAP_NET_RAW`, `CAP_NET_ADMIN`, `CAP_NET_BIND_SERVICE`  
        ⚠️ mais peut **casser / limiter** l’exécution de certains scripts NSE.
        
- Peut **faire tomber des services fragiles** (risque en prod).

---
### Defaults Nmap (comportement important)

|Contexte|Scan par défaut|
|---|---|
|Nmap lancé **avec sudo**|`-sS` (SYN scan)|
|Nmap lancé **sans sudo**|`-sT` (TCP Connect)|

---

## UDP scan `-sU` — l’essentiel

- **UDP = stateless** : pas de handshake, on envoie des paquets et on “espère” une réponse → **scan plus dur et beaucoup plus lent**.
    
- Switch Nmap : **`-sU`**
    

---

### Interprétation des réponses UDP

|Ce que Nmap envoie|Réponse attendue|Interprétation|
|---|---|---|
|Paquet UDP vers port **open**|**souvent rien**|**open\|filtered** (open ou filtré firewall)|
|Paquet UDP vers port **open**|Réponse UDP (rare)|**open**|
|Paquet UDP vers port **closed**|ICMP “port unreachable”|**closed**|

**Comportement Nmap** (cas “rien”) :

- Pas de réponse → **renvoie une 2e fois** (double-check)
    
- Toujours rien → reste en **open|filtered** et passe au suivant
    

---

### Pourquoi c’est lent (à retenir)

- Comme “pas de réponse” peut vouloir dire **open** ou **filtré**, Nmap attend / retente → **temps énorme**
    
- Ordre de grandeur cité : **~20 min pour les 1000 premiers ports** (bonne connexion)
    

---

### Bonne pratique : réduire la surface scannée

|Option|But|Exemple|
|---|---|---|
|`--top-ports <n>`|Scanner seulement les ports UDP les plus courants|`nmap -sU --top-ports 20 <target>`|

---

### Détail utile

- En UDP, Nmap envoie souvent des **paquets vides** (raw UDP).
    
- Pour certains ports “connus”, il envoie un **payload spécifique au protocole** (pour provoquer une réponse et être plus fiable).

---

## NULL / FIN / Xmas scans — résumé utile

Ces 3 scans TCP sont **moins courants**, mais parfois utilisés car **encore plus “stealth”** que `-sS`, surtout pour **évasion de firewall** (ancienne génération).

---

### Différences entre les 3

|Scan|Switch|Flags envoyés|Idée|
|---|---|---|---|
|NULL|`-sN`|_(aucun flag)_|paquet “vide”|
|FIN|`-sF`|`FIN`|simule une fermeture|
|Xmas|`-sX`|`PSH + URG + FIN`|paquet “sapin de Noël” (visible en capture)|

---

### Réponses attendues (logique identique pour les 3)

|Réponse de la cible|Interprétation Nmap|Commentaire|
|---|---|---|
|`RST`|**closed**|attendu si port fermé (RFC 793)|
|Rien|**open\|filtered**|port ouvert **ou** filtré (ambigu)|
|ICMP unreachable|**filtered**|souvent la raison d’un “filtered” ici|

➡️ Comme l’UDP : **“pas de réponse” ≠ forcément open** → souvent **open|filtered**.

---

### Limites / pièges importants

- **Pas toujours fiable en pratique** :
    
    - **Windows** (et beaucoup d’équipements **Cisco**) peuvent renvoyer `RST` **pour tout paquet malformé**, même si le port est ouvert → résultat : **tout apparaît “closed”**.
        
- **Evasion firewall** (principe) :
    
    - Beaucoup de firewalls bloquent surtout les paquets avec **`SYN`** (init de connexion).
        
    - Ici on envoie des paquets **sans `SYN`** → peut **bypasser** ce type de filtrage.
        
- **Mais** : la plupart des **IDS modernes** détectent ces scans → ne pas compter dessus comme solution “magique”.

---
## ICMP network scanning / Ping sweep (Nmap)

- **Objectif (black box)** : faire une **carte du réseau** → repérer les **hôtes actifs** (IP “alive”) vs inactifs.
    
- Méthode : **ping sweep** → Nmap envoie des paquets (principalement ICMP) sur une plage d’IP et marque **alive** si réponse.
    
- **Limite** : pas toujours fiable (sera expliqué plus tard), mais bon **baseline**.
    

---
### Commande & formats de plage

|But|Commande|
|---|---|
|Scanner 192.168.0.1 à .254|`nmap -sn 192.168.0.1-254`|
|Scanner un /24|`nmap -sn 192.168.0.0/24`|

---
### Ce que fait `-sn` (à retenir)

|Point|Détail|
|---|---|
|“No port scan”|`-sn` **désactive le scan de ports**|
|Détection “alive” principale|**ICMP echo** (ping)|
|Cas réseau local + sudo/root|Utilise aussi des **requêtes ARP** (souvent plus fiable en LAN)|
|Bonus probes envoyés|En plus d’ICMP : envoie **TCP SYN sur 443** + **TCP ACK sur 80** (ou **TCP SYN sur 80** si pas root)|

_(Donc même en “ping sweep”, Nmap peut envoyer un peu de TCP pour améliorer la détection.)_

---
## NSE (Nmap Scripting Engine) — aperçu

- **NSE = extension majeure de Nmap** : ajoute énormément de fonctionnalités.
    
- Scripts écrits en **Lua**.
    
- Usages : **recon/énumération**, **scan de vulnérabilités**, et parfois **automatisation d’exploits** → bibliothèque très large.
    

---
### Catégories utiles (à mémoriser)

|Catégorie|Rôle|
|---|---|
|`safe`|N’affecte pas la cible (faible risque)|
|`intrusive`|Peut impacter la cible (risqué)|
|`vuln`|Détecte des vulnérabilités|
|`exploit`|Tente d’exploiter une vulnérabilité|
|`auth`|Test/bypass d’auth (ex : FTP anonymous)|
|`brute`|Bruteforce d’identifiants|
|`discovery`|Récupère des infos via services (ex : SNMP)|

more list : https://nmap.org/book/nse-usage.html

---

## Utiliser NSE avec `--script` (notes rapides)

- `--script=<catégorie>` lance les scripts de cette catégorie **qui s’appliquent aux services détectés** (pas “tout” au hasard).
    
    - Ex : `--script=vuln`, `--script=safe`, etc.
        
- `--script=<nom-du-script>` pour un script précis.
    
- Plusieurs scripts : séparés par **virgules**.
    

---

### Syntaxe & exemples (table)

|Objectif|Exemple|
|---|---|
|Lancer une catégorie|`nmap --script=safe <target>`|
|Lancer 1 script|`nmap --script=http-fileupload-exploiter <target>`|
|Lancer plusieurs scripts|`nmap --script=smb-enum-users,smb-enum-shares <target>`|

---

### `--script-args` (arguments de scripts)

- Certains scripts demandent des paramètres (ex : creds, URL, chemin fichier…)
    
- Format : `--script-args script.arg=value,script.arg=value`
    
    - **virgules** entre arguments
        
    - liaison script↔argument via **point** : `<script-name>.<argument>`
        

**Exemple (http-put)** :

nmap -p 80 --script http-put --script-args http-put.url='/dav/shell.php',http-put.file='./shell.php'

---

### Aide locale

|Besoin|Commande|
|---|---|
|Voir l’aide d’un script|`nmap --script-help <script-name>`|

_(utile en local, mais souvent moins complet que la doc/liste en ligne des scripts + arguments + use-cases.)_

Full list : https://nmap.org/nsedoc/

---
## Trouver des scripts NSE (local + web)

- **2 sources à combiner** :
    
    1. **Liste officielle** sur le site Nmap (tous les scripts “officiels”) https://nmap.org/nsedoc/
        
    2. **Scripts installés localement** (Linux) : `/usr/share/nmap/scripts/`  
        → c’est là que Nmap va chercher quand tu fais `--script=...`
        

---
### Rechercher des scripts installés (2 méthodes)

|Méthode|Où|Commande|Usage|
|---|---|---|---|
|`script.db` (fichier texte index)|`/usr/share/nmap/scripts/script.db`|`grep "ftp" /usr/share/nmap/scripts/script.db`|trouver scripts + catégories|
|Wildcards `ls`|dossier scripts|`ls -l /usr/share/nmap/scripts/*ftp*`|chercher par nom (pattern)|

Astuce : `*mot*` = wildcard des deux côtés pour matcher “contient mot”.

---

### Rechercher par catégorie

|But|Exemple|
|---|---|
|Lister scripts d’une catégorie|`grep "safe" /usr/share/nmap/scripts/script.db`|

---

### Installer / mettre à jour des scripts

- Si un script “officiel” manque localement :
    
    - solution simple : `sudo apt update && sudo apt install nmap`
        
    - ou **manuel** : télécharger le `.nse` dans le bon dossier, puis **réindexer**.
        

|Étape|Commande|
|---|---|
|Télécharger un script|`sudo wget -O /usr/share/nmap/scripts/<script-name>.nse https://svn.nmap.org/nmap/scripts/<script-name>.nse`|
|Mettre à jour l’index|`nmap --script-updatedb`|

⚠️ `--script-updatedb` est aussi nécessaire si tu **ajoutes ton propre script** (Lua) dans le dossier.

---
## Bypass firewall ICMP (cas Windows) : `-Pn`

- **Problème courant** : le **firewall Windows par défaut bloque l’ICMP** → `ping` ne répond pas.
    
- **Impact** : Nmap **ping** par défaut avant de scanner → peut marquer l’hôte comme **dead** et **ne pas le scanner**.
    
- **Solution** : `-Pn` = **pas de host discovery** (pas de ping avant scan)  
    → Nmap considère la cible **alive** et scanne quand même.
    

**Trade-off** :

- si l’hôte est réellement down → scan peut être **très long** (Nmap tente/retente les ports).
    

**Note LAN** :

- Sur le **réseau local**, Nmap peut utiliser **ARP** pour détecter les hôtes (plus fiable que l’ICMP quand dispo).
    

---

### Options utiles “evasion” (à connaître)

|Switch|Effet|Pourquoi ça aide|
|---|---|---|
|`-Pn`|Ignore le ping préalable|Bypass blocage ICMP / host discovery|
|`-f`|**Fragmente** les paquets|Moins détectable par firewall/IDS (parfois)|
|`--mtu <n>`|Contrôle taille fragments (multiple de 8)|Variante plus fine de `-f`|
|`--scan-delay <time>ms`|Délai entre paquets|Évite triggers “rate/time-based”, aide réseau instable|
|`--badsum`|Checksum **invalide**|Les hôtes drop, mais un firewall/IDS peut répondre → **détecter présence** firewall/IDS|

_(Les autres options existent, mais celles-ci reviennent souvent en pratique.)_

