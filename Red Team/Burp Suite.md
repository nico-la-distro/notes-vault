Burp Suite is an integrated platform for performing security testing of web applications. It includes various tools for scanning, fuzzing, intercepting, and analysing web traffic. It is used by security professionals worldwide to find and exploit vulnerabilities in web applications. it is Java based framework.

## **Principes clés**

- **Définition :** Burp Suite est un framework **Java** conçu pour le **pentest d’applications web** (et aussi mobiles), y compris celles qui utilisent des **API**. C’est un **standard de l’industrie** pour les tests de sécurité “hands-on”.

- **Fonction principale :** Burp **intercepte tout le trafic HTTP/HTTPS** entre le navigateur et le serveur.
    
    - Permet de **voir, modifier et rejouer** des requêtes avant qu’elles arrivent au serveur.
    - Permet aussi de **manipuler des réponses** avant qu’elles soient reçues par le navigateur.
    - Les requêtes interceptées peuvent être **redirigées vers différents modules** de Burp (à voir ensuite).

## **Editions de Burp**

- **Community Edition (CE)**
    - **Gratuite** (usage non commercial + légal).
    - On se concentre dessus ici (core features).

- **Professional (Pro) — payante**
    - Version “débridée” + fonctionnalités avancées :
        - **Scanner** de vulnérabilités automatisé
        - **Fuzzer/brute-forcer** sans limitation de vitesse
        - **Sauvegarde de projets**, génération de **rapports**
        - **API** pour intégration avec d’autres outils
        - **Extensions** sans restriction
        - **Burp Collaborator** (réception/capture de requêtes “out-of-band”, hébergé chez PortSwigger ou self-host)

- **Enterprise — payante**
    - Orientée **scan continu** (automatisé et périodique), style **Nessus** mais pour apps web.
    - Fonctionne comme un service **sur serveur** qui scanne en continu (plutôt que des attaques manuelles locales).


## **Features of Burp Suite**

Même si **Community** est plus limitée que **Professional**, elle reste très utile grâce à plusieurs modules :

- **Proxy**
    - Fonction centrale de Burp.
    - **Intercepte** et permet de **modifier** les requêtes/réponses HTTP(S) pendant l’utilisation d’une appli web.

- **Repeater**
    - Permet de **reprendre une requête**, la modifier, puis la **renvoyer en boucle**.
    - Très pratique pour tests “essai/erreur” (ex : **SQLi**) et validation de comportements d’un endpoint.

- **Intruder**
    - Sert à **pulvériser** un endpoint avec des requêtes (bruteforce / fuzzing).
    - En **Community**, il y a des **limitations de vitesse** (rate limit).

- **Decoder**
    - Outil de **transformation de données** : **encoder/décoder** (ex : URL, Base64, etc.).
    - Gain de temps car intégré directement au workflow Burp.

- **Comparer**
    - Compare deux contenus **au niveau mot** ou **byte**.
    - Utile pour repérer rapidement des différences (ex : réponses proches) via envoi direct depuis Burp.

- **Sequencer**
    - Sert à tester la **randomness** de tokens (cookies de session, jetons, etc.).
    - Si l’aléatoire est faible → risque d’attaques graves (prédiction/vol de session, etc.).

## **Extensions (Extender + BApp Store)**

- Burp est en **Java**, donc extensible via des **extensions** :
    - **Java**
    - **Python** (via **Jython**)
    - **Ruby** (via **JRuby**)

- **Extender** : module pour **charger rapidement** des extensions.

- **BApp Store** : “marketplace” pour télécharger des extensions tierces.

- Certaines extensions nécessitent une licence Pro, mais il en existe **beaucoup compatibles Community**.
    - Exemple : **Logger++** améliore le logging intégré de Burp.

## **Burp Dashboard**

![[Burp Dashboard.png]]

1 - **Tasks**
- Permet de définir des tâches en arrière-plan pendant l’utilisation.
- En Community, la tâche par défaut **Live Passive Crawl** suffit : elle **log** automatiquement les pages visitées.
- En Pro, il y a des fonctionnalités en plus comme des scans à la demande.

