**IDOR** =  Insecure Direct Object Reference

|Élément|À retenir|
|---|---|
|Définition|IDOR = référence directe à un objet accessible via un paramètre manipulable|
|Type|Vulnérabilité de **contrôle d’accès**|
|Cause|Entrée utilisateur **non vérifiée côté serveur** (ownership / autorisation)|
|Impact typique|Accès à des données/fichiers d’autres utilisateurs|
## IDOR exemple

| Indice                                     | Ce que tu fais                                         | Ce que ça prouve                                                 |
| ------------------------------------------ | ------------------------------------------------------ | ---------------------------------------------------------------- |
| Paramètre `user_id` dans l’URL             | Changer la valeur (ex: 1305 → 1000)                    | Si données d’un autre user = contrôle d’accès absent/insuffisant |
| Accès direct à un objet via un identifiant | Tester d’autres identifiants (séquentiels, devinables) | Risque d’énumération + fuite de données                          |
| Pas d’erreur / pas de redirection          | Observer la réponse du serveur                         | Autorisation non vérifiée côté serveur                           |

---
## Finging IDORs in Encoded IDs

| Élément                | À retenir                                       | Indices visuels                        | Action                          |
| ---------------------- | ----------------------------------------------- | -------------------------------------- | ------------------------------- |
| Encodage ≠ chiffrement | Base64 n’empêche pas la modification            | chaîne “random” + souvent `=` à la fin | Décoder → modifier → ré-encoder |
| Où ça se cache         | URL, body POST, cookies                         | paramètres suspects (ex: `id=...`)     | Tester le même workflow         |
| Ce que tu cherches     | Changement de contenu / accès à d’autres objets | réponse différente après modification  | Si oui → contrôle d’accès cassé |

![[IDOR (base64 encoded IDs).png]]

---

## Finging IDORs in Hashed IDs

| Type d’ID        |        Réversible ? | Indice                           | Méthode de test                                           |
| ---------------- | ------------------: | -------------------------------- | --------------------------------------------------------- |
| Encodé (base64…) |               ✅ Oui | souvent `=` / alphabet base64    | décoder → modifier → ré-encoder                           |
| Hashé (md5/sha…) | ❌ Non (en pratique) | format hex long (md5 = 32 chars) | “crack”/lookup DB, deviner le pattern (ex: hash d’entier) |

**crackstation** : https://crackstation.net/

---
## Finging IDORs in Unpredictable IDs

|Étape|Compte A|Compte B|Ce que tu observes|
|---|---|---|---|
|1. Collecte|note l’ID/objet de A|note l’ID/objet de B|où l’ID apparaît (URL, POST, cookie, JSON)|
|2. Swap|A envoie requête avec ID de B|B envoie requête avec ID de A|si données cross-account → fail contrôle d’accès|
|3. Variante|tester sans être connecté|—|si accès anonyme → contrôle d’accès encore plus grave|

---
## Where are IDORs located

|Où chercher|Indices|Tests typiques|
|---|---|---|
|Requêtes AJAX (Network)|endpoints JSON, XHR/fetch|modifier `id`, `user_id`, `account`, `doc`, etc.|
|Fichiers JS|URLs d’API, routes internes|explorer endpoints + paramètres|
|Paramètres “fantômes”|comportement change selon un param|essayer `user_id`, `id`, `uid`, `account_id`, `role`, etc.|

**Parameter mining** : trouver des **paramètres cachés** (non visibles dans l’UI) qui modifient le comportement d’un endpoint et peuvent ouvrir une IDOR.

