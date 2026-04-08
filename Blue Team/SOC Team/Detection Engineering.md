
La **detection engineering** est le processus continu qui consiste à **concevoir, tester, déployer et maintenir des détections** pour identifier :

- des **activités potentiellement malveillantes**
    
- des **mauvaises configurations**
    
- des comportements anormaux dans un environnement
    

Elle repose aussi sur un **alignement entre les équipes sécurité et le management** afin de construire une défense efficace.

---

# What is Detection Engineering ?
## Les 2 grandes perspectives de détection

|Perspective|Objectif|Catégories|
|---|---|---|
|**Environment-based detection**|Détecter les écarts dans l’environnement à partir de la configuration et du comportement normal|**Configuration Detection**, **Modelling**|
|**Threat-based detection**|Détecter l’activité liée à l’adversaire|**Indicator Detection**, **Threat Behaviour Detection**|

---

## 1) Configuration Detection

Détecter les **écarts de configuration** par rapport à ce qui est attendu dans l’infrastructure.

Exemples :

- réseau
    
- assets
    
- identités
    

### À retenir

On compare l’environnement réel à un **état connu / attendu**.

|Avantages|Limites|
|---|---|
|Facile à créer et maintenir en environnement **statique**|Difficile en environnement **dynamique**|
|Peut détecter beaucoup d’activités malveillantes si la couverture est bonne|Moins efficace si la visibilité est limitée|
|Peut être réalisée par différents profils|Suppose une bonne connaissance de l’infrastructure|
|Utile en complément pour la forensic et la réponse|Changements fréquents = beaucoup de faux positifs|

---

## 2) Modelling

Créer une **baseline** du comportement normal, puis détecter les **déviations**.

Exemples de baseline :

- événements habituels
    
- horaires
    
- volumes de données
    
- seuils d’activité
    

### À retenir

On part du principe que le **malveillant se distingue du bénin** par un écart au comportement normal.

|Avantages|Limites|
|---|---|
|Peut détecter des activités inconnues|Donne peu de contexte sur la menace|
|Facile à maintenir en environnement très statique|Difficile en environnement dynamique|
||Visibilité limitée = efficacité réduite|
||Nécessite une bonne connaissance de l’infra|
||Risque d’intégrer une activité malveillante déjà présente dans la baseline|

---

## 3) Indicator Detection

Détecter à partir d’**indicateurs** (IOC - Indicator of Compromission) observés lors d’analyses ou d’incidents.

Exemples :

- IP malveillante
    
- hash
    
- domaine
    
- artefact connu
    

### À retenir

C’est la détection la plus **rapide à produire**, mais elle dépend fortement de la **stabilité des indicateurs**.

|Avantages|Limites|
|---|---|
|Très rapide à créer et déployer|Dépend de la vitesse de changement de l’adversaire|
|Donne un contexte de menace précis|Réactive : il faut avoir vu l’indicateur avant|
|Utile pour enrichir les données et les détections|Nombre d’indicateurs traitables limité|
|Pratique pour faire du scoping après incident|Expiration / changement des IOC = faux positifs|

---

## 4) Threat Behaviour Detection

Détecter les **TTPs** de l’adversaire (**Tactics, Techniques, Procedures**) plutôt que des indicateurs précis.

### À retenir

On cherche **comment attaque l’adversaire**, pas seulement **avec quoi** il attaque.

|Avantages|Limites|
|---|---|
|Résiste mieux aux changements de l’adversaire|Nécessite beaucoup de données pour bien couvrir|
|Facile à ajuster selon l’environnement|Mise en place initiale plus complexe|
|Peu de faux positifs|Détecte surtout les comportements déjà modélisés|
|S’intègre bien aux playbooks et à l’automatisation|Peut nécessiter des adaptations selon les secteurs|

---

## Idée clé

La meilleure défense repose sur la **combinaison** de plusieurs types de détection.

Exemple :

- **Modelling** pour repérer une anomalie
    
- **Configuration detection** pour confirmer et réduire les faux positifs
    

### À retenir

Aucune méthode n’est parfaite seule.  
**Plusieurs approches combinées = détection plus robuste.**

---

## Detection as Code (DaC)

Le **Detection as Code** consiste à gérer les détections comme du **code**, en appliquant les bonnes pratiques du software engineering.

Objectif :

- rendre les détections **scalables**
    
- améliorer leur **qualité**
    
- suivre les changements plus facilement
    