2 - **Event log**
- Journal des actions effectuées par Burp (ex : démarrage du proxy).
- Donne aussi des infos sur les connexions passant via Burp.

3 - **Issue Activity**
- Section surtout utile en **Burp Suite Professional**.
- Affiche les vulnérabilités détectées par le scanner automatique (tri par sévérité + filtres selon la certitude).

4 - **Advisory**
- Donne des infos détaillées sur les vulnérabilités : références, remédiations, export en rapport.
- En Community, cette section peut ne rien afficher.

## **Hot keys**

|Shortcut|Onglet|
|---|---|
|Ctrl + Shift + D|Dashboard|
|Ctrl + Shift + T|Target|
|Ctrl + Shift + P|Proxy|
|Ctrl + Shift + I|Intruder|
|Ctrl + Shift + R|Repeater|

## **Settings**

**Global settings (User settings)**
- S’appliquent à toute l’installation Burp.
- Utilisés comme configuration “par défaut” à chaque démarrage.

**Project settings**
- Spécifiques au **projet/session** en cours.
- En **Burp Community**, on ne peut pas **sauvegarder les projets** → ces réglages sont **perdus** à la fermeture de Burp.
### **Proxy settings**

**Response Interception**
- Par défaut, seules les **requêtes** sont interceptées (pas les réponses).
- Pour intercepter aussi des réponses : activer **Intercept responses based on the following rules** et définir des **règles**.

**Match and Replace**
- Modifie automatiquement le trafic entrant/sortant via **regex** (regular expression).
- Exemples : remplacer le **User-Agent**, éditer des **cookies**, changer des headers/paramètres.

**Conseil**
- Teste ces options (règles + match/replace) pour gagner en contrôle et en rapidité dans tes analyses.
## **Burp Proxy**

**Intercepting Requests**
- Les requêtes passant par le proxy peuvent être **interceptées** et **bloquées** avant d’atteindre le serveur.
- Elles apparaissent dans l’onglet **Proxy**, où tu peux :
    - **Forward** (laisser continuer)
    - **Drop** (bloquer)
    - **Edit** (modifier)
    - **Send to…** (Repeater, Intruder, etc.)
- Pour laisser passer sans arrêt : bouton **Intercept** (le désactiver).

**Taking Control**
- Intercepter = **contrôle total** sur le trafic web → très utile pour tester et comprendre le comportement d’une appli.

**Capture et logging**
- Burp **enregistre** les requêtes par défaut, **même si l’interception est OFF**.
- Pratique pour analyser après coup et revoir des échanges.

**WebSocket Support**
- Burp capture et log aussi les **communications WebSocket** (utile pour les applis modernes en temps réel).

**Logs / History**
- Les requêtes capturées se retrouvent dans :
    - **HTTP history**
    - **WebSockets history**
- Permet l’analyse “rétro” et l’envoi vers d’autres modules.

---

## Burp Suite Repeater 

- Permet de **modifier** et **renvoyer** des requêtes HTTP(S) autant de fois que nécessaire.
- Requêtes souvent envoyées depuis **Proxy**, ou créées **à la main** (comme avec `curl`, mais en interface graphique).
- Idéal pour le **test manuel** d’endpoints (essais/erreurs, payloads) + plusieurs vues de réponse (dont un **render**).

|Zone|Rôle|
|---|---|
|Request List|Liste des requêtes Repeater (gérer plusieurs requêtes).|
|Request Controls|Envoyer, annuler une requête bloquée, naviguer dans l’historique.|
|Request / Response View|Éditer la requête + voir la réponse correspondante.|
|Layout Options|Choisir l’affichage (côte à côte, vertical, ou en onglets).|
|Inspector|Analyse/modif “assistée” (plus simple que l’éditeur brut).|
|Target|IP/domaine cible ; auto-rempli si envoyé depuis un autre module.|

