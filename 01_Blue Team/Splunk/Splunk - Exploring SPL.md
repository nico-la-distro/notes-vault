
**_SPL : **Search Processing Language**_
## Search & Reporting

Interface par défaut de Splunk pour rechercher et analyser des données.

|Élément|Rôle|
|---|---|
|Search Head|Saisie des requêtes SPL|
|Time Picker|Sélection de la plage temporelle|
|Search History|Historique des requêtes|
|Data Summary|Vue des hosts, sources, sourcetypes disponibles|

![[splunk exploring spl search & reporting.png]]

### Your First Search

Index = conteneur/base de données Splunk.

spl

```spl
index=windowslogs
```

`index=windowslogs` et `index = windowslogs` sont tous deux valides.

### Fields Sidebar

Panneau gauche de l'interface de recherche.

|Section|Description|
|---|---|
|Selected Fields|Champs extraits par défaut (modifiables via toggle)|
|Interesting Fields|Champs détectés automatiquement comme pertinents|
|`#`|Champ numérique|
|`α`|Champ alphanumérique (texte)|
|Count|Nombre d'événements contenant ce champ|
|More available fields|Champs supplémentaires accessibles si présents|

![[splunk exploring spl fields sidebar.png]]

---

## Search Operators

Splunk's **Search Processing Language** (SPL) is behind every search in Splunk. It combines commands, functions, and operators that allow you to filter, transform, and analyze data from your ingested logs. In essence, SPL lets you search through massive amounts of data, apply filters to narrow down results, and format the output. Let's see how to use SPL.
https://help.splunk.com/en/splunk-enterprise/search/search-manual/10.0/expressions-and-predicates/predicate-expressions
### Free Text Search

spl

```spl
index=windowslogs alice
```

Recherche tous les événements contenant le mot-clé (insensible à la casse). Utile pour une chasse rapide sans connaître les noms de champs.

### Relational Operators

|Opérateur|Exemple|
|---|---|
|`=`|`UserName=Mark`|
|`!=`|`UserName!=Mark`|
|`<`|`Age<10`|
|`<=`|`Age<=10`|
|`>`|`Outbound_Traffic>50`|
|`>=`|`Outbound_Traffic>=50`|

### Logical Operators

|Opérateur|Exemple|Note|
|---|---|---|
|`NOT`|`NOT UserName=*`|Retourne les événements où le champ n'existe pas|
|`AND`|`UserName=David AND IPAddress=10.10.10.10`|Implicite : `A B` = `A AND B`|
|`OR`|`UserName=David OR UserName=John`||
|`IN`|`UserName IN(David, John)`|Alternative à `OR` pour les longues listes|

### Wildcards and CIDR Search

|Syntaxe|Exemple|Résultat|
|---|---|---|
|`*` (partiel)|`status=*fail*`|failed, failure, appfail…|
|`*` (préfixe)|`DestinationIp=172.*`|Toute IP commençant par 172.|
|CIDR|`DestinationIp=172.18.0.0/16`|Toute IP dans le sous-réseau|

### Order of Evaluation

**Quotes** — `""` pour phrase exacte ou échapper des opérateurs :

spl

```spl
index=windowslogs failed login        -- "failed" ET "login", ordre quelconque
index=windowslogs "failed login"      -- phrase exacte
index=windowslogs "TO BE OR NOT TO BE" -- OR/NOT traités comme du texte
```

**Parenthèses** — `OR` a la priorité sur `AND` → toujours expliciter :

spl

```spl
-- INCORRECT (Splunk évalue OR en premier)
index=windowslogs alice AND bob OR charlie

-- CORRECT
index=windowslogs (alice AND bob) OR charlie
```

---

## Filtering Results

Les commandes SPL sont chaînées via `|` — chaque pipe passe la sortie vers la commande suivante.

### Useful Filtering Commands

**Fields** — inclure/exclure des champs :

spl

```spl
index=windowslogs | fields host User SourceIp       -- inclusion
index=windowslogs | fields - User                   -- exclusion avec -
```

![[splunk exploring spl fields.png]]

**Dedup** — supprime les doublons sur un champ :

spl

```spl
index=windowslogs
| fields EventID User Image Hostname SourceIp
| dedup SourceIp
```

Utile pour les subsearches et pour dédupliquer des sources bruyantes (ex: M365).

![[splunk exploring spl dedup.png]]

**Rename** — renomme un champ, ou aplatit des sous-champs JSON/XML :

spl

```spl
index=windowslogs
| fields EventID User Image Hostname SourceIp
| rename User as Employee

index=jsondata
| rename request.* as *    -- request.path -> path, request.ip -> ip
```

![[splunk exploring spl rename.png]]

**Regex** — filtre par expression régulière PCRE (Perl Compatible Regular Expressions) sur la valeur d'un champ :

