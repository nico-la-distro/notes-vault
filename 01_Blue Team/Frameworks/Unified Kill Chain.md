Le but de la room est de montrer comment le **Unified Kill Chain (UKC)** aide à comprendre une attaque de bout en bout, de la reconnaissance jusqu’à l’atteinte des objectifs de l’attaquant, tout en complétant d’autres frameworks comme **MITRE ATT&CK** et la **Cyber Kill Chain** de Lockheed Martin.

![[Unified Kill Chain (UKC).png]]

## 1) Idée générale

Un **kill chain** décrit les différentes étapes suivies par un attaquant pour compromettre une cible. En cybersécurité, l’intérêt est de **reconstituer ce chemin** pour mieux **prévenir, détecter ou casser** l’attaque avant qu’elle n’aboutisse.

Le **threat modelling** consiste à :

1. identifier les systèmes/applications critiques,
    
2. repérer leurs faiblesses,
    
3. définir un plan de sécurisation,
    
4. mettre en place des mesures pour éviter la réapparition des mêmes vulnérabilités.  
    Le UKC aide ce travail en donnant une vision structurée des surfaces d’attaque et des modes d’exploitation possibles.
    

## 2) Ce qu’il faut retenir sur le UKC

Le **Unified Kill Chain**, publié par **Paul Pols** en **2017**, a été conçu pour **compléter** les autres frameworks, pas pour les remplacer. La room indique qu’il comporte **18 phases**, qu’il est **plus détaillé**, **plus moderne**, et surtout **plus réaliste**, car une attaque ne suit pas toujours une ligne droite : certaines étapes reviennent plusieurs fois pendant la compromission.

### À retenir

|Point clé|Résumé|
|---|---|
|But du UKC|Comprendre toute l’attaque, du repérage à l’objectif final|
|Positionnement|Complémentaire à MITRE ATT&CK / Cyber Kill Chain|
|Force principale|Très détaillé (18 phases)|
|Vision réaliste|Les phases peuvent se répéter pendant l’attaque|
|Intérêt défensif|Aide à analyser, détecter et améliorer les contre-mesures|

Sources pour ce tableau :

---

## 3) Goal: **In** — Initial Foothold

Cette partie correspond à la **prise d’appui initiale** : l’attaquant cherche à entrer dans le système ou le réseau, puis à garder cet accès.

|Phase|Idée essentielle|Exemple typique|
|---|---|---|
|Reconnaissance|Collecte d’informations sur la cible|services exposés, employés, identifiants potentiels, topologie réseau|
|Weaponization|Préparation de l’infrastructure d’attaque|serveur C2, payloads, listener pour reverse shell|
|Social Engineering|Manipulation d’utilisateurs|phishing, fausse page, usurpation d’identité|
|Exploitation|Exploitation d’une faiblesse pour exécuter du code|reverse shell, vulnérabilité web, script détourné|
|Persistence|Maintien de l’accès|service malveillant, backdoor, accès récurrent|
|Defence Evasion|Contournement des mécanismes de défense|AV, IDS, firewall, WAF|
|Command & Control|Communication entre l’attaquant et la machine compromise|exécuter des commandes, voler des données, préparer le pivot|
|Pivoting|Rebond vers d’autres systèmes non accessibles directement|utiliser un serveur exposé pour viser le réseau interne|

### Idée clé

Le but n’est pas juste d’entrer, mais de **rester discret**, **garder l’accès**, puis **se préparer à se déplacer** dans l’environnement.

Sources pour cette section :

---

## 4) Goal: **Through** — Network Propagation

Une fois le premier accès obtenu, l’attaquant essaie d’**étendre sa présence** dans le réseau interne. Il utilise souvent la machine déjà compromise comme **point de pivot** pour découvrir le réseau, élever ses privilèges et atteindre d’autres hôtes.

|Phase|Idée essentielle|Ce que cherche l’attaquant|
|---|---|---|
|Pivoting|Transformer la machine compromise en point d’appui|tunnel vers le réseau interne|
|Discovery|Cartographier le système et le réseau|comptes, permissions, applis, fichiers, partages, config|
|Privilege Escalation|Obtenir des droits plus élevés|SYSTEM/ROOT, admin local, comptes privilégiés|
|Execution|Déployer ou lancer du code malveillant|trojans, scripts C2, scheduled tasks|
|Credential Access|Voler des identifiants|keylogging, credential dumping|
|Lateral Movement|Passer d’une machine à une autre|progression furtive vers les cibles utiles|

### Idée clé

Cette phase correspond à la **propagation interne** : l’attaquant consolide sa présence, **vole des credentials**, **monte en privilèges** et **avance latéralement** jusqu’aux systèmes vraiment intéressants.

Sources pour cette section :

---

## 5) Goal: **Out** — Action on Objectives

C’est la phase où l’attaquant a enfin accès aux **actifs critiques** et cherche à atteindre son but final, souvent lié à la **confidentialité**, l’**intégrité** ou la **disponibilité** du SI.

|Phase|Idée essentielle|Exemple|
|---|---|---|
|Collection|Rassembler les données visées|fichiers, emails, navigateurs, audio/vidéo|
|Exfiltration|Sortir les données du réseau|compression, chiffrement, transfert via C2|
|Impact|Endommager, modifier ou rendre indisponibles les actifs|ransomware, disk wipe, defacement, DoS|
|Objectives|Réaliser le but stratégique de l’attaque|extorsion, sabotage, fuite publique d’informations|

### Idée clé

À ce stade, l’attaquant ne cherche plus seulement l’accès : il veut **voler**, **détruire**, **perturber** ou **monétiser**.

---

## 6) Conclusion

Le message principal de la room est que le **UKC sert à reconstruire une attaque de manière réaliste et détaillée**, ce qui aide les défenseurs à **identifier les risques**, **comprendre les techniques utilisées** et **améliorer leur posture de sécurité**. La room se termine aussi par un exercice pratique, mais le contenu interactif et les réponses nécessitent une connexion.