![[Burp Suite repeater interface.png]]

---

### Basic Usage

Le plus courant : **capturer une requête dans Proxy** puis l’envoyer dans **Repeater** pour la modifier et la renvoyer.

**Envoyer une requête vers Repeater**
- Clic droit sur la requête → **Send to Repeater**
- Raccourci : **Ctrl + R*    

**Dans Repeater**
- La requête apparaît dans **Request view** (Target + Inspector se remplissent).
- Pas de réponse tant que tu n’as pas cliqué **Send** → la **Response view** se remplit.

**Modifier / retester**
- Tu modifies directement la requête (headers, params, body…) → **Send** à nouveau → la réponse se met à jour.
- Exemple : changer `Connection: close` → `Connection: open` peut donner une réponse avec `Connection: keep-alive`.

**Historique**
- Les boutons d’historique (près de **Send**) permettent de naviguer **en arrière / avant** dans tes modifications.

---

### Message Analysis Toolbar

| Vue    | À quoi ça sert                                                                   |
| ------ | -------------------------------------------------------------------------------- |
| Pretty | Vue par défaut : réponse légèrement formatée pour mieux lire.                    |
| Raw    | Réponse brute telle que reçue du serveur (sans formatage).                       |
| Hex    | Vue au niveau **bytes** (utile pour fichiers binaires).                          |
| Render | Affiche la page comme dans un navigateur (moins utilisé, mais pratique parfois). |

**Show non-printable characters (`\n`)**
- Affiche les caractères invisibles (ex: `\r\n` fin de ligne).
- Utile pour comprendre l’interprétation des **headers HTTP** dans certains cas.

---
### Inspector

- Panneau à droite qui donne une vue **structurée** (type tableau) des éléments d’une requête/réponse.
- Sert à analyser plus vite et à **modifier** la requête sans tout éditer en brut.
- Utile aussi pour voir comment tes changements “haut niveau” se reflètent dans la vue **Raw**.

| Section                  | Contenu                                                       | Modifiable ? |
| ------------------------ | ------------------------------------------------------------- | ------------ |
| Request Attributes       | URL/chemin, méthode (GET/POST…), protocole (HTTP/1 ↔ HTTP/2)  | Oui          |
| Request Query Parameters | Paramètres dans l’URL (`?a=1&b=2`), Specific to POST requests | Oui          |
| Request Body Parameters  | Paramètres envoyés en POST (body)                             | Oui          |
| Request Cookies          | Cookies envoyés avec la requête                               | Oui          |
| Request Headers          | Headers de la requête (ajout/suppression/édition)             | Oui          |
| Response Headers         | Headers renvoyés par le serveur (visible après **Send**)      | Non          |

**À retenir**
- Modifier des headers/params/cookies via Inspector est rapide et pratique.
- Tester des ajouts/suppressions aide à comprendre l’impact sur la requête brute et la réponse serveur.

---

### Pratical example

![[Brup Suite pratical example exercice.png]]

![[Burp Suite pratical example solution.png]]

---

### Challenge

![[Burp Suite challenge exercice + flag.png]]

![[Burp Suite challenge solution.png]]

---
### Extra-mile Challenge

## revoir "Extra-mile Challenge" à https://tryhackme.com/room/burpsuiterepeater après https://tryhackme.com/room/sqlinjectionlm

## Burp Suite Intruder

**Intruder = outil de fuzzing intégré à Burp** : il automatise l’envoi de **requêtes répétées** en **modifiant des paramètres** (valeurs injectées) à partir d’une requête capturée (souvent via **Proxy**).

**Usages typiques**
- **Brute-force** (ex : login) : remplacer `username/password` par des valeurs d’une **wordlist**.
- **Fuzzing** : tester **sous-répertoires**, **endpoints**, **virtual hosts** avec une wordlist.
- Comparable à des outils CLI : **ffuf**, **wfuzz**.