---

### Ce que DaC apporte

| Élément                | Intérêt                                                                       |
| ---------------------- | ----------------------------------------------------------------------------- |
| **Version Control**    | Suivre les modifications, revoir l’historique, tester et améliorer les règles |
| **Automation / CI/CD** | Automatiser les tests et accélérer le déploiement en production               |

**CI/CD** : (Continuous Integration / Continuous Delivery ou Continuous Deployment)

---

### Bénéfices du Detection as Code

|Bénéfice|Explication|
|---|---|
|**Détections flexibles et personnalisables**|Langages comme **Sigma** ou **YARA** = approche plus agnostique vis-à-vis des outils|
|**Test-Driven Development**|Permet d’identifier plus tôt les faux positifs et angles morts|
|**Collaboration d’équipe**|Favorise le travail commun entre équipes sécurité|
|**Réutilisation du code**|Permet de réemployer des patterns de détection existants|

---

## Résumé ultra-court

- La **detection engineering** = créer et maintenir des détections contre activités malveillantes et mauvaises configurations.
    
- 2 axes :
    
    - **Environment-based** : configuration + baseline
        
    - **Threat-based** : IOC + comportements adverses
        
- 4 types :
    
    - **Configuration Detection**
        
    - **Modelling**
        
    - **Indicator Detection**
        
    - **Threat Behaviour Detection**
        
- Le plus efficace = **combiner plusieurs méthodes**
    
- **Detection as Code** = gérer les détections comme du code avec versioning, tests, automatisation et réutilisation
    

---

## Mini mémo examen

|Notion|Mot-clé|
|---|---|
|Configuration Detection|Écart de configuration|
|Modelling|Déviation par rapport à une baseline|
|Indicator Detection|IOC|
|Threat Behaviour Detection|TTP|
|Detection as Code|Détections gérées comme du code|



![[Detection Engineering workflow.png]]

---

# Detection Engineering Methodologies

**Vue d’ensemble**

|Étape|Objectif|À retenir|
|---|---|---|
|**Detection Gap Analysis**|Trouver les zones mal couvertes|Identifier ce qu’on ne détecte pas encore|
|**Datasource Identification**|Savoir quelles données/logs sont disponibles|Vérifier logs existants, manquants, nécessaires|
|**Baseline Creation**|Définir le comportement normal|Base essentielle pour repérer les anomalies|
|**Log Collection**|Centraliser les logs utiles|Collecte host + réseau + métadonnées|
|**Rule Writing**|Créer des règles de détection|Détecter des patterns anormaux|
|**Deployment / Automation / Tuning**|Mettre en production et améliorer|La détection est un processus continu|

---

## 1) Detection Gap Analysis

Chercher les **faiblesses de détection** dans l’environnement.  
Ici, cela correspond à une forme de **threat modelling**.

|Approche|Description|
|---|---|
|**Reactive**|Se baser sur les incidents internes récents pour voir ce qui a été raté|
|**Proactive**|Utiliser **MITRE ATT&CK** et la threat intelligence pour anticiper les TTPs possibles|

- **Reactive** = on apprend des attaques déjà subies
    
- **Proactive** = on anticipe les attaques possibles
    

---

## 2) Datasource Identification and Log Collection

Identifier les **sources de données pertinentes** selon :

- les threat actors
    
- les TTPs
    
- les risques connus
    

### Objectif

Déterminer :

- quels logs sont déjà disponibles
    
- quels logs manquent
    
- quels logs sont nécessaires pour détecter les menaces
    

### À retenir

Pas de bonne détection sans **bonne visibilité**.

---

## 3) Baseline Creation

Définir ce qui est **normal** dans l’organisation avant de chercher le malveillant.

Cette baseline doit être **mise à jour en continu** et nécessite la participation de plusieurs équipes.

|Type|Description|
|---|---|
|**High-level**|Standards généraux, indépendants de l’OS, guidés par la politique de sécurité|
|**Technical**|Standards techniques liés à l’OS, aux services et aux comportements attendus|

**Exemples de technical baseline**

- hardening OS
    
- activités réseau
    
- politiques IAM
    
- politiques applicatives
    

### À retenir

La baseline sert de **référence** pour distinguer le normal de l’anormal.

---

## 4) Log Collection

Collecter les **logs** et **métadonnées** utiles à la détection.

**Exemples**

- **Network sensors** → données réseau
    
- **Sysmon** → données hôte / endpoint
    
