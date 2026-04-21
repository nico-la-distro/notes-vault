## Statistics Summary

- Donner une **vue globale du trafic**
- Aider à **formuler une hypothèse d’analyse**
- Fournit :
    - Protocoles
    - Endpoints
    - Conversations
    - Détails (DNS, DHCP, HTTP/2…)

### 📊 **Options principales**

#### 1. Resolved Addresses

- Associe **IP ↔ noms DNS (hostnames)**
- Basé sur les **réponses DNS du capture**
- Permet :
    - Identifier les ressources accédées
    - Évaluer leur pertinence

👉 Menu : `Statistics → Resolved Addresses`

![[wireshark pecket operations resolved addresses.png]]

#### 2. Protocol Hierarchy

- Vue en **arborescence des protocoles**
- Affiche :
    - Nombre de paquets
    - Pourcentages
- Permet :
    - Comprendre l’usage global du trafic
    - Cibler un protocole suspect

👉 Astuce : clic droit → filtrer

👉 Menu : `Statistics → Protocol Hierarchy`

![[wireshark packet operations protocol hierarchy.png]]
#### 3. Conversations

- Trafic entre **2 endpoints**
- Formats :
    - Ethernet, IPv4, IPv6, TCP, UDP
- Permet :
    - Identifier communications spécifiques

👉 Menu : `Statistics → Conversations`

![[wireshark packet operations conversations.png]]

#### 4. Endpoints

- Liste des **points uniques** (pas les paires)
- Même formats que Conversations
- Permet :
    - Identifier tous les hôtes impliqués

👉 Menu : `Statistics → Endpoints`

![[wireshark packet operations endpoints.png]]

### 🔁 **Différence clé**

|Feature|Description|
|---|---|
|Conversations|Communication entre **2 entités**|
|Endpoints|Liste des **entités uniques**|

### 🏷️ **Name Resolution**

#### 🔹 Types supportés :

- MAC → constructeur (IEEE)
- IP → nom DNS
- Port → nom service

#### ⚙️ Activation :

- MAC : bouton direct dans Endpoints
- IP & Port :  
    `Edit → Preferences → Name Resolution`

![[wireshark packet opretations name resolution.png]]

![[wireshark packet operations endpoints name resolution.png]]

💡 Intérêt :

- Lecture **plus humaine** des données

### 🌍 **GeoIP Mapping**

- Associe IP → **localisation géographique**
- Nécessite :
    - Base **MaxMind**
    - Configuration manuelle

👉 Chemin :  
`Edit → Preferences → Name Resolution → MaxMind DB`

![[wireshark packet operations maxmind db.png]]

![[wireshark packet operations geoip.png]]

---

## Statistics Protocol Details

- Analyser **en détail un protocole spécifique**
- Isoler les données utiles pour une investigation

### 🌐 **IPv4 / IPv6 Statistics**

- Filtre les paquets selon **version IP**
- Permet :
    - Isoler trafic **IPv4 ou IPv6**
    - Analyser événements liés à une version précise

👉 Menu : `Statistics → IPvX Statistics`

![[wireshark packet operations ipv4 and ipv6 statistics.png]]

### 🌍 **DNS Statistics**

- Analyse détaillée du trafic DNS
- Vue en **arborescence + statistiques**

![[wireshark packet operations dns statistics.png]]

📊 Infos disponibles :

- RCODE (réponse DNS)
- OPCODE
- Classe
- Type de requête
- Services
- Stats globales DNS

🎯 Utilité :

- Comprendre activité DNS
- Détecter anomalies (ex : erreurs, requêtes suspectes)

👉 Menu : `Statistics → DNS`

### 🌐 **HTTP Statistics**

- Analyse détaillée du trafic HTTP
- Vue en **arborescence + statistiques**

![[wireshark packet operations http statistics.png]]

📊 Infos disponibles :

- Requêtes HTTP
- Codes de réponse (200, 404, etc.)
- Requêtes originales

🎯 Utilité :

- Comprendre navigation web
- Identifier comportements suspects

👉 Menu : `Statistics → HTTP`

---

## Packet Filtering Principles

- Réduire et analyser le trafic réseau
- Identifier les paquets liés à un **événement précis**

### ⚖️ **Types de filtres**

