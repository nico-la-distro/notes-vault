## Learning Objectives

### Advanced Port Scan Types

|Scan|Flags|Objectif|
|---|---|---|
|Null|Aucun|Inférer les ports ouverts via absence de réponse|
|FIN|FIN|Sonder sans initier de connexion|
|Xmas|FIN + PSH + URG|Contourner les firewalls sans état|
|Maimon|FIN + ACK|Exploiter un comportement BSD|
|ACK|ACK|Cartographier les règles firewall (pas les ports ouverts)|
|Window|ACK + analyse TCP Window|Différencier open/closed via RST|
|Custom|`--scanflags`|Combinaison personnalisée|

### Evasion and Spoofing Techniques

|Technique|Option|
|---|---|
|Spoof IP|`-S`|
|Spoof MAC|`--spoof-mac`|
|Decoy scan|`-D`|
|Fragmentation|`-f` / `-ff`|
|Idle/Zombie scan|`-sI`|

Verbosité : `--reason`, `-v`, `-vv`, flags debug.

---

## TCP Null Scan, FIN Scan, and Xmas Scan

Ces 3 scans exploitent le comportement RFC : pas de réponse sur port ouvert, RST sur port fermé -> résultat toujours `open|filtered`.

|Scan|Option|Flags envoyés|
|---|---|---|
|Null|`-sN`|Aucun (0)|
|FIN|`-sF`|FIN|
|Xmas|`-sX`|FIN + PSH + URG|

**Logique commune :**

- Port ouvert -> pas de réponse -> `open|filtered`
- Port fermé -> RST reçu -> `closed`
- Impossible de distinguer `open` de `filtered`

```bash
sudo nmap -sN <IP>   # Null
sudo nmap -sF <IP>   # FIN
sudo nmap -sX <IP>   # Xmas
```

**Cas d'usage :** Contourner un firewall **stateless** (qui filtre uniquement les paquets SYN). Un firewall **stateful** bloquera ces paquets -> scans inutiles dans ce cas.

---

## TCP Maimon Scan

- Option : `-sM` | Flags : FIN + ACK
- Cible normalement -> RST
- Certains systèmes BSD : drop le paquet si port ouvert -> révèle les ports ouverts

**Problème :** La plupart des systèmes modernes répondent RST peu importe l'état du port -> `open` et `closed` indiscernables -> scan quasi inutile en pratique.

bash

```bash
sudo nmap -sM <IP>
```

---

## TCP ACK, Window, and Custom Scan

### TCP ACK Scan

- Option : `-sA` | Flag : ACK
- La cible répond toujours RST (port ouvert ou fermé) -> **ne détecte pas les ports ouverts**
- **Objectif réel : cartographier les règles firewall**
    - Sans firewall -> tous les ports `unfiltered`
    - Avec firewall -> seuls les ports non bloqués répondent (`unfiltered`)

bash

```bash
sudo nmap -sA <IP>
```

### Window Scan

- Option : `-sW` | Quasi-identique à ACK, mais analyse le champ **TCP Window** du RST retourné
- Avec firewall -> les ports non bloqués apparaissent `closed` (au lieu de `unfiltered` avec ACK) -> comportement différent = non bloqué par le firewall

bash

```bash
sudo nmap -sW <IP>
```

| |Sans firewall|Avec firewall (ports autorisés)|
|---|---|---|
|ACK `-sA`|`unfiltered`|`unfiltered`|
|Window `-sW`|`closed`|`closed`|

> ACK et Window révèlent les **règles firewall**, pas les services. Un port non bloqué ne signifie pas qu'un service écoute dessus.

### Custom Scan

Combinaison de flags personnalisée :

bash

```bash
sudo nmap --scanflags RSTSYNFIN <IP>
```

Les noms de flags sont concaténés directement (`SYN`, `ACK`, `FIN`, `RST`, `PSH`, `URG`).

---

## Spoofing and Decoys

### Spoofing IP

Condition : l'attaquant doit pouvoir **capturer les réponses** (même réseau ou accès au trafic) -> sinon scan inutile.

```bash
sudo nmap -e NET_INTERFACE -Pn -S SPOOFED_IP <IP_CIBLE>
```

