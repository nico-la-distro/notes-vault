
Elastic Stack = ensemble de composants **open-source** pour **collecter**, **traiter**, **stocker/rechercher** et **visualiser** de gros volumes de données **en temps réel**.  
À l’origine orienté **monitoring / performance / recherche**, il est devenu très utilisé en **SOC** (souvent “comme” un **SIEM**).

> En tant qu’analyste SOC : focus sur **analyse de logs & investigations** via ELK, sans devoir maîtriser l’architecture en profondeur (mais connaître les bases aide).

---

## Overview

### Composants clés

![[Elastic Stack - ELBK.png]]

|Composant|Rôle principal|À retenir|
|---|---|---|
|**Elasticsearch**|Moteur de **recherche + analytics** sur documents **JSON**|Stocke / analyse / corrèle, accès via **API REST**|
|**Logstash**|**Pipeline de traitement** (ingestion → normalisation → envoi)|Fichier config en **3 blocs** : `input` / `filter` / `output` + plugins|
|**Beats**|**Agents (data shippers)** sur hôtes|Chaque beat est **spécialisé** (ex: **Winlogbeat** = logs Windows, **Packetbeat** = flux réseau)|
|**Kibana**|**Visualisation web** + exploration|Dashboards, visualisations, investigation en temps réel sur data ES|

**Beats**

![[Elastic Stack - Beats family.png]]

---

### Logstash — structure de config

- **Input** : d’où viennent les données (source ingest)
    
- **Filter** : parsing / **normalisation** (transforme en champs “clé:valeur”)
    
- **Output** : destination (ex: **Elasticsearch**, **Kibana**, port, fichier)
    

---

### Chaîne de fonctionnement (flow)

1. **Beats** collectent des données depuis les endpoints (ex: Winlogbeat / Packetbeat).
    
2. **Logstash** récupère (beats/ports/fichiers), **parse & normalise** en champs.
    
3. **Elasticsearch** stocke (base) et permet **recherche/analyse**.
    
4. **Kibana** affiche et permet de **visualiser/explorer** (charts, timelines, dashboards, etc.).

![[Elastic Stack - Work flow.png]]

---

## Discover Tab (Kibana)

Le **Discover tab** est l’espace où un analyste SOC passe le plus de temps : **voir les logs ingérés**, **chercher**, **investiguer**, et **filtrer** par **termes** + **période de temps**.

---

![[Elastic Stack kibana tabs.png]]

| Élément                         | À quoi ça sert                                                | Usage SOC typique                                 |
| ------------------------------- | ------------------------------------------------------------- | ------------------------------------------------- |
| 1. **Logs (table)**             | Chaque ligne = **1 événement/log** avec champs + valeurs      | Lire un event précis, pivoter sur un champ        |
| 2. **Fields pane (gauche)**     | Liste des **champs normalisés** détectés                      | Ajouter/enlever des champs, filtrer vite          |
| 3. **Index pattern**            | Choisir **quelle source / quel type de logs** explorer        | Ex: sélectionner l’index pattern des **VPN logs** |
| 4. **Search bar**               | Écrire des **requêtes** + appliquer des filtres               | Recherche d’IOC, user, IP, action, etc.           |
| 9. **Add filter**               | Ajouter un filtre sur un champ **sans taper toute une query** | Filtrer sur `user=...`, `src_ip=...`, etc.        |
| 5. **Time filter**              | Restreindre la fenêtre temporelle                             | “Dernières 24h”, “entre 10:00 et 12:00”, etc.     |
| 6. **Timeline / Time interval** | Histogramme **count d’events dans le temps**                  | Détecter **spikes**, zoom sur une période         |
| 7. **Top bar**                  | Sauver / ouvrir recherches, partager, etc.                    | Reprendre une investigation, standardiser         |

---

### Index pattern

![[Elastic Stack index pattern.png]]

- Kibana a besoin d’un **index pattern** pour accéder aux données d’Elasticsearch.
    
- Un index pattern = “**quelles données je veux explorer**” + **propriétés des champs**.
    
- **Un index pattern peut pointer vers plusieurs indices**.
    
- Les sources de logs ayant des structures différentes, on **normalise en champs/valeurs** via un index pattern dédié.
    
- Dans le lab : **`vpn_connections`** = index pattern des logs VPN.
    

---

### Fields pane (gauche)

![[Elastic Stack fields pane.png]]

- Montre les **champs normalisés** disponibles.
    
- Cliquer sur un champ affiche les **top 5 valeurs** + **% d’occurrence**.
    
- Boutons :
    
    - **`+`** : inclure ce champ=valeur (filtre “show only”)
        
    - **`-`** : exclure ce champ=valeur (filtre “NOT”)
        

---

### Time filter + Timeline (histogramme)

![[Elastick Stack time filter.png]]

![[Elastic Stack timeline.png]]

- **Time filter** : filtre sur une période (plein d’options).
    
- **Timeline** : vue “volume d’events dans le temps”.
    
    - Utile pour repérer des **pics** (ex: spike le **11 janvier 2022** dans l’exemple).
        
    - Tu peux **cliquer une barre** pour ne voir que cette période.
        
    - Le compteur indique le **nombre d’events** sur la fenêtre sélectionnée.
        

