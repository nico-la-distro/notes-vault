## Network Discovery
### Attackers and Network Discovery

Objectif : cartographier la surface d'attaque exposée sur internet. Cibles : IP, ports, OS, services, versions → trouver une vulnérabilité exploitable.

### Defenders and Network Discovery

Objectif : réduire la surface d'attaque. Actions : inventorier les assets, fermer les ports/services inutiles, patcher les vulnérabilités.

### The Challenge in Detecting Network Discovery

Problème : attaquants, défenseurs, crawlers et moteurs de recherche font les mêmes actions → difficile de distinguer le bon du mauvais scan.

Techniques SOC pour différencier :

- **Allowlist** les scanners internes et externes connus (pas d'alertes sur ces sources)
- **Threat Intelligence** → alerter uniquement sur les sources malveillantes/suspectes connues
- Pour compenser les angles morts du point précédent : TI utilisée pour **élever la sévérité** des alertes + règles génériques sur les comportements de scan (détaillées dans les tâches suivantes)

---

## External vs Internal Scanning

|Type|Source IP|Destination IP|Phase MITRE ATT&CK|Sévérité|
|---|---|---|---|---|
|External|IP externe|IP org (périmètre)|Reconnaissance|Faible|
|Internal|IP privée|IP privée (interne)|Discovery|Élevée|

**External** → attaquant sans foothold, cherche un accès initial. Réponse : bloquer l'IP sur le firewall périmètre (limité : l'attaquant peut changer d'IP). https://attack.mitre.org/tactics/TA0043/

![[network discovery reconnaissance.png]]

**Internal** → attaquant déjà dans le réseau, prépare le lateral movement. Réponse : escalade + Incident Response + investigation approfondie + root cause analysis. Bloquer l'IP seul est insuffisant. https://attack.mitre.org/tactics/TA0007/

![[network discovery discovery.png]]

### Identifying Internal and External Scanning

bash

```bash
head -n2 log-session-1.csv        # aperçu du fichier
```

```bash
cut -d ',' -f <N> log-session-1.csv # filtrer une colonne (attention : la date contient une virgule → décaler les indices)
```

Champs clés dans les logs Zeek :

| Champ                              | Description                                        |
| ---------------------------------- | -------------------------------------------------- |
| `source.ip` / `id.orig_h`          | IP source                                          |
| `destination.ip` / `id.resp_h`     | IP destination                                     |
| `source.port` / `destination.port` | Ports                                              |
| `network.protocol` / `proto`       | Protocole                                          |
| `conn_state`                       | État de la connexion (ex: `S0` = SYN sans réponse) |
### Question : How many log entries are present for the internal IP performing internal scanning activity?

step 1 : check how the file is construct

![[network discovery t3q2.png]]
see that the ip source is in 2nd column, and there is a comma. so the command to answer this question is :

```bash
cat log-session-0.csv | cut -d ',' -f 3 | uniq -c

  1 "source.port"
   2276 "192.168.230.127"
```

This work because it's the same source ip all over this file

let's dissect this command :

```bash
cat log-session-0.csv        # lit le fichier

| cut -d ',' -f 3            # découpe par virgule, garde uniquement le 3e champ

| uniq -c                    # compte les lignes identiques CONSÉCUTIVES et affiche le nombre devant
```

**Answer : 2276**

---

## Horizontal vs Vertical Scanning

|Type|Source IP|Destination IP|Destination Port|Objectif|
|---|---|---|---|---|
|Horizontal|fixe|multiple|fixe|Trouver quels hosts exposent un port spécifique|
|Vertical|fixe|fixe|multiple|Cartographier les services d'un seul host|
|Mixed|fixe|multiple|multiple|Les deux à la fois|

**Horizontal** → exemple : WannaCry scannait le port 445 (SMB) sur tout le réseau.

![[network discovery horizontal scanning.png]]

**Vertical** → exemple : serveur unique exposé sur internet, l'attaquant scanne tous ses ports avant exploitation.

![[network discovery vertical scanning.png]]

**Détection dans les logs :**

- Horizontal : même `source.ip` + même `destination.port` + `destination.ip` qui varie
- Vertical : même `source.ip` + même `destination.ip` + `destination.port` qui varie

### Questions
#### One of the log files contains evidence of a horizontal scan. Which IP range was scanned? Format X.X.X.X/X

I was actualy serching for question 1 with this command :

```bash
cat log-session-0.csv | cut -d ',' -f 5 | uniq -c

      1 "destination.port"
   2013 "192.168.230.145"
      1 "239.255.255.250"

```

I wanted to search for destination ip pattern and see just 1 ip in this log file. 
So the ip address 192.168.230.145 is the answer of the question 2 "In the same log file, there is one IP address on which a vertical scan is performed. Which IP address is this?"

Let's continue to search for horizontal scan.

With the same previous command but in log-session-2 we see all the /24 range of 203.0.113.0.

```bash
cat log-session-2.csv | cut -d ',' -f 5 | uniq -c
```

**Answer (Q1) : 203.0.113.0/24**

**Answer (Q2) : 192.168.230.145**

---

#### On one of the IP addresses, only a few ports are scanned which host common services. Which are the ports that are scanned on this IP address? Format: port1, port2, port3 in ascending order.

i had to check on internet bruuh this is the command :

```bash
grep ',"192.168.230.1",' log-session-2.csv | cut -d',' -f6

3389
445
3389
445
80
0
"-"
"-"
0
```

let's dissect :

```bash
grep ',"192.168.230.1",' log-session-2.csv    # filtre les lignes contenant cette IP
| cut -d',' -f6                                # extrait la 6e colonne
```

**`grep ',"192.168.230.1",'`** Cherche les lignes où `192.168.230.1` est entouré de virgules → évite les faux positifs comme `192.168.230.10` ou `192.168.230.100`.

**`cut -d',' -f6`** Découpe par `,` et garde le 6e champ → probablement le port de destination, pour voir quels ports cette IP a scannés.

**Answer : 80, 445, 3389** (http/smb/rdp)

---

## The Mechanics of Scanning

|Type|Mécanisme|Port ouvert|Port fermé|Fiabilité|
|---|---|---|---|---|
|Ping Sweep|ICMP request → ICMP reply|Host en ligne|Pas de réponse|Souvent bloqué|
|TCP SYN|SYN → SYN-ACK|SYN-ACK reçu|RST reçu|Fiable, furtif|
|UDP|Paquet UDP vide|UDP reply (rare) ou silence|ICMP port unreachable|Lent, peu fiable|

**TCP SYN** : n'établit pas la connexion complète (pas d'ACK final) → se fond dans le trafic normal, difficile à détecter.