- système centralisé → agrégation des logs
    

### À retenir

La centralisation facilite l’analyse et la corrélation.

---

## 5) Rule Writing

Écrire des règles de détection adaptées :

- à l’infrastructure
    
- aux sources de données
    
- au SIEM / outil utilisé
    

**Exemples**

- **Snort** → trafic réseau
    
- **YARA** → fichiers
    
- **Sigma** → règles génériques sur logs
    

### À retenir

Les règles cherchent des **patterns anormaux** dans les événements journalisés.

---

## 6) Deployment, Automation & Tuning

Une fois testées, les règles sont déployées en production.

Ensuite, elles doivent être :

- ajustées
    
- mises à jour
    
- automatisées si possible
    

**Pourquoi ?**

Parce que :

- l’environnement change
    
- les attaques évoluent
    
- les patterns changent
    

### À retenir

La détection n’est **jamais figée** : c’est un **processus continu**.

---

## Résumé ultra-court

|Élément|Idée clé|
|---|---|
|**Gap Analysis**|Trouver ce qui manque|
|**Datasource Identification**|Savoir quelles données exploiter|
|**Baseline**|Définir le normal|
|**Log Collection**|Collecter les événements utiles|
|**Rule Writing**|Écrire les détections|
|**Deployment / Tuning**|Déployer puis améliorer en continu|

---

# Detection Engineering Frameworks 1

|Framework|Rôle en detection engineering|À retenir|
|---|---|---|
|**MITRE ATT&CK**|Cartographier les **TTPs** d’un adversaire selon la plateforme / l’environnement|Sert à savoir **quoi chercher**|
|**MITRE CAR**|Proposer des **analytics de détection** basées sur les comportements adverses|Complète ATT&CK côté **détection**|
|**Pyramid of Pain**|Mesurer à quel point une détection gêne l’adversaire|Plus on détecte haut dans la pyramide, plus ça lui coûte|
|**Cyber Kill Chain**|Découper une attaque en **phases**|Sert à situer où l’attaque en est|
|**Unified Kill Chain**|Version étendue de la Kill Chain, enrichie par d’autres frameworks|Vision plus complète du cycle d’attaque|

---

## MITRE ATT&CK / CAR

Base de connaissances des **tactiques** et **techniques** utilisées par les attaquants sur :

- Windows
    
- Linux
    
- macOS
    
- Mobile
    

**Utilité :**

- aide à la **detection gap analysis**
    
- permet de mapper les actions adverses à l’environnement
    

_https://attack.mitre.org/_

### MITRE CAR

Base orientée **détection**.

**Utilité :**

- détecter les **comportements adverses**
    
- **prioriser** les analytics en s’appuyant sur ATT&CK
    

_https://car.mitre.org/_

---

## Pyramid of Pain

Montre le **coût pour l’adversaire** selon ce que l’on détecte.

|Ce qu’on détecte|Impact pour l’adversaire|
|---|---|
|IOC simples (hash, IP, domaine)|Faible à moyen|
|Outils / artefacts|Plus gênant|
|**TTPs**|Très coûteux à changer|

**Idée clé :**  
détecter les **TTPs** fait plus mal à l’adversaire que détecter de simples IOC.

![[Detection Engineering pyramide of pain.png]]

---

## Cyber Kill Chain

Décompose une attaque en **7 phases** :

|Ordre|Phase|
|---|---|
|1|Reconnaissance|
|2|Weaponisation|
|3|Delivery|
|4|Exploitation|
|5|Installation|
|6|Command & Control|
|7|Actions on Objectives|

**Utilité :**

- reconnaître les tentatives d’intrusion
    
- placer les détections à différentes étapes de l’attaque
    

![[Detection Engineering cyber kill chain.png]]

---

## Unified Kill Chain

Extension de la Cyber Kill Chain.

**But :**

- couvrir plus complètement le cycle d’attaque
    
- combiner la Kill Chain avec d’autres frameworks, notamment **MITRE ATT&CK**
    

**À retenir :**

- **Cyber Kill Chain** = vue simple en 7 phases
    
- **Unified Kill Chain** = vue plus large et plus détaillée

![[Detection Engineering unified kill chain.png]]

---

## Ressources importantes

