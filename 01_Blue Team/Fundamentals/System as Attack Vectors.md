
- Même avec des utilisateurs bien formés (anti-phishing / anti-deepfake), si **le “verrou” (le système)** est faible, un attaquant peut **entrer directement** sans passer par l’utilisateur.
    
- En cybersécurité : des acteurs malveillants peuvent **attaquer des systèmes vulnérables** (serveurs, VM, cloud…) **sans que les utilisateurs ne s’en rendent compte**.
    

---

## Définition de “System”

Un **système** = l’endroit où les données/services sont stockés ou exécutés :

- **Serveur physique**
    
- **Machine virtuelle (VM)**
    
- **Plateforme cloud** (ex : Microsoft 365)
    

**Pourquoi c’est critique :**

- Compromettre **une boîte mail via phishing** → impact limité (1 utilisateur)
    
- Compromettre **le serveur mail** → impact massif (des milliers de boîtes)
    

---

### Valeur d’attaque selon le système (tableau)

|Système compromis|Valeur pour l’attaquant (exemples)|
|---|---|
|PC perso d’un étudiant|Vol de compte (Steam) + ajout au botnet|
|Laptop d’un admin IT senior d’une banque|Accès aux systèmes bancaires internes|
|Serveur mail d’un cabinet pénaliste|Dump de toutes les boîtes + chantage|
|Serveur central d’un réseau industriel|Rançongiciel : chiffrement de tout le réseau|
|Panneau d’admin d’un site gouvernemental|Défiguration / activisme (defacement)|

---
## Attacks on System

- Dans la plupart des attaques “sérieuses”, l’objectif initial est **d’obtenir un accès au système cible**.
    
- Ensuite, ça dépend de la motivation :
    
    - **Vol de données**
        
    - **Déploiement de ransomware**
        
    - **Destruction / sabotage** (parfois sans possibilité de récupération)
        
- Malgré des objectifs différents, **le point de départ est souvent similaire** : 3 grands vecteurs.
    

---
### 1) Human-led attacks (pilotée par l’humain)

**Idée :** l’utilisateur (volontairement ou par erreur) déclenche l’infection/compromission.

- Exemples typiques :
    
    - Brancher une **clé USB malveillante** trouvée (ex : RubberDucky = USB qui exécute des commandes)
        
    - Télécharger des **logiciels piratés** infectés
        
    - **Réutiliser un mot de passe faible** partout
        

**Chiffre clé :**

- **81% des brèches impliquent des mots de passe volés/compromis** → énorme levier d’attaque.
    

---
### 2) Vulnerabilities

**Idée :** les logiciels ont des failles exploitables (et la mauvaise config empire tout).

- Constat :
    
    - En **2024 : >40 000 vulnérabilités publiées**
        
    - **>300** exploitées activement dans des attaques majeures
        
- Facteurs aggravants côté admins :
    
    - **mots de passe faibles**
        
    - **accès trop ouverts** (exposition inutile)
        

_Exemple illustratif (Shodan) : machines anciennes exposées sur Internet → cible facile._

---
### 3) Supply chain (chaîne d’approvisionnement)

**Idée :** compromettre **un éditeur / une librairie / une dépendance**, puis infecter tous les utilisateurs via une mise à jour.

- Pourquoi c’est puissant :
    
    - Un PC a **des centaines d’apps**
        
    - Chaque app dépend de **milliers de librairies**
        
    - 1 composant compromis → diffusion massive
        
- Exemples connus :
    
    - **SolarWinds**
        
    - **3CX**
        
- Menace “émergente” :
    
    - Difficile à prévenir car tu ne contrôles pas tout le logiciel présent
        
    - Même **TryHackMe** a été touché (lib **Lottie Player** utilisée pour des animations)
        

---
### Tableau récap (vecteur → mécanisme → impact)

|Vecteur|Comment ça démarre|Impact typique|
|---|---|---|
|Human-led|USB, piratage, mots de passe réutilisés|Compromission initiale discrète, vol d’identifiants|
|Vulnérabilités|Exploit d’une faille logicielle / mauvaise config|Prise de contrôle, mouvement latéral, ransomware|
|Supply chain|Update “légitime” d’un logiciel/dépendance compromis|Infection en masse multi-organisations|

---

### À retenir (SOC / défense)

- La plupart des attaques commencent par **accès initial** → ensuite **objectif** (vol, ransomware, sabotage).
    