|Type|Description|Moment d’utilisation|Modifiable ?|
|---|---|---|---|
|Capture Filter|Filtre **avant capture** (enregistre seulement certains paquets)|Avant capture|❌ Non|
|Display Filter|Filtre **après capture** (affiche certains paquets)|Pendant/après|✅ Oui|

⚠️ À retenir

- ❌ Non interchangeables
- ✅ Bonne pratique : **capturer tout → filtrer ensuite**
- ⚠️ Capture filter risqué (tu peux rater du trafic)

### 🧩 **Capture Filters**

- Syntaxe **complexe (bas niveau)**
- Basée sur :
    - offsets, hex, masks

🧱 Structure

|Élément|Options|
|---|---|
|Scope|host, net, port, portrange|
|Direction|src, dst, src/dst|
|Protocol|ether, ip, tcp, udp…|

📌 Exemple

tcp port 80

![[wireshark packet operations capture filter.png]]

### 🧠 **Display Filters (IMPORTANT)**

📌 Caractéristiques

- Feature **la plus puissante**
- Supporte **3000+ protocoles**
- Filtrage très précis

📌 Exemple

tcp.port == 80

![[wireshark packet operations display filter.png]]

### ⚖️ **Opérateurs de comparaison**

|Nom|Symbole|Exemple|Description|
|---|---|---|---|
|eq|==|ip.src == 10.10.10.100|égal|
|ne|!=|ip.src != 10.10.10.100|différent|
|gt|>|ip.ttl > 250|supérieur|
|lt|<|ip.ttl < 10|inférieur|
|ge|>=|ip.ttl >= 0xFA|≥|
|le|<=|ip.ttl <= 0xA|≤|

⚠️ Note

- Support **décimal + hexadécimal**
- ❗ `!=` déconseillé → préférer `!(...)`

### 🔗 **Opérateurs logiques**

![[wireshark packet operations logical operators.png]]

### 🎛️ **Filter Toolbar (Display)**

|Couleur|Signification|
|---|---|
|🟢 Vert|Valide|
|🔴 Rouge|Invalide|
|🟡 Jaune|Fonctionne mais déconseillé|
![[wireshark packet operations toolbar colors.png]]

💡 Astuces

- Toujours en **minuscule**
- **Autocomplete** avec `.`
- Aide à construire des filtres valides

![[wireshark packet operations toolbar features.png]]

### 🧠 **À retenir

- **Capture filter = avant (risqué)**
- **Display filter = après (flexible, recommandé)**
- **Syntaxe différente → non interchangeable**
- **Display filters = outil principal d’analyse**
- Utiliser :
    - `==`, `>`, `<`
    - `AND`, `OR`, `NOT`
- Toolbar = aide visuelle + validation

---

## Packet Filtering Protocol Filters

- Filtrer selon les **couches du modèle OSI**
- Analyser :
    - Réseau (IP)
    - Transport (TCP/UDP)
    - Application (HTTP/DNS)

### 🌐 **1. IP Filters (Layer 3)**

📌 Permet de filtrer :

- Adresse IP
- Version
- TTL, flags, checksum…

📊 Filtres courants

|Filtre|Description|
|---|---|
|`ip`|Tous les paquets IP|
|`ip.addr == X`|IP source **ou** destination|
|`ip.src == X`|IP source|
|`ip.dst == X`|IP destination|
|`ip.addr == X/24`|Sous-réseau|

![[wireshark packet operations ip filters.png]]

⚠️ À retenir

- `ip.addr` = **bidirectionnel**
- `ip.src / ip.dst` = **directionnel**

### 🚚 **2. TCP / UDP Filters (Layer 4)**

📌 Permet de filtrer :

- Ports
- Flags
- Seq / Ack
- Erreurs

📊 Filtres courants

|Filtre|Description|
|---|---|
|`tcp.port == 80`|TCP port 80|
|`udp.port == 53`|UDP port 53|
|`tcp.srcport == X`|Port source TCP|
|`tcp.dstport == X`|Port destination TCP|
|`udp.srcport == X`|Port source UDP|
|`udp.dstport == X`|Port destination UDP|

### 🌍 **3. HTTP / DNS Filters (Layer 7)**

📌 Permet de filtrer :