**Limite importante**
- En **Burp Community**, Intruder est **rate-limité** ⇒ beaucoup plus **lent** que Burp Pro, donc certains préfèrent des outils externes.
- Malgré ça, **utile à connaître** et à maîtriser.

---
### Interface Intruder

| Onglet            | À quoi ça sert                                                       | Points clés                                                   |
| ----------------- | -------------------------------------------------------------------- | ------------------------------------------------------------- |
| **Positions**     | Définir **où** injecter les payloads + choisir le **type d’attaque** | On marque les champs/parties de requête à fuzz                |
| **Payloads**      | Définir **quoi injecter** (wordlist, etc.) + règles de traitement    | Pré-traitement : prefix/suffix, match/replace, skip via regex |
| **Resource Pool** | Allocation de ressources (surtout Pro)                               | **Peu utile** en Community                                    |
| **Settings**      | Paramétrer le **comportement** et l’analyse des résultats            | Flags sur texte, gestion des redirections 3xx, etc.           |

---

### Position (intruder)

**Objectif :** définir les _emplacements_ dans la requête où Intruder va **injecter les payloads**.

Burp **pré-sélectionne automatiquement** des positions probables :
- **surbrillance verte**
- entourées par le marqueur **`§`** (section sign)

| Bouton      | Rôle                                                                        | Quand l’utiliser                                            |
| ----------- | --------------------------------------------------------------------------- | ----------------------------------------------------------- |
| **Add §**   | Ajouter une position **manuellement** (en sélectionnant du texte puis clic) | Quand Burp n’a pas marqué la bonne zone ou pour être précis |
| **Clear §** | Supprimer **toutes** les positions                                          | Quand tu veux repartir de zéro (positions custom)           |
| **Auto §**  | Re-détecter automatiquement les positions                                   | Après un Clear, pour retrouver la sélection par défaut      |

---
### Payloads (intruder)

L’onglet **Payloads** sert à **créer / assigner / configurer** les payloads injectés dans les positions définies. Il est découpé en **4 sections**.

**4 Sections**

| Section                | Rôle                                                                 | À retenir / exemples                                                                                                                                                                                     |
| ---------------------- | -------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Payload Sets**       | Choisir **quel payload set** configure-t-on + le **type de payload** | **Sniper / Battering Ram** ⇒ **1 seul** payload set. **Pitchfork / Cluster Bomb** ⇒ **1 set par position**. Numérotation des sets = **haut→bas, gauche→droite** (ex : username = set1, password = set2). |
| **Payload settings**   | Options propres au **type de payload** sélectionné                   | Ex “**Simple list**” : ajout manuel, coller des lignes, charger fichier, Remove/Clear. ⚠️ grosses wordlists ⇒ risque de **crash**.                                                                       |
| **Payload Processing** | Règles appliquées **avant envoi**                                    | Transformations / filtres : capitaliser, match/skip via **regex**, etc. Rarement utilisé mais **très utile** selon le besoin.                                                                            |
| **Payload Encoding**   | Gérer comment Burp **encode** les payloads                           | Par défaut **URL-encoding**. Possibilité d’override : modifier les caractères encodés ou désactiver “URL-encode these characters”.                                                                       |

**A retenir** :

- **1 set vs plusieurs sets** dépend **du type d’attaque**.
- **Processing** = transformer/filtrer **avant** envoi.
- **Encoding** = attention aux cas où l’URL-encoding par défaut casse le test (ou inversement).

---

### Attack Types

|Attack type|Logique d’injection|Quand l’utiliser|
|---|---|---|
|**Sniper** (défaut)|**1 payload à la fois** dans **1 position**, en parcourant les positions|Tests ciblés / fuzz d’un paramètre à la fois|
|**Battering ram**|**Même payload** injecté **en même temps** dans **toutes** les positions|Quand les champs doivent rester synchronisés (même valeur partout) / essais “en parallèle”|
|**Pitchfork**|**1 payload set par position**, injectés **en parallèle** (ligne 1 avec ligne 1, etc.)|Quand chaque paramètre a sa **propre liste** mais doit être testé **pairé** (ex user/pass correspondants)|
|**Cluster bomb**|Produit **toutes les combinaisons** entre payload sets (cartésien)|Quand tu veux tester **toutes les combinaisons** entre plusieurs paramètres (plus complet mais explose vite en volume)|