- Les 3 portes d’entrée majeures : **humain**, **failles**, **supply chain**.
    
- Supply chain = **dur à empêcher**, donc il faut être prêt à **détecter + répondre** (incident response).


---
## Vulnerabilties

|Section|Points clés|Exemples / mots-clés|Réponse / actions (SOC)|
|---|---|---|---|
|Vulnérabilités logicielles|Tout logiciel a des failles, parfois découvertes très tard|**Shellshock** : faille présente **1992** → découverte **2014**|Cartographier l’expo (assets/services), surveiller comportements anormaux|
|Pire scénario : découverte attaquant|L’attaquant trouve la faille avant divulgation publique|**Zero-day**|Détection basée sur **traces d’exploitation** (logs, EDR, réseau), chasse aux IOC/TTP|
|Publication officielle|Une fois publique, la faille reçoit un identifiant|**CVE** (_Common Vulnerabilities and Exposures_)|Prioriser selon criticité + exposition, déclencher processus de patch|
|Course de vitesse|Après CVE : attaquants codent des exploits, défenseurs patchent|“Race” exploit vs patch|Patch management + mesures temporaires immédiates|
|Timeline Windows (illustratif)|Exemple de CVE critiques marquantes au fil des ans|**EternalBlue (2017)**, **PrintNightmare (2021)**, **Follina (2022)**, puis CVE critiques chaque année|Leçons : patch rapide, segmentation, durcissement|
|Réponse à une CVE|Réponse durable = **patch éditeur**|Patch = update vendor|Déployer patchs, valider, monitorer post-déploiement|
|Gestion d’un zero-day (avant patch)|En attendant le patch : survivre via mitigations + monitoring|période “stress” avant correctif|**Restreindre accès** (IPs de confiance), appliquer **workarounds vendor**, bloquer patterns sur **IPS/WAF**, surveiller exploitation|

Un **patch** = une **mise à jour fournie par l’éditeur** qui **corrige une vulnérabilité** (ou un bug), en modifiant le code/config du logiciel pour empêcher l’exploitation.

Shellshock : https://www.invicti.com/blog/web-security/cve-2014-6271-shellshock-bash-vulnerability-scan

Zero-day : https://en.wikipedia.org/wiki/Zero-day_vulnerability

---

## Misconfiguration

- Une **misconfiguration** ≠ bug logiciel : c’est une **erreur de paramétrage / déploiement** du système (souvent côté IT).
    
- Souvent faite “pour simplifier” (ex : **mot de passe trop simple** type _1111_).
    

---

### Exemples

|Exemple|Ce que ça illustre|
|---|---|
|MDP “123456” → chats exposés (64M candidatures McDo)|Mots de passe faibles / accès non protégé|
|AWS mal configuré → fuite (106M clients banque)|Cloud mal paramétré (exposition de données)|
|Frigos connectés mal configurés → botnets|IoT exposé / enrôlement silencieux|

---

### Scénario typique

|Étape|Mauvaise pratique|Résultat|
|---|---|---|
|1|Admin met un **MDP faible**|Compte/ressource facile à brute-force|
|2|Base de données **exposée sur Internet**|Surface d’attaque énorme|
|3|Quelques jours plus tard|**Compromission** par des threat actors|

---

### Réponse / prévention (SOC)

- Pas besoin de patch logiciel : il faut **corriger la config**.
    

|Approche proactive|But|
|---|---|
|**Penetration testing**|Simuler une attaque (hackers éthiques) + rapport de failles|
|**Vulnerability scans**|Détecter périodiquement MDP par défaut, soft obsolète, services exposés|
|**Configuration audits**|Revue manuelle vs bonnes pratiques (ex : **CIS benchmarks**)|

---

### À retenir

- Les misconfigs sont **fréquentes** et **très exploitables** car elles créent des “portes ouvertes”.
    
- En SOC, on les voit souvent **après exploitation**, mais en petite structure tu peux aussi faire du **proactif** (scan/audit/pentest).

---

## Ressources

- [The DFIR Report: How Real Intrusions Happen](https://thedfirreport.com/)
- [CISA: Known Exploited Vulnerabilities Catalog](https://www.cisa.gov/known-exploited-vulnerabilities-catalog)
- [BleepingComputer: Latest Supply Chain Attacks](https://www.bleepingcomputer.com/tag/supply-chain-attack/)
- [CheckPoint: Interactive Live Cyber Threat Map](https://threatmap.checkpoint.com/)

