- **CyberChef** = appli web (dans le navigateur) pour manipuler des données.
- Comparable à un **couteau suisse** : plein d’outils “cyber” pour transformer/décoder/chiffrer.
- Fonctionne avec des **recipes** : une **suite d’opérations** exécutées **dans l’ordre**

CyberChef sert à **transformer, analyser et décoder/chiffrer des données** directement dans le navigateur en **enchaînant des opérations** (une “recette”) jusqu’à obtenir le résultat voulu.
## **Accessing the tool**

### Online Access

All you need is a web browser and an internet connection. Then, you can click this [link](https://gchq.github.io/CyberChef) to open CyberChef directly within your web browser.
### Offline or Local Copy

You can run this offline or locally on your machine by downloading the latest release file from this [link](https://github.com/gchq/CyberChef/releases). This will work on both Windows and Linux platforms. As best practice, you should download the most stable version.


## **Vue d'ensemble**

| Zone       | Rôle                                             | Actions typiques                                   |
| ---------- | ------------------------------------------------ | -------------------------------------------------- |
| Operations | Liste d’outils disponibles (classés + recherche) | Chercher “Base64”, “XOR”, “ROT13”, etc.            |
| Recipe     | Pipeline d’opérations exécutées **dans l’ordre** | Drag & drop ops → régler options → BAKE!/Auto Bake |
| Input      | Entrée des données                               | Coller/écrire/glisser un fichier, multi-onglets    |
| Output     | Résultat des transformations                     | Copier / sauvegarder / remplacer l’input           |

## **Operations**

|Opération|À quoi ça sert|Exemple (idée)|
|---|---|---|
|From Morse Code|Morse → texte (A–Z, 0–9)|`- ....` → lettres|
|URL Encode|Caractères spéciaux → `%xx`|URL → version encodée|
|To Base64|Texte brut → Base64 ASCII|“This is fun!” → `VGhpcy...`|
|To Hex|Texte → hex (avec séparateur)|“...” → `54 68 69...`|
|To Decimal|Texte → codes décimaux (ordinal)|“...” → `84 104 105...`|
|ROT13|Substitution César (défaut 13)|texte → version ROT13|

## **Categories to know**

### **Extractor**

|Opération|Sert à|Point d’attention|
|---|---|---|
|Extract IP addresses|Extraire toutes les IPv4/IPv6|repère vite des IOC réseau|
|Extract email addresses|Extraire emails (`anything@domain.com`)|utile sur logs, dumps, textes|
|Extract URLs|Extraire des URL|**le protocole est requis** (http/https/ftp) sinon trop de faux positifs|

---

### **Date / Time**

|Opération|Sert à|Exemple|
|---|---|---|
|To UNIX Timestamp|Date/heure UTC → timestamp|“Fri Sep 6 … 2024” → `1725654622`|
|From UNIX Timestamp|Timestamp → date lisible|`1725654622` → datetime|

**Rappel** : timestamp UNIX = **secondes** depuis **01/01/1970 UTC** (epoch), souvent présenté comme valeur 32-bit dans le texte.

---

### **Date format (décodages/encodages)**

|Opération|Sert à|Exemple|
|---|---|---|
|From Base64|Base64 → texte brut|`V2VsY29tZ...` → “Welcome…”|
|URL Decode|`%xx` → caractères normaux|`https%3A%2F...` → `https://...`|
|From Base85|Base85 → texte brut (plus compact que Base64)|`BOu!rD]...` → “hello world”|
|From Base58|Base58 → texte brut (évite `l I 0 O`)|`AXLU7qR` → `Thm58`|
|To Base62|texte → Base62 (alphabet restreint, court)|`Thm62` → `6NiRkOY`|

**Idée clé** : les “BaseXX” sont des **encodages** (binaire → texte ASCII).

---

### **Exemple Base64**

**THM -> Base64**

|Étape|Action|Résultat|
|---|---|---|
|1|ASCII → binaire puis concaténer|`T=01010100` `H=01001000` `M=01001101` → `010101000100100001001101`|
|2|Découper en 6 bits + convertir en décimal|`010101 000100 100001 001101` → `21, 4, 33, 13`|
|3|Table Base64 (index → char)|`21=V`, `4=E`, `33=h`, `13=N` → **`VEhN`**|

---

### **URL decode**

Convertit `%xx` (UTF-8) → caractère.

| Caractère | Encodage |
| --------- | -------- |
| `:`       | `%3A`    |
| `/`       | `%2F`    |
| `.`       | `%2E`    |
| `=`       | `%3D`    |
| `#`       | `%23`    |

---

### **Astuce CTF**

Quand tu vois un blob “bizarre” :
- commence par **From Base64 / From Base58 / From Base85** + **URL Decode**,
- puis si ça ressemble à des IOC/logs → **Extract IP/URL/email**,
- si tu vois des gros nombres → **From UNIX Timestamp**.