- `-S` -> source IP forgée
- `-e` -> interface réseau à utiliser
- `-Pn` -> désactive le ping (pas de réponse attendue vers l'attaquant)

### Spoofing MAC

Uniquement si attaquant et cible sont sur le **même réseau Ethernet/WiFi** :

```bash
--spoof-mac SPOOFED_MAC
```

### Decoy Scan

Noie la vraie IP parmi plusieurs leurres -> les réponses partent aussi vers les decoys.

```bash
# IP fixes + position ME explicite
nmap -D 10.10.0.1,10.10.0.2,ME <IP_CIBLE>

# IP aléatoires + position ME
nmap -D 10.10.0.1,10.10.0.2,RND,RND,ME <IP_CIBLE>
```

- `ME` -> position de la vraie IP dans la liste
- `RND` -> IP aléatoire générée à chaque exécution
- Sans `ME` -> Nmap place la vraie IP aléatoirement

---

## Fragmented Packets

### Firewall / IDS

- **Firewall** -> filtre sur headers IP + transport (parfois données)
- **IDS** -> inspecte aussi les données transport, détecte patterns/signatures malveillants

### Fragmented Packets

Diviser les paquets pour échapper aux firewalls/IDS traditionnels.

|Option|Taille des fragments|
|---|---|
|`-f`|8 bytes max|
|`-ff` / `-f -f`|16 bytes max|
|`--mtu NUM`|Personnalisé (multiple de 8 obligatoire)|

```bash
sudo nmap -sS -p80 -f <IP>       # fragments de 8 bytes
sudo nmap -sS -p80 -ff <IP>      # fragments de 16 bytes
sudo nmap -sS --mtu 32 <IP>      # fragments de 32 bytes
```

Exemple concret : TCP header = 24 bytes

- Avec `-f` -> 3 fragments de 8 bytes
- Avec `-ff` -> 2 fragments (16 + 8 bytes)

### Padding

Pour faire paraître les paquets innocents en augmentant leur taille :

```bash
--data-length NUM    # ajoute NUM bytes de données au paquet
```

---

## Idle/Zombie Scan

Scan via un hôte tiers **inactif** (zombie) -> la vraie IP de l'attaquant n'apparaît jamais.

bash

```bash
nmap -sI ZOMBIE_IP <IP_CIBLE>
```

**Prérequis :** le zombie doit être réellement idle (sinon les IP ID sont inutilisables).

#### Mécanisme (3 étapes)

1. Envoyer SYN/ACK au zombie -> récupérer son IP ID actuel (zombie répond RST)
2. Envoyer SYN à la cible avec **IP source = zombie** -> la cible répond au zombie
3. Envoyer SYN/ACK au zombie -> comparer le nouvel IP ID

#### Interprétation

|Delta IP ID|Signification|
|---|---|
|+1|Port **fermé ou filtré** (zombie n'a pas reçu de SYN/ACK)|
|+2|Port **ouvert** (zombie a reçu SYN/ACK -> a répondu RST -> IP ID +1, puis +1 pour l'étape 3)|

#### Pourquoi ça fonctionne

- Port ouvert -> cible envoie SYN/ACK au zombie -> zombie répond RST (inattendu) -> **IP ID +1**
- Port fermé -> cible envoie RST au zombie -> zombie ignore -> **IP ID inchangé**
- Port filtré -> pas de réponse -> même résultat que port fermé

---

## Getting More Details

|Option|Effet|
|---|---|
|`--reason`|Explique pourquoi un port est open/closed et pourquoi l'hôte est up|
|`-v`|Verbose|
|`-vv`|Plus verbose|
|`-d`|Debug (output très long)|
|`-dd`|Debug maximal|

bash

```bash
sudo nmap -sS --reason <IP>
sudo nmap -sS -vv <IP>
sudo nmap -sS -d <IP>
```

---

## Summary

### Scan Types

|Scan|Commande|
|---|---|
|Null|`sudo nmap -sN <IP>`|
|FIN|`sudo nmap -sF <IP>`|
|Xmas|`sudo nmap -sX <IP>`|
|Maimon|`sudo nmap -sM <IP>`|
|ACK|`sudo nmap -sA <IP>`|
|Window|`sudo nmap -sW <IP>`|
|Custom|`sudo nmap --scanflags URGACKPSHRSTSYNFIN <IP>`|
|Spoof IP|`sudo nmap -S SPOOFED_IP <IP>`|
|Spoof MAC|`--spoof-mac SPOOFED_MAC`|
|Decoy|`nmap -D DECOY_IP,ME <IP>`|
|Zombie|`sudo nmap -sI ZOMBIE_IP <IP>`|
|Fragments 8B|`-f`|
|Fragments 16B|`-ff`|

### Options utiles

|Option|Rôle|
|---|---|
|`--source-port PORT`|Forcer un port source|
|`--data-length NUM`|Ajouter des données aléatoires au paquet|
|`--reason`|Explique les conclusions de Nmap|
|`-v` / `-vv`|Verbose|
|`-d` / `-dd`|Debug|

#### Rappel comportement

- **Null, FIN, Xmas** -> réponse uniquement des ports **fermés** (RST)
- **Maimon, ACK, Window** -> réponse des ports **ouverts et fermés**