**À retenir :**
- **Sniper** = simple & précis.
- **Pitchfork** = “zip” (pairing 1-1).
- **Cluster bomb** = “combo” (toutes combinaisons) ⇒ attention au nombre de requêtes.

---

#### Sniper

- **Mode par défaut** et le plus utilisé.
- Idéal pour des attaques **1 paramètre à la fois** : brute-force password, fuzz d’endpoints/API, etc.
- Intruder prend **un seul payload set** (wordlist, plage de nombres…) et l’injecte **dans chaque position, l’une après l’autre**.

```http
POST /support/login/ HTTP/1.1
Host: 10.81.156.136
User-Agent: Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:80.0) Gecko/20100101 Firefox/80.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Content-Type: application/x-www-form-urlencoded
Content-Length: 37
Origin: http://10.81.156.136
Connection: close
Referer: http://10.81.156.136/support/login/
Upgrade-Insecure-Requests: 1

username=§pentester§&password=§Expl01ted§
```


**Exemple (2 positions : username + password)**

Template :  
`username=§pentester§&password=§Expl01ted§`

Wordlist : `burp`, `suite`, `intruder`

|#|Corps de requête|
|---|---|
|1|`username=burp&password=Expl01ted`|
|2|`username=suite&password=Expl01ted`|
|3|`username=intruder&password=Expl01ted`|
|4|`username=pentester&password=burp`|
|5|`username=pentester&password=suite`|
|6|`username=pentester&password=intruder`|

**Logique :**
1. Remplit **username** avec tous les payloads → 2) puis **password** avec tous les payloads.

**Formule à retenir :**
**Nombre de requêtes = nbPayloads × nbPositions**

---
#### Battering Ram

Différence clé vs **Sniper** :

- **Sniper** = payloads injectés **position par position**
    
- **Battering ram** = **le même payload** est injecté **dans toutes les positions en même temps** (même requête)

```http
POST /support/login/ HTTP/1.1
    Host: 10.81.156.136
    User-Agent: Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:80.0) Gecko/20100101 Firefox/80.0
    Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8
    Accept-Language: en-US,en;q=0.5
    Accept-Encoding: gzip, deflate
    Content-Type: application/x-www-form-urlencoded
    Content-Length: 37
    Origin: http://10.81.156.136
    Connection: close
    Referer: http://10.81.156.136/support/login/
    Upgrade-Insecure-Requests: 1
    
    username=§pentester§&password=§Expl01ted§
```

**Exemple (2 positions : username + password)**

Template :  
`username=§pentester§&password=§Expl01ted§`

Wordlist : `burp`, `suite`, `intruder`

|#|Corps de requête|
|---|---|
|1|`username=burp&password=burp`|
|2|`username=suite&password=suite`|
|3|`username=intruder&password=intruder`|

**Logique :** 1 payload = 1 requête, injecté partout.

**Formule à retenir**
**Nombre de requêtes = nbPayloads** (peu importe le nombre de positions)

---
#### Pitchfork

- Comparable à **plusieurs Sniper en parallèle**.
    
- **1 payload set par position** (jusqu’à **20**).
    
- À chaque requête, Intruder prend **l’item i** de **chaque** liste et l’injecte dans **sa** position (mode “zip”).

**Exemple (2 listes : usernames + passwords)**

|#|Corps de requête|
|---|---|
|1|`username=joel&password=J03l`|
|2|`username=harriet&password=Emma1815`|
|3|`username=alex&password=Sk1ll`|

**Logique :** (user[1] + pass[1]), puis (user[2] + pass[2]), etc.

---

**Règle importante**

- L’attaque **s’arrête dès qu’une liste est épuisée**.  
    ➡️ Donc idéalement, les payload sets doivent avoir **la même longueur**, sinon les valeurs “en trop” de la liste la plus longue **ne seront jamais testées**.

