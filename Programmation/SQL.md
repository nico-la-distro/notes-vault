## **Definition**

SQL (Structured Query Language) is a declarative programming language used to store, query, and manipulate data in relational databases, which organize information in tables made of rows and columns. SQL allows users to manage data and perform maintenance and optimization tasks on databases.

- **Rapide** : traitement et restitution de grands volumes de données très efficaces.
- **Facile à apprendre** : syntaxe proche de l’anglais, lisible et accessible.
- **Fiable** : structure stricte garantissant la cohérence et l’intégrité des données.
- **Flexible** : puissant pour l’analyse et les requêtes complexes.

## **Databases types**

- **Bases relationnelles (SQL)**  
    Données **structurées** stockées dans des **tables** (lignes et colonnes).  
    Chaque enregistrement suit un schéma précis et des **relations** peuvent exister entre plusieurs tables (ex. utilisateurs ↔ commandes).  
    Adaptées aux données cohérentes et critiques (ex. transactions e-commerce).

- **Bases non relationnelles (NoSQL)**  
    Données **non tabulaires** et flexibles (documents, clés-valeurs, etc.).  
    Adaptées aux données hétérogènes ou évolutives (ex. contenus générés par les utilisateurs sur les réseaux sociaux).

![[Databases types.png]]

## **Tables, Rows and Columns (SQL)**

- Une **table** stocke les données (ex. _Books_).
- Les **colonnes** définissent les champs (id, nom, date, etc.) avec des **types de données** (string, integer, float, date…).
- Chaque **ligne** représente un enregistrement unique.
- Toute donnée ne respectant pas le type défini est rejetée.

![[SQL (tables, rows, columns).png]]

## **Primary and foreign keys**

- **Clé primaire (Primary Key)** : identifiant unique d’un enregistrement dans une table.
    - Une seule par table.

- **Clé étrangère (Foreign Key)** : champ faisant référence à la clé primaire d’une autre table.
    - Permet de **lier les tables** et de créer des relations (ex. _author_id_ dans _Books_).

![[SQL (Primary & Foreign Key).png]]

## **DBMS (Database Management System)**

 Databases are usually controlled using a Database Management System (DBMS). Serving as an interface between the end user and the database, a DBMS is a software program that allows users to retrieve, update and manage the data being stored. Some examples of DBMSs include MySQL, MongoDB, Oracle Database and Maria DB.

## **Statements**

**Database Statements**

|Commande|Rôle|Exemple|
|---|---|---|
|`CREATE DATABASE`|Créer une base|`CREATE DATABASE thm_bookmarket_db;`|
|`SHOW DATABASES`|Lister les bases|`SHOW DATABASES;`|
|`USE`|Sélectionner la base active|`USE thm_bookmarket_db;`|
|`DROP DATABASE`|Supprimer une base|`DROP DATABASE database_name;`|

---

**Table Statements**

|Commande|Rôle|Exemple|
|---|---|---|
|`CREATE TABLE`|Créer une table|`CREATE TABLE book_inventory (...);`|
|`SHOW TABLES`|Lister les tables|`SHOW TABLES;`|
|`DESCRIBE / DESC`|Voir la structure|`DESCRIBE book_inventory;`|
|`ALTER TABLE`|Modifier une table|`ALTER TABLE book_inventory ADD page_count INT;`|
|`DROP TABLE`|Supprimer une table|`DROP TABLE book_inventory;`|

---

**Contraintes & Types**

|Élément|Rôle|Exemple|
|---|---|---|
|`PRIMARY KEY`|Identifiant unique|`book_id INT PRIMARY KEY`|
|`AUTO_INCREMENT`|ID automatique|`book_id INT AUTO_INCREMENT`|
|`NOT NULL`|Champ obligatoire|`book_name VARCHAR(255) NOT NULL`|
|`INT`|Entier|`page_count INT`|
|`VARCHAR(n)`|Texte|`book_name VARCHAR(255)`|
|`DATE`|Date|`publication_date DATE`|

## **CRUD operations**

**CRUD** regroupe les **opérations fondamentales** de manipulation des données dans une base :  **Create, Read, Update, Delete.**

|Opération|Commande SQL|Rôle|Exemple|
|---|---|---|---|
|**Create**|`INSERT INTO`|Ajouter un enregistrement|`INSERT INTO books (...) VALUES (...);`|
|**Read**|`SELECT`|Lire / récupérer des données|`SELECT * FROM books;`|
|**Update**|`UPDATE`|Modifier un enregistrement|`UPDATE books SET ... WHERE id = 1;`|
|**Delete**|`DELETE`|Supprimer un enregistrement|`DELETE FROM books WHERE id = 1;`|

- `INSERT` ajoute une **nouvelle ligne** dans une table.
- `SELECT` permet de récupérer **toutes** (`*`) ou **certaines colonnes**.
- `UPDATE` modifie des données existantes et doit presque toujours être accompagné d’un **WHERE**.
- `DELETE` supprime des données ; sans **WHERE**, toute la table peut être vidée.