**UDP** : timeout = port possiblement ouvert (pas une preuve). Lent car dépend de l'expiration du timer.

**Bonne pratique SOC** : les scans internes légitimes (vuln management, inventaire) doivent être **exclus des règles de détection** pour réduire le bruit → nécessite de connaître les IP sources, types de scans et plannings.

### Identifying Scan Types

Analyse via **Kibana** (`http://10.129.128.67:5601`, `elastic` / `changeme`).

Navigation : hamburger menu → Discover → Data View : `All logs` → `Search entire time range`.

- `+` sur un champ → l'ajoute en colonne
- Clic sur une valeur → filtre inclusion/exclusion

### Questions
#### Which source IP performs a ping sweep attack across a whole subnet?

![[network discovery t5q1.png]]

**Answer 192.168.230.137**

#### The zeek.conn.conn_state value shows the connection state. Using the information provided by this value, identify the type of scan being performed by 203.0.113.25 against 192.168.230.145

![[network discovery t5q2.png]]

**Answer : TCP SYN Scan**

#### Is there any UDP scanning attempt in the logs? Y/N

**Answer : N**

---

## Tools & Commands Used

bash

```bash
head -n2 log-session-*.csv                                    # aperçu tous les fichiers d'un coup
cat log-session-2.csv | cut -d',' -f3 | uniq -c              # compter les occurrences
cut -d',' -f2,4,5 log-session-2.csv | tr ',' ' ' > simple-scan.txt  # nettoyer le CSV
grep ',"192.168.230.1",' log-session-2.csv | cut -d',' -f6   # filtrer une IP et extraire un champ
```

### What I Learned

**Logs réels** : JSON imbriqué dans du CSV, guillemets, champs variables → rarement propres. Combiner `head`, `grep`, `cut`, `tr` est la méthode standard pour les rendre exploitables.

**CLI vs SIEM** :

- CLI → analyse approfondie sur logs bruts
- SIEM → triage rapide, reconnaissance visuelle des patterns

