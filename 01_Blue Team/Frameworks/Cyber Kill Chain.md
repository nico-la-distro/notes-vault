Le framework, créé par Lockheed Martin en 2011, sert à comprendre **comment un attaquant progresse étape par étape** et comment un défenseur peut **casser la chaîne** avant l’objectif final. Il est présenté comme utile pour analyser les intrusions, les ransomwares, les APT et pour repérer les contrôles de sécurité manquants.

![[Cyber Kill Chain.png]]
## Vue d’ensemble des 7 phases

|Phase|But de l’attaquant|À retenir pour les notes|
|---|---|---|
|**1. Reconnaissance**|Collecter un maximum d’informations sur la cible|Phase de recherche/préparation. Peut être **passive** (OSINT, WHOIS, réseaux sociaux, médias, forums) ou **active** (social engineering, scans, banner grabbing). Plus la recon est bonne, plus l’attaque sera crédible et ciblée.|
|**2. Weaponization**|Transformer les infos collectées en arme opérationnelle|L’attaquant prépare le **malware**, l’**exploit** et le **payload**. Exemples : documents Office piégés avec **macros/VBA**, malware sur clé USB, backdoor, templates de phishing crédibles, infra C2.|
|**3. Delivery**|Faire parvenir la charge malveillante à la victime|Vecteurs principaux : **phishing**, **USB drop**, **watering hole**. Le but est d’amener la victime à ouvrir un lien, une pièce jointe ou visiter un site compromis.|
|**4. Exploitation**|Déclencher l’exécution du code malveillant|L’attaquant exploite une vulnérabilité via macro, **zero-day** ou **CVE connue** non patchée. Après l’accès initial, il peut chercher l’élévation de privilèges et le mouvement latéral. Indices : processus anormaux, modifications du registre, nouveaux services, commandes suspectes.|
|**5. Installation**|Maintenir l’accès dans le temps|Mise en place de la **persistance** : **web shell**, backdoor, services Windows modifiés, **Run Keys / Startup Folder**. La room mentionne aussi le **timestomping** pour falsifier les dates de fichiers et gêner la détection/forensic.|
|**6. Command & Control (C2)**|Contrôler la machine compromise à distance|L’hôte infecté “beacon” vers un serveur externe. Canaux fréquents : **HTTP/HTTPS** pour se fondre dans le trafic normal, ou **DNS tunneling**. Une fois le canal ouvert, l’attaquant pilote la victime à distance.|
|**7. Actions on Objectives**|Atteindre le but final de l’attaque|Objectifs typiques : vol d’identifiants, élévation de privilèges, reconnaissance interne, mouvement latéral, **exfiltration de données**, suppression des sauvegardes/**shadow copies**, corruption ou destruction de données.|

## Mémo express par phase

**Reconnaissance**  
C’est la phase la plus discrète. L’attaquant collecte des infos publiques sur l’entreprise, ses employés, ses technos et son exposition externe. La room insiste sur l’**OSINT** et sur l’**email harvesting** pour préparer du phishing. Outils cités : **theHarvester**, **Hunter.io**, **OSINT Framework**.

**Weaponization**  
Les informations collectées deviennent une arme exploitable. Retenir les définitions : **malware** = logiciel malveillant, **exploit** = code exploitant une faille, **payload** = charge exécutée sur la machine cible.

**Delivery**  
Ici, l’attaquant choisit **comment** livrer la charge. Le point important à mémoriser : la livraison peut être **numérique** (mail, lien, site compromis) ou **physique** (clé USB).

**Exploitation**  
C’est le moment où le code s’exécute réellement. L’idée clé : une vulnérabilité non corrigée ou inconnue permet l’accès initial, puis potentiellement l’élévation de privilèges et le pivot interne.

**Installation**  
L’objectif n’est plus juste d’entrer, mais de **rester**. Toute technique de persistance sert à récupérer l’accès plus tard, même si l’accès initial est perdu.

**C2**  
Une machine compromise doit souvent “appeler” l’infrastructure de l’attaquant. Le mot-clé à retenir est **beaconing** : communication régulière entre l’hôte infecté et le serveur C2.

**Actions on Objectives**  
C’est la finalité métier de l’attaque : voler, chiffrer, détruire, se propager, ou saboter. C’est la conséquence visible, mais elle dépend des 6 étapes précédentes.

## Termes-clés à retenir

|Notion|Définition courte|
|---|---|
|**OSINT**|Renseignement à partir de sources ouvertes/publics.|
|**Email harvesting**|Collecte d’adresses mail pour préparer du phishing.|
|**Macro / VBA**|Scripts automatisés dans Office, détournables à des fins malveillantes.|
|**Watering hole**|Compromission d’un site fréquenté par une cible précise.|
|**Zero-day**|Exploit d’une faille inconnue de l’éditeur.|
|**Web shell**|Script malveillant placé sur un serveur web pour garder un accès distant.|
|**Timestomping**|Modification des horodatages des fichiers pour masquer l’activité.|
|**DNS tunneling**|C2 via requêtes DNS régulières vers l’infrastructure de l’attaquant.|
|**Shadow Copy**|Technologie Windows de snapshots/sauvegardes de fichiers/volumes.|
## Limites du modèle

La conclusion de la room rappelle que la **Cyber Kill Chain traditionnelle n’est pas suffisante seule**. Elle date de **2011**, est surtout pensée pour la **protection du périmètre réseau** et les attaques orientées **malware**, et couvre moins bien les menaces modernes combinant plusieurs TTP, ainsi que les **insider threats**. La recommandation est de la compléter avec **MITRE ATT&CK** et **Unified Kill Chain**.