---

### Create table (réduction du bruit)

Par défaut, les logs sont “raw”. Tu peux :

- ouvrir un log → **sélectionner les champs utiles** → construire une **table** avec seulement ces champs.
    
- avantage : **moins de bruit**, lecture plus rapide, investigation plus propre.
    
- tu peux **sauvegarder** ce format pour le retrouver identique à chaque session.
    

---

## KQL (Kibana Query Language) Overview

**KQL** = langage de requête dans la **Search Bar** de Kibana pour rechercher dans les logs/documents ingérés dans **Elasticsearch**.

![[Elastic Stack KQL.png]]

✅ 2 façons de chercher :

1. **Free text search** (texte libre)
    
2. **Field-based search** (par champ)

---

### 1) Free text search (texte libre)

- Tu tapes un mot/une phrase → Kibana retourne tous les documents qui contiennent ce texte **peu importe le champ**.
    
- Exemple : chercher `United States` → match tous les logs contenant cette expression (dans l’exemple : **2304 hits**).
    

![[Elastic Stack KQL free text search.png]]
#### Point important (piège)

- KQL cherche le **terme complet** :  
    `United` **ne match pas** `United States` → **0 résultat** (dans l’exemple).
    

![[Elastic Stack KQL free text search (piège).png]]

#### Wildcard `*`

- `United*` → match `United` + tout ce qui suit (ex: `United States`, `United Nations`, etc.)
    

![[Elastic Stack KQL wildcard.png]]

---

### Opérateurs logiques (AND / OR / NOT)

|But|Exemple|Effet|
|---|---|---|
|**AND**|`"United States" AND "Virginia"`|logs contenant **les 2**|
|**OR**|`"United States" OR "England"`|logs contenant **l’un ou l’autre**|
|**NOT**|`"United States" AND NOT ("Florida")`|logs US **en excluant** Florida|
**AND**
![[Elastic Stack KQL AND.png]]

**OR**
![[Elastic Stack KQL OR.png]]

**NOT**
![[Elastic Stack KQL NOT.png]]

---

### 2) Field-based search (par champ)

Syntaxe : **`Field : Value`** (séparateur = `:`)

Exemple :  
`Source_ip : 238.163.231.224 AND UserName : Suleman`

➡️ Affiche les logs où :

- `Source_ip` = `238.163.231.224`
    
- **et** `UserName` = `Suleman`
    

💡 Astuce : cliquer dans la search bar affiche la liste des **champs disponibles** utilisables dans tes requêtes.

---

## Kibana - Creating Visualization

Le **Visualization tab** sert à représenter les données sous forme **lisible** : **tables**, **pie charts**, **bar charts**, etc., pour rendre les résultats “présentables”.

---

### Accès + création (essentiel)

- Accéder au tab Visualize/Visualization :
    
    - depuis **Discover** : cliquer sur un **field** → option **visualization** (raccourci).
        
- Tu peux créer **plusieurs types** de viz : table, pie, bar…
    

---

### Corrélation (multi-champs)

Quand tu veux “croiser” plusieurs champs :

- **glisser-déposer** un 2e champ au centre → crée une vue de **corrélation**.
    
- Exemple décrit : corréler **Source_IP** (clients) avec **Source_Country** (pays).
    
    - Résultat possible : **pie chart TOP 5 Source_Country**
        
    - ou **table** “IPs vs country count”
        

![[Elastic Stack kibana vizu.png]]



---

### Sauvegarde (étape la plus importante)

Sans sauvegarde = viz perdue / non réutilisable.

![[Elastic Stack kibana vizu save.png]]

#### Workflow de sauvegarde (checklist)

1. Créer la visualization → **Save** (en haut à droite)
    
2. Renseigner **Title** + **Description** (clairs)
    
3. **Save and add to the library** (pour réutiliser dans dashboards, etc.)
    

---

### Exemple d’usage SOC : Failed connection attempts

Objectif : créer une **table** montrant :

- **User**
    
- **IP address**  
    impliqués dans des **tentatives échouées** (failed attempts)  
    ➡️ utile pour identifier rapidement **qui** attaque / est ciblé et **d’où**.

---

## Kibana - Creating Dashboard

Les **dashboards** donnent une **visibilité globale** sur les logs. On peut en créer plusieurs selon le besoin (ex: **visibilité VPN**).  
Un dashboard = assemblage de **saved searches** + **visualizations**.

---

### Créer un dashboard custom (VPN visibility) — étapes

1. Aller dans l’onglet **Dashboard**
    
2. Cliquer **Create dashboard**
    
3. Cliquer **Add from Library**
    
4. Sélectionner les **visualizations** + **saved searches** → ils s’ajoutent au dashboard
    
5. **Réorganiser / redimensionner** les éléments (layout)
    
6. **Save** le dashboard (important)
    

---

### À retenir

- Sauvegarder d’abord **searches** (Discover) et **visualizations** → ensuite tu les réutilises via **Library**.
    
- Un bon dashboard = **orienté objectif** (ex: “VPN overview”, “Failed logins”, “Top source countries/IPs”, etc.).