spl

```spl
index=windowslogs | regex Image = "\.exe$"
```

The query above applies a regular expression to the `Image` field, returning only events where the field value ends with `.exe`. The `$` symbol specifies that the match must occur at the end of the string. That was the simplest example, but regex is irreplaceable for complex searches, especially on custom or poorly-parsed data sources.

Indispensable pour des formats complexes ou des sources mal parsées.

![[splunk exploring spl regex.png]]

---

## Structuring Results

### Table Command

https://help.splunk.com/en/splunk-enterprise/spl-search-reference/9.0/search-commands/table
Affiche uniquement les champs sélectionnés dans un format lisible, trié par timestamp :

spl

```spl
index=windowslogs | table _time EventID Hostname SourceName
```

![[splunk exploring spl table command.png]]

### Useful Structuring Commands

Other commands can be used alone or combined with `table` to hone in on the data you're really interested in. Let's look at some examples in the table below.

| Commande  | Exemple                        | Effet                                     |
| --------- | ------------------------------ | ----------------------------------------- |
| `head`    | index=windowslogs \| head 20   | 20 premiers événements (les plus récents) |
| `tail`    | index=windowslogs \| tail 20   | 20 derniers événements (les plus anciens) |
| `sort`    | index=windowslogs \| sort User | Tri alphabétique par champ                |
| `reverse` | index=windowslogs \| reverse   | Inverse l'ordre des événements            |

### Timelining With Table

spl

```spl
index=windowslogs Hostname=Salena.Adam
| table _time Hostname EventID Category
| reverse
```

Reconstruit une chronologie d'actions sur un host.

![[splunk exploring spl timelining with table.png]]

### Subsearches

Corrèle deux sources de données via un champ commun. Exemple : enrichir des events Sysmon (EventID=1) avec le contexte de logon (EventID=4624) via `LogonId` :

![[splunk exploring spl subsearches.png]]

spl

```spl
index=windowslogs EventID=1
| join LogonId
    [ search index=windowslogs EventID=4624
    | rename TargetLogonId as LogonId
    | fields LogonId LogonType IpAddress]
| table _time Image User LogonType IpAddress
```

**Fonctionnement :**

1. La subsearch `[ ... ]` s'exécute en premier → renomme `TargetLogonId` en `LogonId` → stocke les tuples `(LogonId, LogonType, IpAddress)`
2. La recherche principale trouve les events `EventID=1` → joint sur `LogonId` → ajoute `LogonType` et `IpAddress` si match

> Puissant mais lent sur de gros datasets. Préférer `stats` + `eval` quand possible.

---

## Transforming Commands