- Contenu applicatif
- Requêtes / réponses

📊 Filtres courants

|Filtre|Description|
|---|---|
|`http`|Tous les paquets HTTP|
|`dns`|Tous les paquets DNS|
|`http.response.code == 200`|Réponses OK|
|`http.request.method == "GET"`|Requêtes GET|
|`http.request.method == "POST"`|Requêtes POST|
|`dns.flags.response == 0`|Requêtes DNS|
|`dns.flags.response == 1`|Réponses DNS|
|`dns.qry.type == 1`|Enregistrements A|

![[wireshark packet operations http dns filters.png]]

### 🛠️ **Display Filter Expressions**

📌 Fonction

- Aide à construire des filtres
- Liste :
    - Tous les protocoles
    - Champs disponibles
    - Types de valeurs

👉 Menu : `Analyse → Display Filter Expression`

💡 Pourquoi utile ?

- Impossible de tout mémoriser
- Sert de **guide interactif**

![[wireshark packet operations display filter expression.png]]

### 🎨 **Coloring Rules (bonus)**

- Permet de **mettre en évidence** des filtres
- Utile pour repérer rapidement anomalies

👉 Menu : `View → Coloring Rules`

### 🧠 **Résumé rapide**

- **Layer 3 (IP)** → qui communique
- **Layer 4 (TCP/UDP)** → comment ça communique (ports)
- **Layer 7 (HTTP/DNS)** → contenu échangé
- **Display Filter Expression** = aide essentielle
- **Coloring Rules** = analyse visuelle rapide

---

## Advanced Filtering

- Aller **plus loin que les filtres basiques**
- Permet une **analyse fine et ciblée**

### 🧠 **Opérateurs & Fonctions avancés**

📊 Vue d’ensemble

| Filtre     | Type                | Description                                           | Exemple                                   |
| ---------- | ------------------- | ----------------------------------------------------- | ----------------------------------------- |
| `contains` | Comparaison         | Recherche une valeur (case-sensitive)                 | `http.server contains "Apache"`           |
| `matches`  | Comparaison (regex) | Recherche via expression régulière (case-insensitive) | `http.host matches "\.(php\|html)"`       |
| `in`       | Ensemble            | Vérifie appartenance à une liste                      | `tcp.port in {80 443 8080}`               |
| `upper()`  | Fonction            | Convertit en MAJUSCULE                                | `upper(http.server) contains "APACHE"`    |
| `lower()`  | Fonction            | Convertit en minuscule                                | `lower(http.server) contains "apache"`    |
| `string()` | Fonction            | Convertit en string                                   | `string(frame.number) matches "[13579]$"` |

### 🔹 `contains`

- Cherche **une chaîne exacte**
- Sensible à la casse  
    👉 utile pour filtrage simple

![[wireshark packet operations contains.png]]

### 🔹 `matches` (regex)

- Recherche **patterns complexes**
- Insensible à la casse
- ⚠️ Peut produire erreurs si mal utilisé  
    👉 puissant mais à manier avec soin

![[wireshark packet operations matches.png]]

### 🔹 `in`

- Simplifie les conditions multiples  
    👉 remplace plusieurs `OR`

![[wireshark packet operations in.png]]

### 🔹 `upper()` / `lower()`

- Ignore les problèmes de casse  
    👉 rend les filtres plus fiables

![[wireshark packet operations upper.png]]

![[wireshark packet operations lower.png]]

### 🔹 `string()`

- Permet regex sur champs non texte  
    👉 utile pour champs numériques

![[wireshark packet operations string.png]]
### 🔖 **Bookmarks & Filter Buttons**

📌 Fonction

- Sauvegarder filtres complexes
- Réutilisation rapide

💡 Avantages

- Gain de temps
- Standardisation analyse

![[wireshark packet operations bookmarks.png]]

![[wireshark packet operations create & use display filters.png]]

### ⚙️ **Profiles**

📌 Fonction

- Sauvegarder configurations complètes :
    - Filtres
    - Couleurs
    - Préférences

🎯 Utilité

- Adapter Wireshark selon :
    - Type d’analyse
    - Cas d’investigation

👉 Menu :

- `Edit → Configuration Profiles`
- ou barre en bas (profil actif)

