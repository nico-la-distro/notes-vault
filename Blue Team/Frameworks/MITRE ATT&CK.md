
**A**dversarial **T**actics, **T**echniques, **a**nd **C**ommon **K**nowledge
## ATT&CK Framework

https://attack.mitre.org/

**MITRE ATT&CK** = base de connaissances **publique** qui recense les **tactiques, techniques et procédures (TTP)** observées dans de vraies attaques.  
Elle sert à **comprendre les adversaires**, **modéliser les menaces**, **défendre**, mais aussi à **simuler des attaques réalistes** en red team.
#### TTP en un coup d’œil

| Terme         | Définition                                 | Idée-clé                 |
| ------------- | ------------------------------------------ | ------------------------ |
| **Tactic**    | Objectif de l’attaquant                    | **Pourquoi** il agit     |
| **Technique** | Méthode utilisée pour atteindre l’objectif | **Comment** il s’y prend |
| **Procedure** | Mise en œuvre concrète de la technique     | **Exécution réelle**     |

_**Tactics** : https://attack.mitre.org/tactics/enterprise/_

_**Technique** : https://attack.mitre.org/techniques/enterprise/_

#### Évolution du framework

- Créé car MITRE voulait **documenter et classer les TTP** des groupes **APT** (advanced persistent threat). 
    
- Au départ centré sur **Windows**.
    
- Puis étendu à **macOS, Linux, cloud**, etc. dans la matrice **Enterprise**.
    
- Il existe aussi des versions pour **Mobile** et **ICS**. https://attack.mitre.org/matrices/mobile/ / https://attack.mitre.org/matrices/ics/
    
- Le framework continue d’évoluer grâce à la **communauté cyber**.

#### ATT&CK Matrix

https://attack.mitre.org/matrices/

https://mitre-attack.github.io/attack-navigator/

![[MITRE ATT&CK matrix.png]]

- Représentation visuelle de toutes les **tactiques** et **techniques**.
    
- Les **tactiques** sont en haut de la matrice.
    
- Sous chaque tactique se trouvent les **techniques**.
    
- Certaines techniques contiennent des **sub-techniques**.
    
- Outil associé utile : **ATT&CK Navigator** pour explorer/annoter la matrice.

https://attack.mitre.org/techniques/T1595/

![[MITRE ATT&CK matrix active scanning.png]]



#### Exemple

|Niveau|Exemple|
|---|---|
|**Tactic**|**Reconnaissance**|
|**Technique**|**Active Scanning**|
|**Sub-techniques**|Scanning IP Blocks / Vulnerability Scanning / Wordlist Scanning|

#### Ce qu’on trouve sur une page ATT&CK

- **Description** de la technique
    
- **ID** de la technique
    
- **Sub-techniques**
    
- **Procedure examples** : groupes, logiciels, campagnes
    
- **Mitigations**
    
- **Detections**
    
- **References**
    

#### À retenir

- **ATT&CK = référentiel des comportements adverses**
    
- **Tactic = objectif**, **Technique = méthode**, **Procedure = exécution**
    
- Très utile pour **blue team** comme pour **red team**
    
- Peut sembler dense au début à cause de la **quantité d’informations**

---

## ATT&CK in Operation

#### Pourquoi ATT&CK est important

- Fournit un **langage standardisé** pour décrire le comportement des attaquants.
    
- Évite les confusions entre plusieurs noms pour une même action.
    
- Les **IDs uniques** facilitent la comparaison entre rapports, incidents et outils.
    
- Améliore la **communication** entre équipes cyber.

#### Lien entre threat intel et défense

- Un rapport de threat intel explique souvent **ce que l’attaquant a fait**.
    
- ATT&CK permet de transformer ça en actions défensives concrètes :
    
    - **détections**
        
    - **requêtes**
        
    - **playbooks**
        
- Idée clé : on **mappe** l’activité observée vers des **TTPs ATT&CK**.

#### Qui utilise ATT&CK ?

|Équipe / rôle|Objectif|Usage d’ATT&CK|
|---|---|---|
|**CTI Teams**|Collecter et analyser la menace|Mapper les comportements observés aux **TTPs** pour créer des profils exploitables|
|**SOC Analysts**|Investiguer / trier les alertes|Relier l’activité à des **tactiques** et **techniques** pour contextualiser et prioriser|
|**Detection Engineers**|Améliorer la détection|Mapper les règles **SIEM / EDR** à ATT&CK|
|**Incident Responders**|Répondre aux incidents|Mapper la chronologie d’un incident aux tactiques/techniques pour visualiser l’attaque|
|**Red / Purple Teams**|Tester les défenses|Construire des scénarios d’émulation alignés sur des techniques et groupes connus|

#### Mapping en action

- Après un incident, il faut **reconstruire les étapes de l’attaque**.
    
- Le **mapping ATT&CK** permet de structurer cette analyse.
    
- But : **mieux comprendre l’attaque** et **mieux se préparer aux futures campagnes**.
    

![[MITRE ATT&CK mapping.png]]

#### Exemple : Mustang Panda

