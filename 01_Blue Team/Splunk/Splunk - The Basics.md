| Élément    | Idée clé                                            |
| ---------- | --------------------------------------------------- |
| Splunk     | SIEM leader du marché                               |
| Fonction   | Collecte + analyse + corrélation de logs            |
| Temps réel | Traitement/visibilité immédiate                     |
| Bénéfices  | Meilleure visibilité réseau + détection plus rapide |

---

## Splunk Components

Splunk repose sur **3 composants** qui travaillent ensemble pour **collecter → traiter/stocker → rechercher/visualiser** les logs : **Forwarder**, **Indexer**, **Search Head**.

|Composant|Rôle principal|Entrée|Sortie|
|---|---|---|---|
|**Forwarder**|Agent léger sur l’endpoint : **collecte** les logs et les **envoie**|Sources de logs (machines/services)|Données brutes vers Indexer|
|**Indexer**|**Traite** : parse + normalise en **field=value**, **catégorise**, **stocke** en “events”|Données des forwarders|Events indexés, recherchables|
|**Search Head**|Interface utilisateur : **recherche** (via **SPL**) + **tableaux/visualisations**|Requêtes SPL|Résultats (field=value) + dashboards/charts|

#### **Forwarder**

- **Installé sur l’endpoint** à surveiller (agent léger).
    
- Mission : **collecter** les données et **les transmettre** au **Splunk Indexer**.
    
- **Faible impact** sur les performances (peu de ressources).
    

**Exemples de sources de logs :**

- **Web server** : trafic web
- **Windows** : Event Logs, PowerShell, Sysmon
- **Linux** : logs “host-centric”
- **Database** : requêtes connexions, réponses, erreurs

![[Splunk Forwarder.png]]

#### **Indexer**

- Cœur du traitement : reçoit les données, puis
    
    - **parse** et **normalise** en **paires champ=valeur**
        
    - **catégorise**
        
    - **stocke** sous forme d’**events**
        
- Rend la donnée **facile à rechercher/analyser**.

![[Splunk Indexer.png]]

#### **Search Head**

- L’endroit (Search & Reporting App) où l’utilisateur :
    
    - lance des recherches via **SPL (Search Processing Language)**
        
    - transforme les résultats en **tables** et **visualisations** (pie/bar/column charts)
        
- Fonctionnement : la requête part du Search Head → **Indexer** → retour des **events** (field=value).

![[Splunk Search Head.png]]

---

## Navigating Splunk

L’interface d’accueil Splunk est découpée en **4 zones** : **Splunk Bar**, **Apps Panel**, **Explore Splunk**, **Home Dashboard**.

|Zone|À quoi ça sert|Points clés à retenir|
|---|---|---|
|**Splunk Bar (haut)**|Barre de navigation + actions globales|Permet aussi de **changer d’app** sans passer par Apps Panel|
|**Apps Panel**|Liste des **apps installées**|App par défaut : **Search & Reporting**|
|**Explore Splunk**|Raccourcis utiles|**Add Data**, **Add Apps**, **Documentation**|
|**Home Dashboard**|Tableau de bord d’accueil|Vide par défaut → choisir un dashboard existant ou en créer|

![[Splunk Home screen.png]]

_review the Splunk documentation on Navigating Splunk [here](https://docs.splunk.com/Documentation/Splunk/8.1.2/SearchTutorial/NavigatingSplunk)._

---

## Suite - [Splunk: Exploring SPL](https://tryhackme.com/room/splunkexploringspl)-  [Incident Handling with Splunk](https://tryhackme.com/room/splunk201)- [Investigating With Splunk](http://tryhackme.com/jr/investigatingwithsplunk)- [Benign - Challenge](http://tryhackme.com/jr/benign)- [PoshEclipse - Challenge](http://tryhackme.com/jr/posheclipse)