- [Pyramid of Pain](https://tryhackme.com/room/pyramidofpainax)
- [Cyber Kill Chain](https://tryhackme.com/room/cyberkillchainzmt)
- [Unified Kill Chain](https://tryhackme.com/room/unifiedkillchain)
- [MITRE](https://tryhackme.com/room/mitre)

---

# Detection Engineering Frameworks 2

|Framework|But|Idée clé|
|---|---|---|
|**ADS Framework**|Structurer la création et la documentation des détections|Produire des alertes **utiles, testées, compréhensibles**|
|**DML Model**|Mesurer la maturité de détection d’une organisation|Plus on monte, plus on détecte des éléments **abstraits / orientés comportement**|

---

## 1) ADS Framework (Palantir)

**Objectif**

Réduire l’**alert fatigue** en concevant des alertes mieux pensées et plus exploitables.

https://github.com/palantir/alerting-detection-strategy-framework

### Flow ADS

|Étape|Rôle|
|---|---|
|**Goal**|Pourquoi créer l’alerte, quel comportement détecter|
|**Categorisation**|Mapper la détection à **MITRE ATT&CK** / kill chain|
|**Strategy Abstract**|Résumé du fonctionnement : quoi détecter, sources, enrichissement, réduction des faux positifs|
|**Technical Context**|Contexte technique : environnement, plateformes, outils|
|**Blind Spots & Assumptions**|Limites, hypothèses, cas de contournement possibles|
|**False Positives**|Cas bénins pouvant déclencher l’alerte|
|**Validation**|Vérifier que la détection déclenche bien sur un vrai positif|
|**Priority**|Définir la priorité de la détection|
|**Response**|Expliquer comment trier et investiguer l’alerte|

![[Detection Engineering ADS framework flow.png]]

### Validation

|Étape|Action|
|---|---|
|1|Préparer un scénario produisant un **true positive**|
|2|Documenter la procédure|
|3|Déclencher l’alerte en environnement de test|
|4|Vérifier que la stratégie fonctionne|

---

## 2) Detection Maturity Level (DML)

**Objectif**

Mesurer la capacité d’une organisation à **utiliser la threat intelligence pour détecter et répondre**.

|Principe|Sens|
|---|---|
|La maturité ne dépend pas seulement de l’intelligence collectée|Elle dépend surtout de sa **mise en application** dans la détection et la réponse|
|Pas de détection = pas de réponse efficace|La réponse dépend d’une capacité de détection existante|
_ressource : https://ryanstillions.blogspot.com/2014/04/the-dml-model_21.html_

---

### Niveaux DML

|Niveau|Nom|Idée|
|---|---|---|
|**DML-8**|Goals|Détecter les **objectifs** de l’adversaire|
|**DML-7**|Strategy|Détecter son **intention / plan global**|
|**DML-6**|Tactics|Détecter une **tactique** sans forcément connaître l’outil|
|**DML-5**|Techniques|Détecter une **technique** spécifique|
|**DML-4**|Procedures|Détecter une **séquence d’actions**|
|**DML-3**|Tools|Détecter les **outils** utilisés|
|**DML-2**|Host & Network Artefacts|Détecter les **artefacts / IOCs observables**|
|**DML-1**|Atomic Indicators|Détecter via **IP, domaines, hash**|
|**DML-0**|None|Aucune capacité de détection|

![[Detection Engineering DML levels.png]]

---

### Logique du DML

|Bas du modèle|Haut du modèle|
|---|---|
|Plus **simple**, plus technique, plus concret|Plus **mature**, plus abstrait, plus orienté renseignement|
|IOC, artefacts, outils|tactiques, stratégie, objectifs|

#### Idée clé

- **DML bas** = détection réactive, technique
    
- **DML haut** = détection plus avancée, contextualisée, orientée comportement
    

---

### Use cases du DML

|Use case|Intérêt|
|---|---|
|**Lexique commun**|Faciliter la communication autour de la menace|
|**Mesurer la maturité face à un acteur**|Savoir jusqu’où on sait le détecter|
|**Évaluer vendors / produits**|Comparer leur niveau réel de détection|
|**Donner du contexte aux règles**|Ajouter un niveau DML à des règles YARA, Snort, SIEM|

---

## Notes rapides

|Élément|À retenir|
|---|---|
|**ADS**|Framework de **construction de détection**|
|**Validation**|Équivalent d’un **unit test** pour une règle|
|**DML**|Framework de **maturité de détection**|
|**DML-1/2**|IOC / artefacts|
|**DML-5/6+**|techniques, tactiques, stratégie, objectifs|