---
#### Cluster bomb

- **1 payload set par position** (jusqu’à **20**), comme Pitchfork.
    
- Différence clé : **teste toutes les combinaisons possibles** entre les listes (produit cartésien), pas un “zip”.

**Exemple (usernames × passwords)**

Usernames : `joel`, `harriet`, `alex`  
Passwords : `J03l`, `Emma1815`, `Sk1ll`

|#|Corps de requête|
|---|---|
|1|`username=joel&password=J03l`|
|2|`username=harriet&password=J03l`|
|3|`username=alex&password=J03l`|
|4|`username=joel&password=Emma1815`|
|5|`username=harriet&password=Emma1815`|
|6|`username=alex&password=Emma1815`|
|7|`username=joel&password=Sk1ll`|
|8|`username=harriet&password=Sk1ll`|
|9|`username=alex&password=Sk1ll`|

---

**Formule à retenir**

**Nombre de requêtes = (taille set1) × (taille set2) × …**  
➡️ Ça **explose vite** si les listes sont grandes.

---

**Points d’attention**

- Génère **beaucoup de trafic**.
    
- En **Burp Community** (rate-limit Intruder) ⇒ peut devenir **très long** même avec des listes “moyennes”.

---

#### Pratical exemple

**Fichiers / wordlists**

Commande de récup :

- `wget http://10.81.156.136:9999/Credentials/BastionHostingCreds.zip`

Contenu zip :

- `emails.txt`
- `usernames.txt` ✅ utilisé
- `passwords.txt` ✅ utilisé
- `combined.txt` (email:password)

---

**Procédure (Burp Proxy → Intruder Pitchfork)**

|Étape|Action|Détail important|
|---|---|---|
|1|Télécharger + extraire le zip|Récupérer `usernames.txt` + `passwords.txt`|
|2|Ouvrir la page login + activer Proxy|Faire un login test (n’importe quels creds) pour capturer la requête|
|3|Send to Intruder|Clic droit → **Send to Intruder** ou **Ctrl+I**|
|4|Positions|Garder **uniquement** `username` et `password`|
|5|Attack type|Choisir **Pitchfork**|
|6|Payloads|Set 1 = usernames|
|7|Payloads|Set 2 = passwords|
|8|Start Attack|Lancer (OK si warning rate-limit)|

---

**Trouver le bon login (résultats Intruder)**

- Ici, **tous** les essais renvoient **302** ⇒ le status code ne distingue pas succès/échec.
    
- Il faut utiliser **Length (taille réponse)** :
    
    1. Trier la colonne **Length** (clic sur l’en-tête)
    2. Repérer la requête avec **la longueur la plus courte** (indice de succès)
    3. Tester ces identifiants directement sur le portail pour confirmer

---

**À retenir**

- **Pitchfork** = idéal pour credential stuffing (listes alignées user/pass).
    
- Quand **codes identiques**, regarde **Length** (ou parfois cookies/Location/headers selon cas).

![[Burp Suite Intruder (set both payloads).png]]

![[Burp Suite intruder (credentials).png]]

---
#### Pratical challenge

**Task**

![[Burp Suite intruder pratical challenge (task).png]]

**Sniper Payload setup**

![[Burp Suite intruder pratical challenge (sniper payload setup).png]]

**Tickets find**

![[Burp Suite intruder pratical challenge (tickets fuzzing).png]]

---
#### Extra Mile Challenge

**Task**
see task 12 : https://tryhackme.com/room/burpsuiteintruder

**Set the payload**

![[Burp Suite intruder extra mile (setting payload).png]]

**Setting the Macro (grab CSRF) to bypass loginToken**

![[Burp Suite intruder extra mile (set macro).png]]

**finding credential after etablished a Macro**
![[Burp Suite intruder extra mile (credentials).png]]

---


## Suite : https://tryhackme.com/room/burpsuiteom

