## Task 1 — Brief

- Room basée sur un site type “Acme IT Support” avec machine déployable.
    
- Objectif : contourner des mécanismes d’auth **sans** creds légitimes, via mauvaises implémentations.
    

---

## Task 2 — Username Enumeration

### Idée clé

Si la page d’inscription/login renvoie un message **différent** quand le username existe (“username already exists”), tu peux **déduire** les usernames valides → tu réduis énormément l’espace d’attaque.

### Automatisation avec `ffuf` (pattern)

- Envoi de requêtes **POST** vers le formulaire signup.
    
- Détection via **match response** (`-mr`) sur une chaîne distinctive.
    

#### Commande type (modèle)

```bash
ffuf -w /usr/share/wordlists/SecLists/Usernames/Names/names.txt \  
-X POST \  
-d "username=FUZZ&email=x&password=x&cpassword=x" \  
-H "Content-Type: application/x-www-form-urlencoded" \  
-u http://MACHINE_IP/customers/signup \  
-mr "username already exists"
```

### Tableau mémo `ffuf`

|Option|Rôle|Mémo|
|---|---|---|
|`-w`|wordlist|source des valeurs injectées|
|`FUZZ`|placeholder|position d’injection|
|`-X`|méthode HTTP|POST ici (pas GET)|
|`-d`|body|champs du formulaire|
|`-H`|headers|souvent `Content-Type`|
|`-mr`|match regex/string|“preuve” qu’un username est valide|

✅ Sortie attendue : tu te fais un fichier `valid_usernames.txt` pour la suite.

---

## Task 3 — Brute Force

### Idée clé

Tu testes une liste de passwords **sur des usernames validés** (Task 2).  
Point important : tu peux détecter le succès par un **status code différent** (ex: redirection 302) plutôt qu’un texte dans la page.

### `ffuf` avec 2 wordlists (username + password)

ffuf -w valid_usernames.txt:W1,/usr/share/wordlists/SecLists/Passwords/Common-Credentials/10-million-password-list-top-100.txt:W2 \  
-X POST \  
-d "username=W1&password=W2" \  
-H "Content-Type: application/x-www-form-urlencoded" \  
-u http://MACHINE_IP/customers/login \  
-fc 200

- `W1` = username, `W2` = password (placeholders custom car plusieurs wordlists).
    
- `-fc 200` = “filtre” les réponses 200 ; un succès peut être 302 (redirect).
    

---

## Task 4 — Logic Flaw

### 1) Exemple “classique” : check de route sensible à la casse

Un check du type “si l’URL commence par `/admin` alors vérifier admin” peut être contourné si la comparaison est **trop stricte** (casse) et que `/adMin` n’est pas détecté comme zone admin.

### 2) Cas pratique (Reset Password) : confusion GET vs POST (`$_REQUEST`)

Le flow reset :

- étape 1 : tu donnes un **email** (GET querystring),
    
- étape 2 : tu donnes un **username** (POST),
    
- et l’app utilise une variable qui fusionne GET+POST (ex: `$_REQUEST`)… **en favorisant POST** si clé dupliquée.  
    ➡️ Résultat : tu peux injecter un champ `email` **dans le POST** pour influencer où part le mail de reset.
    

#### Modèle de requêtes `curl`

|Requête|But|Exemple|
|---|---|---|
|1|reset “normal”|`curl 'http://MACHINE_IP/customers/reset?email=victim%40domain' -H 'Content-Type: application/x-www-form-urlencoded' -d 'username=victim'`|
|2|exploitation|même requête + `&email=attacker@...` **dans le POST**|

---

## Task 5 — Cookie Tampering

### A) Cookies en clair (cas le plus simple)

Si le serveur pose des cookies “logiques” du style :

- `logged_in=true/false`
    
- `admin=true/false`  
    …et que l’app **fait confiance** au client, alors changer `admin=true` peut suffire à obtenir des privilèges.
    

#### Mémo requêtes `curl`

|Niveau|Cookie envoyé|Effet attendu|
|---|---|---|
|0|aucun|“Not logged in”|
|user|`logged_in=true; admin=false`|user logged|
|admin|`logged_in=true; admin=true`|admin logged|

### B) Hash vs Encoding (à ne pas confondre)

|Concept|Réversible ?|À retenir|Exemples / outils|
|---|---|---|---|
|**Hash**|❌ non|même entrée → même sortie, mais pas de “décodage” (sauf lookup/wordlist)|crackstation (lookup)|
|**Encoding (Base64)**|✅ oui|c’est une représentation transportable, pas une “protection”|base64 decode/encode, CyberChef|

### C) Cas fréquent : cookie Base64 contenant du JSON

Ex: cookie `session=eyJpZCI6MSwiYWRtaW4iOmZhbHNlfQ==` → une fois décodé ça ressemble à `{"id":1,"admin": false}` ; tu peux modifier `admin` puis ré-encoder.