## **Clauses**

Une **clause** précise **comment les données sont sélectionnées, filtrées ou organisées** dans une requête SQL.

|Clause|Rôle|Point clé|Exemple|
|---|---|---|---|
|`DISTINCT`|Éliminer les doublons|Retourne uniquement des valeurs uniques|`SELECT DISTINCT name FROM books;`|
|`GROUP BY`|Regrouper des lignes|Souvent utilisé avec des fonctions d’agrégation|`GROUP BY name`|
|`ORDER BY`|Trier les résultats|`ASC` (croissant) / `DESC` (décroissant)|`ORDER BY published_date DESC`|
|`HAVING`|Filtrer après agrégation|S’applique aux groupes, pas aux lignes|`HAVING COUNT(*) > 1`|
**Points clés à retenir**

- `DISTINCT` agit sur les **résultats retournés**, pas sur la table.
- `GROUP BY` est utilisé pour **agréger des données** (ex. `COUNT`, `SUM`).
- `ORDER BY` contrôle uniquement **l’affichage des résultats**.
- `HAVING` filtre **après** un `GROUP BY`, contrairement à `WHERE` qui filtre **avant**.

**Règle mentale simple**

- `WHERE` → filtre les **lignes**
- `GROUP BY` → regroupe les **lignes**
- `HAVING` → filtre les **groupes**
- `ORDER BY` → trie le **résultat final**

## **Operators**

Les **opérateurs SQL** permettent de filtrer et comparer des données dans les requêtes, principalement avec `WHERE`.

**logical operators**

|Opérateur|Rôle|Exemple|Point clé|
|---|---|---|---|
|`LIKE`|Recherche par motif|`WHERE description LIKE "%guide%"`|`%` = joker (wildcard)|
|`AND`|Toutes les conditions doivent être vraies|`WHERE category="Offensive Security" AND name="Bug Bounty Bootcamp"`|Logique cumulative|
|`OR`|Au moins une condition vraie|`WHERE name LIKE "%Android%" OR name LIKE "%iOS%"`|Logique alternative|
|`NOT`|Inverse une condition|`WHERE NOT description LIKE "%guide%"`|Exclusion|
|`BETWEEN`|Vérifie un intervalle|`WHERE id BETWEEN 2 AND 4`|Inclusif|

**comparison operators

|Opérateur|Rôle|Exemple|
|---|---|---|
|`=`|Égal à|`WHERE name = "Designing Secure Software"`|
|`!=`|Différent de|`WHERE category != "Offensive Security"`|
|`<`|Inférieur à|`WHERE published_date < "2020-01-01"`|
|`>`|Supérieur à|`WHERE published_date > "2020-01-01"`|
|`<=`|Inférieur ou égal|`WHERE published_date <= "2021-11-15"`|
|`>=`|Supérieur ou égal|`WHERE published_date >= "2021-11-02"`|

**à retenir**

- Les opérateurs sont principalement utilisés avec `WHERE`.
- `LIKE` est essentiel pour les recherches partielles.
- `BETWEEN` inclut les bornes.
- Les comparateurs fonctionnent aussi avec dates et nombres.
- `AND` restreint, `OR` élargit, `NOT` exclut.

## **Functions**

Les **fonctions SQL** permettent de manipuler, transformer ou agréger des données dans une requête.

**string functions**

|Fonction|Rôle|Exemple|Point clé|
|---|---|---|---|
|`CONCAT()`|Fusionner plusieurs chaînes|`CONCAT(name, " - ", category)`|Combine des colonnes|
|`GROUP_CONCAT()`|Fusionner plusieurs lignes en une seule|`GROUP_CONCAT(name)`|Utilisé avec `GROUP BY`|
|`SUBSTRING()`|Extraire une partie d’une chaîne|`SUBSTRING(published_date, 1, 4)`|Position + longueur|
|`LENGTH()`|Compter le nombre de caractères|`LENGTH(name)`|Inclut espaces|

**aggregate functions**

|Fonction|Rôle|Exemple|Point clé|
|---|---|---|---|
|`COUNT()`|Compter les lignes|`COUNT(*)`|Compte total|
|`SUM()`|Additionner une colonne|`SUM(price)`|Ignore `NULL`|
|`MAX()`|Valeur maximale|`MAX(published_date)`|Ex : date la plus récente|
|`MIN()`|Valeur minimale|`MIN(published_date)`|Ex : date la plus ancienne|
**à retenir**

- Les fonctions **transforment les résultats** d’une requête.
- Les fonctions d’agrégation travaillent sur **plusieurs lignes**.
- `GROUP BY` est souvent combiné avec `COUNT`, `SUM`, etc.
- Les fonctions peuvent être renommées avec `AS`.