[Mustang Panda](https://attack.mitre.org/groups/G0129/)

 [matrix](https://mitre-attack.github.io/attack-navigator//#layerURL=https%3A%2F%2Fattack.mitre.org%2Fgroups%2FG0129%2FG0129-enterprise-layer.json)

- **Mustang Panda (G0129)** est un groupe déjà mappé dans ATT&CK.
    
- Comportements cités :
    
    - **phishing** pour l’accès initial
        
    - **scheduled tasks** pour la persistance
        
    - **file obfuscation** pour l’évasion
        
    - **ingress tool transfer** pour le command and control


## ATT&CK for Threat Intelligence

- Utiliser ATT&CK pour identifier les **APT** (advanced persistent threat) qui ciblent l’organisation.
    
- Objectif : relever leurs **tactiques / techniques** et voir les **manques de couverture défensive**.
    
- Scénario :
    
    - analyste sécurité
        
    - secteur **aviation**
        
    - migration vers le **cloud**
        
- Méthode :
    
    - chercher un groupe dans **Groups** https://attack.mitre.org/groups/
        
    - analyser son comportement avec **Navigator** + pages **techniques**
        

#### À retenir

- **ATT&CK = outil de recherche de groupes menaçants**
    
- **But final = identifier les gaps défensifs**

![[MITRE ATT&CK for TI.png]]

---

## Cyber Analytics Repository (CAR)

- **CAR (Cyber Analytics Repository)** = base MITRE d’**analytics de détection** basée sur **ATT&CK**. https://car.mitre.org/
    
- But : fournir des détections **validées**, expliquées, et liées à des **TTPs**.
    

### Concret

- Chaque **analytic** montre **comment détecter** un comportement adverse.
    
- CAR aide à transformer les **TTPs ATT&CK** en **détections réelles**.
    
- Exemples d’implémentations selon les outils : **Splunk**, **EQL** (Event Query Language), **LogPoint**.
    

### À trouver dans CAR

|Élément|Rôle|
|---|---|
|**Description**|explique l’analytic|
|**Références ATT&CK**|tactiques / techniques associées|
|**Pseudocode**|logique de détection lisible|
|**Requêtes SIEM**|exemples concrets|
|**Unit Tests**|valider que la détection marche|

### Exemple cité

- **CAR-2020-09-001** : _Scheduled Task - File Access_ https://car.mitre.org/analytics/CAR-2020-09-001/
    

### À retenir

- **CAR = détection pratique à partir d’ATT&CK**
    
- Sert à guider les analystes sur **quoi chercher** et **comment le chercher**
    
- CAR possède aussi sa propre **layer ATT&CK Navigator**

---

## MITRE D3FEND Framework

### D3FEND

(Detection, Denial, and Disruption Framework Empowering Network Defense)

- **ATT&CK** explique **comment l’attaque se déroule**.
    
- **D3FEND** explique **comment la contrer**.
    

### Définition

- **D3FEND** = framework défensif structuré.
    
- Sert à décrire les **techniques de défense** et le rôle des **contrôles de sécurité**.
    
- Il possède sa propre **matrice** ([matrix(opens in new tab)](https://d3fend.mitre.org/)) avec **7 tactiques** :
    
    - Model
        
    - Harden
        
    - Detect
        
    - Isolate
        
    - Deceive
        
    - Evict
        
    - Restore
        

![[MITRE D3FEND matrix.png]]

### Exemple

- **Credential Rotation (D3-CRO)** [Credential Rotation D3-CRO](https://d3fend.mitre.org/technique/d3f:CredentialRotation/) : 
    
    - rotation régulière des mots de passe
        
    - empêche la réutilisation d’identifiants volés
        
    - D3FEND précise **comment la défense fonctionne**, **quoi prendre en compte** et son lien avec **ATT&CK**
        

![[MITRE D3FEND e.g credential rotation D3-CRO.png]]

### À retenir

- **ATT&CK = vision attaquant**
    
- **D3FEND = vision défense**
    
- D3FEND relie les **contre-mesures** aux **techniques offensives**

---

## Other MITRE Projects

- MITRE propose aussi d’autres outils pour **s’entraîner**, **tester les défenses** et **mieux comprendre les attaquants**.
    

### Projets

|Projet|Rôle|
|---|---|
|**Adversary Emulation Library**|bibliothèque gratuite de **plans d’émulation** de groupes réels|
|**Caldera**|outil d’**émulation automatisée** basé sur **ATT&CK** pour tester détection et réponse|
|**AADAPT**|framework sur les menaces visant les **technologies d’actifs numériques** (blockchain, wallets, smart contracts)|
|**ATLAS**|framework sur les menaces visant les systèmes **IA / ML**|

**_ressouces_** : [Adversary Emulation Library(opens in new tab)](https://ctid.mitre.org/resources/adversary-emulation-library/) - [Caldera(opens in new tab)](https://caldera.mitre.org/) - [AADAPT](https://aadapt.mitre.org/) (Adversarial Actions in Digital Asset Payment Technologies) - [ATLAS](https://atlas.mitre.org/)


### À retenir

- **Emulation Library** = guides pas à pas pour reproduire des attaques réelles
    
- **Caldera** = simulation d’attaquants pour exercices **red/blue team**
    
- **AADAPT** = focus **digital assets**
    
- **ATLAS** = focus **IA / machine learning**