[Transforming commands(opens in new tab)](https://help.splunk.com/en/splunk-cloud-platform/search/search-manual/10.0.2503/create-statistical-tables-and-chart-visualizations/about-transforming-commands-and-searches) allow you to change raw event data into useful summaries, statistics, and visualizations. Instead of viewing every individual log, they help analysts aggregate, count, and analyze patterns across many events. Searches that utilize transforming commands are referred to as transforming searches in Splunk.

### General Transformational Commands

| Commande    | Exemple                                                              | Effet                           |
| ----------- | -------------------------------------------------------------------- | ------------------------------- |
| `top`       | index=windowslogs \| top User limit=5                                | Valeurs les plus fréquentes     |
| `rare`      | index=windowslogs \| rare User limit=5                               | Valeurs les moins fréquentes    |
| `highlight` | index=windowslogs \| highlight User EventID Image "Process accessed" | Surligne visuellement (vue Raw) |

![[splunk exploring spl highlight.png]]

#### Stats

https://help.splunk.com/en/splunk-enterprise/spl-search-reference/9.0/search-commands/stats
Calcule des statistiques agrégées sur les champs.

| Fonction | Exemple                   | Explanation                                           |
| -------- | ------------------------- | ----------------------------------------------------- |
| Average  | `stats avg(ProcessCount)` | Calculates the average value of the chosen field      |
| Max      | `stats max(Price)`        | Returns the maximum value of the chosen field         |
| Min      | `stats min(UserAge)`      | Returns the minimum value of the chosen field         |
| Sum      | `stats sum(Cost)`         | Returns the sum of the chosen field values            |
| Count    | `stats count by SourceIp` | Returns the number of occurrences of the chosen field |

spl

```spl
index=windowslogs | stats count by EventID | sort EventID
```

![[splunk exploring spl stats count by eventid.png]]

#### Chart / Timechart

The [chart(opens in new tab)](https://help.splunk.com/en/splunk-enterprise/spl-search-reference/9.0/search-commands/chart) command returns your search results in a table, which you can then use to create helpful visualizations. This command utilizes many of the same functions as `stats`. Let's give it a shot to visualize the `count` of events containing the `User` field with this query.

spl

```spl
index=windowslogs | chart count by User             -- tableau visualisable
```

![[splunk exploring spl chart.png]]

`timechart` : utile pour détecter des tendances, pics, anomalies. `span=` définit l'intervalle.
 [timechart(opens in new tab)](https://help.splunk.com/en/splunk-enterprise/spl-search-reference/9.0/search-commands/timechart)

```spl
index=windowslogs Image!="" | timechart span=30m count by Image limit=5
```

![[splunk exploring spl timechart.png]]
### Data Enrichment and Field Manipulation

#### IP Location 

[iplocation](https://help.splunk.com/en/splunk-enterprise/spl-search-reference/9.0/search-commands/iplocation)

Enrichit les résultats avec des infos géographiques (City, Region, Country) via les tables intégrées Splunk :

spl

```spl
index=windowslogs | iplocation SourceIp | stats count by Country
```

![[splunk exploring spl ip location.png]]

#### Lookup

[lookup(opens in new tab)](https://help.splunk.com/en/splunk-enterprise/spl-search-reference/9.0/search-commands/lookup)

Enrichit les événements via un fichier CSV ou une lookup table externe :

spl

```spl
index=windowslogs
| lookup user_roles Hostname OUTPUT UserRole
| stats count by Hostname UserRole
```

![[splunk exploring spl lookup.png]]

#### Eval

[eval(opens in new tab)](https://help.splunk.com/en/splunk-enterprise/spl-search-reference/9.0/search-commands/eval)

Crée ou modifie des champs à la volée. Supporte `case()` pour la logique conditionnelle :

spl

```spl
index=windowslogs
| eval LogonTypeDesc = case(LogonType == 3, "Network Logon", LogonType == 5, "Service")
| stats count by LogonType LogonTypeDesc
```

The query assigns:

- Network Logon when `LogonType` is 3
- Service when `LogonType` is 5

![[splunk exploring spl eval.png]]

---

## Anomaly Detection

Sometimes you might investigate a data set with lots of different events (e.g., VPN logins) and will need to quickly identify **outliers**, the events that look suspicious compared to the others. For example, imagine a data set of 2,000 VPN logins with just four fields: time of the login, username, source IP, and source country. How would you identify logins from unexpected countries, if field statistics don't show any anomalies?

![[splunk exploring spl anomaly detection.png]]
### Detecting Outliers by Country

`eventstats` = comme `stats` mais préserve les événements bruts. `where` = comme `search` mais plus puissant.

![[splunk exploring spl detecting outliers.png]]

spl

```spl
index=vpnlogs
| eventstats count as logins_by_user by user
| eventstats count as logins_by_user_country by user src_country
| eval country_freq=logins_by_user_country/logins_by_user
| where country_freq < 0.1
| table _time user src_ip src_country country_freq
```

|Ligne|Rôle|
|---|---|
|`eventstats ... by user`|Total logins par user|
|`eventstats ... by user src_country`|Logins par user + pays|
|`eval country_freq=...`|Fréquence du pays pour ce user|
|`where country_freq < 0.1`|Seuil de sensibilité (0.1 = 10%)|

### Detecting Outliers by Hour

Variables calculées :

- `typical_hour` : heure moyenne de connexion de l'employé
- `stdev_hour` : écart-type (prévisibilité) — 0 = très prévisible
- `zscore` : nb d'écarts-types entre l'heure observée et l'heure typique → `zscore > 3` = anomalie

![[splunk exploring spl outliers by hour.png]]

spl

```spl
index=vpnlogs
| eval hour=tonumber(strftime(_time, "%H")) + tonumber(strftime(_time, "%M"))/60
| eventstats avg(hour) as typical_hour stdev(hour) as stdev_hour by user
| eval zscore=abs(hour - typical_hour) / stdev_hour
| where zscore > 3
| eval hour=round(hour, 2), typical_hour=round(typical_hour, 2)
| eval stdev_hour=round(stdev_hour, 2), zscore=round(zscore, 2)
| table _time user src_ip src_country hour typical_hour stdev_hour zscore
```

### ML and Impossible Travel

Détections avancées (ex: Impossible Travel) reposent sur les mêmes commandes + `iplocation` pour la géoloc + `lookup` pour la threat intel + commandes `fit` / `apply` pour le ML.
[Impossible Travel](https://lantern.splunk.com/Security_Use_Cases/Compliance/Running_common_GDPR_compliance_searches/Geographically_improbable_access_detected)

---

## Suite : 
- [Benign](https://tryhackme.com/room/benign)
- [PS Eclipse](https://tryhackme.com/room/posheclipse)
- [Conti](https://tryhackme.com/room/contiransomwarehgh)
- [Volt Typhoon](https://tryhackme.com/room/volttyphoon)

