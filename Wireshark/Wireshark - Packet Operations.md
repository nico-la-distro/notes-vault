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

#### 💡 Intérêt :

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

### ⚠️ Limites :

- Nécessite Internet
- Non fonctionnel en lab offline

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

### ⚠️ Note

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

