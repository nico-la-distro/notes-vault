## **Fonctionnement d'un ordinateur**

**RAM (Mémoire vive):**

- Stocke temporairement les données en cours d'utilisation.
- Plus la RAM est grande, plus l'ordinateur peut gérer de tâches simultanément.
- Types de RAM : DDR3, DDR4, DDR5.

**CPU (Unité centrale de traitement):**

- Exécute les instructions des programmes.
- Détermine la vitesse et la performance de l'ordinateur.
- Caractéristiques : nombre de cœurs, fréquence d'horloge, cache.

**UEFI (Interface micrologicielle extensible unifiée):**

- Remplace le BIOS sur les ordinateurs modernes.
- Offre des fonctionnalités plus avancées, comme le démarrage sécurisé et la gestion des clés de sécurité.

**BIOS (Basic Input Output System):**

- Initialise le matériel et charge le système d'exploitation.
- Permet de configurer certains paramètres du système.

**Disque dur:**

- Stocke les données de manière permanente.
- Types de disques durs : HDD, SSD, NVMe.

**Créer une clé USB bootable**

- Utiliser un outil comme Rufus ou UNetbootin pour copier une image ISO sur une clé USB.
- Permet de démarrer un ordinateur sur un système d'exploitation différent.

**Composants d'une carte mère**

- Socket CPU, slots RAM, ports PCIe, chipset, connecteurs SATA, etc.
- Déterminent les types de composants compatibles avec l'ordinateur.

**Virtualisation**

- Technologie permettant de créer des machines virtuelles sur un seul ordinateur physique.
- Permet de tester des logiciels, d'isoler des environnements et d'économiser des ressources.

**Cloud**

- Stockage et calcul à distance via Internet.
- Offre de nombreux avantages, comme la flexibilité, l'évolutivité et la réduction des coûts.

**Snapshot**

- Capture d'un état précis d'une machine virtuelle.
- Permet de revenir à un état antérieur en cas de problème.

**Hyperviseur**

- Logiciel qui gère les machines virtuelles.
- Exemples : VMware, VirtualBox, Hyper-V.

**Formations Cloud**

- AWS (offre gratuite d'un an), Microsoft Azure, Google Cloud.
- Permettent d'acquérir les compétences nécessaires pour utiliser les services cloud.

## **Equipement**

**Commutateur :**

Un **commutateur** (switch) est un périphérique réseau qui permet de connecter plusieurs appareils entre eux et de les faire communiquer. Il fonctionne au niveau de la **couche 2 du modèle OSI (couche liaison de données)**. Son rôle principal est de **transmettre les trames** entre les ports connectés en fonction de l'adresse MAC de destination.

**Fonctionnalités principales :**

- **Commutation de trames:** Transfert des trames vers le port de destination en fonction de l'adresse MAC.
- **Apprentissage des adresses MAC:** Stockage des adresses MAC des appareils connectés et association avec les ports correspondants.
- **Segmentation du réseau:** Permet de diviser le réseau en plusieurs segments pour limiter le nombre de collisions.
- **Filtrage d'adresses MAC:** Permet de restreindre l'accès au réseau à certains appareils.

**Types de commutateurs :**

- **Commutateurs non manageables:** Configuration simple et fixe.
- **Commutateurs manageables:** Configuration avancée et fonctionnalités supplémentaires (VLAN, QoS, etc.).

**Routeur :**

Un **routeur** est un périphérique réseau qui permet de **relier plusieurs réseaux entre eux** et de **déterminer le cheminement des paquets** en fonction de leur adresse IP. Il fonctionne au niveau de la couche 3 du modèle OSI (couche réseau).

**Fonctionnalités principales :**

- **Routage des paquets:** Déterminer le chemin optimal pour acheminer les paquets vers leur destination.
- **Table de routage:** Stockage des informations de routage pour chaque réseau connecté.
- **Gestion des adresses IP:** Attribution et configuration des adresses IP aux appareils du réseau.
- **Filtrage de paquets:** Permet de restreindre l'accès au réseau et de protéger contre les attaques.

**Types de routeurs :**

- **Routeurs domestiques:** Destinés aux particuliers, configuration simple.
- **Routeurs professionnels:** Offrent des fonctionnalités avancées et une meilleure performance.

**Table de routage :**

Une **table de routage** est une base de données stockée dans le routeur qui contient les informations nécessaires pour **acheminer les paquets vers leur destination**. Elle contient les éléments suivants :

- **Adresse IP de destination:** Réseau ou appareil vers lequel le paquet est envoyé.
- **Masque de sous-réseau:** Permet de déterminer si l'adresse IP de destination est sur le même réseau local ou sur un réseau distant.
- **Passerelle par défaut:** Routeur à utiliser pour atteindre les réseaux distants.
- **Interface de sortie:** Port du routeur par lequel le paquet doit être envoyé.

**Routage statique :**

Le **routage statique** consiste à **configurer manuellement la table de routage** du routeur. Cette méthode est simple mais peut être fastidieuse pour les réseaux complexes.

**Routage dynamique :**

Le **routage dynamique** utilise des protocoles de routage (BGP, OSPF, RIP, etc.) pour **échanger des informations de routage** entre les routeurs. Cette méthode permet de s'adapter automatiquement aux changements de topologie du réseau.

**Pare-feu :**

Un **pare-feu** est un système de sécurité qui **protège un réseau contre les intrusions et les accès non autorisés**. Il fonctionne en **filtrant les paquets** entrants et sortants en fonction de règles prédéfinies.

**Fonctionnalités principales :**

- **Filtrage de paquets:** Autoriser ou bloquer les paquets en fonction de leur adresse IP, port, protocole, etc.
- **Protection contre les attaques:** Détecter et bloquer les attaques connues.
- **Contrôle d'accès:** Restreindre l'accès à certains services et ressources du réseau.

**Types de pare-feu :**

- **Pare-feu matériels:** Offrent une meilleure performance et sécurité.
- **Pare-feu logiciels:** Plus flexibles et moins coûteux.
## **Linux**

- Systèmes d'exploitation libres et open source.
- Offrent une grande sécurité et flexibilité.

**Distributions populaires :** Ubuntu, Debian, Linux Mint.

**Livres utiles :** "How Linux Works", "Linux Basics for Hackers", "The Linux Command Line".

**Fonctionnement du système de fichiers:**

- Linux utilise un système de fichiers arborescents.
- Les fichiers et les répertoires sont organisés en une hiérarchie.
- Les répertoires importants incluent : / (racine), /home (fichiers des utilisateurs), /etc (fichiers de configuration), /bin (commandes binaires), /usr (applications), /var (données variables).

**Commande Sudo:**

- Permet aux utilisateurs non root d'exécuter des commandes avec des privilèges élevés.
- Nécessite la saisie du mot de passe de l'utilisateur root ou d'un membre du groupe sudo.
- Fichiers de configuration importants : /etc/sudoers, /etc/sudoers.d

**Groupes et utilisateurs:**

- Les groupes permettent de regrouper des utilisateurs et de leur accorder des droits spécifiques.
- Les utilisateurs peuvent être membres de plusieurs groupes.
- Commandes utiles : useradd, usermod, groupadd, groupmod, gpasswd.

**Distributions Linux spécialisées**

- **Kali Linux:** pour la sécurité informatique.
- **Parrot OS:** pour la sécurité et l'anonymat.
- **BlackArch:** pour les tests de pénétration.

## **Windows**

- Système d'exploitation propriétaire de Microsoft.
- Largement utilisé pour les jeux et les applications professionnelles.

**Site utile :** Ressources Windows Server.

**Active Directory**

- **Stockage centralisé des informations:** AD stocke des informations sur divers objets réseau, comme les utilisateurs, les ordinateurs, les imprimantes, les groupes et les services.
- **Gestion des accès et des permissions:** Il permet de définir des droits et des permissions pour ces objets, contrôlant ainsi qui peut accéder à quoi et comment.
- **Authentification des utilisateurs:** Lorsqu'un utilisateur se connecte à un domaine Windows, AD vérifie son identité et ses autorisations d'accès.
- **Organisation hiérarchique:** L'information dans AD est organisée de manière hiérarchique, avec des domaines, des unités d'organisation (UO) et des conteneurs. Cela facilite la gestion d'un grand nombre d'objets.
- **Annuaire accessible:** AD utilise le protocole LDAP (Lightweight Directory Access Protocol) pour permettre aux applications et aux services d'accéder aux informations qu'il stocke.

**Contrôleurs de domaine:**

- Serveurs qui stockent la base de données Active Directory.
- Gèrent les utilisateurs, les ordinateurs et les ressources du réseau.

**Configuration de groupes et utilisateurs:**

- Créer des groupes et des utilisateurs pour organiser et gérer les accès aux ressources.
- Définir des droits et des permissions spécifiques pour chaque groupe et utilisateur.
- Utiliser des stratégies de groupe pour simplifier la gestion des configurations.

**Authentification:**

- Processus de vérification de l'identité d'un utilisateur.
- Windows Server prend en charge plusieurs types d'authentification (Kerberos, NTLM, LDAP).
- Configurer l'authentification en fonction des besoins de sécurité du réseau.

**LDAP (Lightweight Directory Access Protocol):**

- Protocole utilisé pour interroger et modifier la base de données Active Directory.
- Permet aux applications et aux services d'accéder aux informations des utilisateurs et des ordinateurs.

**Forêts:**

- Unités d'organisation logique dans Active Directory.
- Contiennent des domaines et des arbres d'arborescence.
- Permettent de gérer de grands réseaux de manière efficace.

## **AD

🖥️ **Windows Domain

**Définition** : Un **domaine Windows** est un regroupement d’utilisateurs et d’ordinateurs gérés de façon centralisée par une entreprise via **Active Directory (AD)**.

- **Objectif** : Centraliser l'administration des utilisateurs, ordinateurs, et politiques de sécurité.

![[Windows Domain.png]]

⚙️ **Composants clés

**Active Directory (AD)** : Répertoire centralisé contenant les informations sur les utilisateurs, ordinateurs, groupes, etc.
**Domain Controller (DC)** : Serveur qui héberge et exécute les services AD.

✅ **Avantages principaux

**Gestion centralisée des identités** : Création et gestion des comptes utilisateurs depuis un seul endroit.
**Déploiement de politiques de sécurité** : Application uniforme des règles de sécurité sur tous les utilisateurs/ordinateurs du domaine.
**Support réseau évolutif** : Adapté aux grandes infrastructures multi-sites.

🎓 **Exemple concret**

Dans une école ou une entreprise :
- Un même identifiant fonctionne sur toutes les machines du réseau.
- Les restrictions (ex. : pas d'accès au panneau de configuration) sont définies via des **politiques de groupe (GPO)** dans AD.

🖥️**Active Directory Domain Services (AD DS)**

AD DS est le **cœur d’un domaine Windows**. Il agit comme un **répertoire centralisé** qui contient tous les objets du réseau (utilisateurs, machines, groupes, imprimantes, partages, etc.).

**Objets principaux d’Active Directory

- **Utilisateurs**  
Objets les plus courants. Représentent des personnes (employés) ou des services (ex : IIS, MSSQL). Ce sont des _security principals_ pouvant être authentifiés et recevoir des privilèges.

- **Machines**  
Chaque ordinateur joint au domaine crée un compte machine (ex : `DC01$`). Ce compte est aussi un _security principal_. Les mots de passe sont gérés automatiquement (120 caractères, rotation automatique).

- **Groupes de sécurité**  
Permettent d’attribuer des droits à plusieurs utilisateurs ou machines en une seule opération. Les groupes peuvent contenir d’autres groupes. Ce sont aussi des _security principals_.
 

Groupes standards :

- **Domain Admins** : contrôle total du domaine
- **Server Operators** : administration des DCs
- **Backup Operators** : accès à tous les fichiers (sauvegardes)
- **Account Operators** : création/modification de comptes
- **Domain Users** : tous les utilisateurs du domaine
- **Domain Computers** : toutes les machines du domaine
- **Domain Controllers** : tous les contrôleurs de domaine

**Outil : Active Directory Users and Computers (ADUC)

- Permet de gérer les objets du domaine depuis le DC. Les objets sont organisés en **Organizational Units (OUs)**.

- Exemples d’OUs : IT, Management, Marketing, R&D, Sales. Les OUs reflètent souvent la structure de l’entreprise.

Conteneurs par défaut :

- **Builtin** : groupes système Windows
- **Computers** : machines nouvellement jointes
- **Domain Controllers** : tous les DCs
- **Users** : utilisateurs/groupes par défaut
- **Managed Service Accounts** : comptes de services

**Différences entre OUs et groupes de sécurité

- **OUs** : servent à **appliquer des politiques (GPO)**. Un utilisateur ou une machine ne peut être dans qu’une seule OU.
- **Groupes** : servent à **gérer les autorisations** sur des ressources. Un utilisateur peut appartenir à plusieurs groupes.

## **Virtualisation et Conteneurisation**

La **virtualisation** et la **conteneurisation** sont deux technologies qui permettent de mieux utiliser les ressources d'un serveur en isolant des applications ou des systèmes, mais elles fonctionnent de manière différente et ont des cas d'utilisation spécifiques.

1. **Virtualisation**

La virtualisation consiste à créer des machines virtuelles (VM) qui simulent des environnements informatiques complets, y compris un système d'exploitation. Cela permet d'exécuter plusieurs systèmes d'exploitation sur un même serveur physique. Voici comment cela fonctionne :

- **Hyperviseur** : Un hyperviseur (comme VMware, Hyper-V, ou KVM) est utilisé pour gérer les machines virtuelles. Il s'exécute directement sur le matériel ou sur un système d'exploitation hôte.
- **Isolation complète** : Chaque machine virtuelle fonctionne de manière indépendante avec son propre système d'exploitation, ses applications et ses fichiers. Cela signifie qu'il peut y avoir plusieurs systèmes d'exploitation (par exemple, Windows, Linux) sur le même matériel.
- **Consommation de ressources** : Les machines virtuelles consomment plus de ressources (CPU, mémoire, stockage) car chaque VM nécessite son propre système d'exploitation et son ensemble de bibliothèques.

Cas d'utilisation de la virtualisation :

- Exécuter des systèmes d'exploitation différents sur une même machine.
- Créer des environnements de développement et de test isolés.
- Optimiser l'utilisation des ressources matérielles dans les centres de données.

2. **Conteneurisation**

La conteneurisation, en revanche, consiste à isoler les applications et leurs dépendances dans des **conteneurs**. Contrairement à la virtualisation, la conteneurisation ne nécessite pas de système d'exploitation complet par conteneur. Tous les conteneurs partagent le noyau du système d'exploitation hôte, mais chaque conteneur fonctionne de manière isolée avec son propre espace utilisateur.

- **Moteurs de conteneurs** : Docker est l'un des moteurs de conteneurisation les plus populaires, mais il en existe d'autres comme Podman ou LXC.
- **Partage du noyau** : Tous les conteneurs partagent le même noyau du système d'exploitation hôte, ce qui les rend beaucoup plus légers que les machines virtuelles.
- **Isolation des processus** : Les conteneurs permettent d'isoler les applications au niveau des processus et des ressources (réseau, système de fichiers) sans nécessiter un hyperviseur.
- **Rapidement déployables** : Les conteneurs démarrent plus rapidement et consomment moins de ressources qu'une machine virtuelle, car ils n'ont pas besoin de lancer un système d'exploitation entier.

Cas d'utilisation de la conteneurisation :

- Déployer des microservices, où chaque conteneur exécute une petite partie d'une application.
- Standardiser les environnements de développement et de production.
- Faciliter la scalabilité des applications dans des environnements cloud.

3. **Différences principales entre la virtualisation et la conteneurisation**

|**Caractéristique**|**Virtualisation (VM)**|**Conteneurisation**|
|---|---|---|
|**Isolation**|Complète, avec un système d'exploitation dédié pour chaque VM|Partagée, les conteneurs utilisent le même noyau du système d'exploitation|
|**Système d'exploitation**|Chaque VM a son propre système d'exploitation complet|Tous les conteneurs partagent le même noyau, mais sont isolés|
|**Performance**|Plus lent à démarrer et nécessite plus de ressources|Démarrage rapide, utilisation plus légère des ressources|
|**Utilisation des ressources**|Moins efficace, plus de duplication (chaque VM a son OS)|Plus efficace, pas besoin de dupliquer l'OS|
|**Cas d'utilisation**|Exécuter différents systèmes d'exploitation|Applications légères et déploiement de microservices|

**En résumé :**

La virtualisation est idéale pour exécuter plusieurs systèmes d'exploitation différents sur un même serveur, tandis que la conteneurisation est mieux adaptée aux applications légères qui partagent le même noyau du système d'exploitation. Les conteneurs sont plus légers et plus rapides à démarrer, mais n'offrent pas le même niveau d'isolation que les machines virtuelles.

![[Virtualisation vs Conteneurisation.png]]
## **Cloud dans la cybersécurité**

Le **cloud** offre de nombreux avantages, mais aussi quelques inconvénients en matière de **cyber sécurité**. Voici un aperçu de ceux-ci :

**Avantages du cloud en cybersécurité**

1. **Sécurité centralisée** :  
    Les fournisseurs de services cloud investissent massivement dans la sécurité pour protéger leurs infrastructures. Ils bénéficient souvent de technologies avancées, de mises à jour automatiques et d'une expertise spécialisée que les entreprises individuelles peuvent avoir du mal à mettre en œuvre en interne.
    
2. **Gestion des correctifs (patching)** :  
    Les fournisseurs cloud appliquent fréquemment des correctifs et des mises à jour de sécurité à leurs systèmes, garantissant que les environnements sont à jour et protégés contre les vulnérabilités récemment découvertes.
    
3. **Redondance et disponibilité** :  
    Le cloud offre une résilience grâce à des mécanismes de redondance (sauvegardes, réplication des données sur plusieurs datacenters) et garantit une haute disponibilité des services, ce qui diminue les risques liés à des pannes locales ou à des attaques qui ciblent spécifiquement une infrastructure.
    
4. **Accès basé sur les rôles** :  
    Les services cloud proposent généralement des contrôles d'accès robustes, comme la gestion des identités et des accès (IAM), qui permettent de restreindre l'accès aux données et aux applications en fonction des besoins des utilisateurs, renforçant ainsi la sécurité.
    
5. **Chiffrement avancé** :  
    Les solutions cloud offrent souvent des fonctionnalités de chiffrement des données en transit et au repos, ce qui limite l'impact en cas d'accès non autorisé.
    
6. **Surveillance et détection des menaces** :  
    Les fournisseurs cloud offrent des outils avancés pour la surveillance continue des menaces, l'analyse des logs et la détection d'anomalies, améliorant ainsi la capacité à identifier rapidement les cyberattaques potentielles.
    

**Inconvénients du cloud en cybersécurité**

1. **Perte de contrôle** :  
    En utilisant des services cloud, une entreprise délègue une partie de la gestion de la sécurité à un tiers. Cela peut poser des problèmes si les politiques de sécurité du fournisseur ne sont pas alignées avec les exigences internes ou si la visibilité sur certains aspects critiques est limitée.
    
2. **Partage de responsabilité** :  
    La sécurité dans le cloud repose sur un modèle de responsabilité partagée. Le fournisseur est responsable de la sécurité de l'infrastructure, mais l'entreprise cliente doit gérer la sécurité de ses données et de ses applications. Cette répartition peut parfois entraîner des malentendus ou des lacunes dans la gestion de la sécurité.
    
3. **Vulnérabilités des API et des interfaces** :  
    Les systèmes cloud sont souvent accessibles via des API, qui, si elles ne sont pas correctement sécurisées, peuvent être des vecteurs d'attaques. La mauvaise configuration des API peut ouvrir des brèches dans la sécurité.
    
4. **Conformité et législation** :  
    Les entreprises doivent s'assurer que leurs données respectent les régulations en matière de protection des données (par exemple, le RGPD en Europe). Le stockage et le traitement des données dans des centres situés dans d'autres pays peuvent poser des problèmes de souveraineté des données.
    
5. **Attaques internes** :  
    Bien que la sécurité soit renforcée pour contrer les menaces externes, des employés malintentionnés chez le fournisseur cloud ou dans l'entreprise cliente pourraient accéder aux données sensibles ou compromettre la sécurité.
    
6. **Risques liés au multitenant** :  
    Les services cloud partagent souvent une infrastructure commune entre plusieurs clients (environnement multitenant). Si une faille dans l'isolation entre les différents locataires se produit, cela pourrait permettre à une organisation malveillante d'accéder aux données d'une autre.
    
7. **Dépendance au fournisseur** :  
    Une entreprise qui devient trop dépendante d'un fournisseur de cloud spécifique peut être vulnérable en cas de défaillance du fournisseur, qu'il s'agisse d'une faille de sécurité, d'une indisponibilité prolongée ou d'un changement dans les politiques de service.
    

**Conclusion :**

Le cloud offre des avantages en matière de sécurité grâce à des technologies et une expertise difficilement accessibles à une seule entreprise. Cependant, il présente aussi des risques liés à la perte de contrôle, la conformité et les attaques ciblant les interfaces ou les fournisseurs eux-mêmes. Une bonne gestion de la sécurité dans le cloud repose sur une compréhension claire du modèle de responsabilité partagée et une vigilance accrue en matière de configuration et de supervision.

## **Backups**

La gestion des backups dans une entreprise est essentielle pour garantir la résilience face aux cyberattaques, comme les ransomwares, ou les défaillances matérielles. Dans le cadre de la cybersécurité, une stratégie efficace de gestion des sauvegardes repose sur plusieurs principes clés :

1. **Principe de la règle 3-2-1 :**

- **3 copies** des données : Une copie principale et deux copies de sauvegarde.
- **2 types de supports différents** : Cela peut inclure des disques durs, des serveurs sur site, ou des services cloud.
- **1 copie hors site** : Stockée dans un endroit géographiquement différent ou dans le cloud, pour éviter la perte en cas de catastrophe locale (incendie, inondation, etc.).

2. **Chiffrement des backups :**

- Il est essentiel de chiffrer les données sauvegardées, que ce soit en transit (lors de la transmission vers l'emplacement de sauvegarde) ou au repos (lorsqu'elles sont stockées), afin de prévenir les accès non autorisés.
- Utiliser des protocoles de chiffrement robustes (AES-256, TLS pour le transfert) permet de sécuriser les données sensibles.

3. **Segmentation des sauvegardes :**

- **Isoler les backups** du réseau principal pour éviter qu'une cyberattaque (comme un ransomware) ne compromette également les sauvegardes.
- Utiliser des **stockages en mode "air-gapped"** (déconnectés physiquement ou logiquement du réseau principal) ou des systèmes de stockage immuables (où les données ne peuvent pas être modifiées après leur création).

4. **Automatisation et tests réguliers :**

- **Automatiser** les processus de sauvegarde pour éviter les erreurs humaines et garantir la continuité des sauvegardes. Des outils de gestion des sauvegardes peuvent planifier des sauvegardes régulières et surveiller leur bon déroulement.
- **Tester régulièrement** la restauration des sauvegardes est crucial. Avoir des sauvegardes corrompues ou non restaurables lors d'une crise peut être catastrophique. Les tests devraient inclure des simulations de scénarios d'attaque ou de panne.

5. **Stratégie de sauvegarde en couches (multi-tiers) :**

- Implémenter une stratégie qui inclut des sauvegardes régulières à des intervalles différents : journalières, hebdomadaires, et mensuelles.
- Utiliser plusieurs technologies (cloud, NAS, disques durs externes) permet de diversifier les options de récupération en fonction du type d’incident.

6. **Gestion des accès aux backups :**

- Limiter les accès aux sauvegardes aux seules personnes ou systèmes qui en ont besoin. L’utilisation d’une **authentification multi-facteurs (MFA)** est également recommandée pour les accès sensibles.
- Implémenter une gestion des droits d’accès stricte (basée sur le principe du **moindre privilège**) pour éviter les fuites ou altérations internes.

7. **Conformité aux régulations :**

- Assurez-vous que la gestion des sauvegardes est conforme aux régulations en vigueur, comme le RGPD pour les entreprises européennes, qui imposent des conditions spécifiques concernant la protection et la conservation des données personnelles.

8. **Plan de reprise après sinistre (DRP) et continuité des opérations :**

- Intégrer la gestion des sauvegardes dans un **plan global de reprise après sinistre**. Ce plan doit préciser les étapes de récupération des données à partir des sauvegardes en cas d’incident.
- Veiller à ce que la durée de restauration (RTO) et la perte maximale acceptable de données (RPO) soient bien définies et correspondent aux besoins de l’entreprise.

9. **Supervision et journalisation :**

- Superviser les systèmes de sauvegarde pour détecter tout comportement anormal ou tentative d’accès suspect. Des **alertes en temps réel** et une journalisation des accès sont des éléments clés de détection proactive.

En adoptant une approche rigoureuse et multi-niveaux pour la gestion des backups, les entreprises peuvent se protéger efficacement contre la perte de données et les impacts des cybermenaces. Une stratégie de sauvegarde bien pensée fait partie intégrante d'une défense solide en matière de cybersécurité.

## **Plan de continuité de l'activité (PCA)**

iso 22301

iso 22300

SMCA

## **Plan de Continuité Informatique (PCI)**


## **Programmation (dans la sécurité)**

**Choisir le bon langage de programmation pour la sécurité informatique est crucial pour maximiser votre efficacité et répondre à vos besoins spécifiques.** Voici une analyse détaillée de six langages populaires et leurs applications dans le domaine de la sécurité :

**1. Python:**

- **Polyvalent et largement utilisé** pour l'automatisation, l'analyse de données et le développement d'outils.
- **Grand nombre de bibliothèques dédiées** à la sécurité, comme Scapy, Nmap et Metasploit.
- **Facile à apprendre** et syntaxe claire, idéal pour les débutants.

**Exemples d'utilisations:**

- Automatisation de la recherche de vulnérabilités.
- Analyse de logs de sécurité.
- Développement de scanners de réseau.

**2. JavaScript:**

- **Principalement utilisé pour le développement web**, mais s'avère également utile pour la sécurité côté client.
- **Peut être utilisé pour injecter du code malveillant** et capturer des données sensibles.
- **Compréhension du fonctionnement de JavaScript** essentielle pour se protéger contre les attaques.

**Exemples d'utilisations:**

- Analyse de code JavaScript pour détecter des vulnérabilités.
- Développement d'extensions de navigateur pour la sécurité.
- Mise en place de protections contre les attaques XSS.

**3. PowerShell:**

- **Langage de script intégré à Windows** offrant un accès puissant aux systèmes et aux applications.
- **Utilisé par les administrateurs système et les pirates informatiques** pour automatiser des tâches.
- **Nécessite une connaissance approfondie de Windows** pour l'utiliser efficacement.

**Exemples d'utilisations:**

- Automatisation de la réponse aux incidents de sécurité.
- Déploiement de correctifs de sécurité.
- Exécution d'analyses de sécurité sur les systèmes Windows.

**4. Perl:**

- **Langage puissant et flexible** souvent utilisé pour l'analyse de données et la manipulation de texte.
- **Utilisé pour développer des outils de sécurité** comme Metasploit et Nmap.
- **Syntaxe particulière** pouvant rebuter les débutants.

**Exemples d'utilisations:**

- Analyse de logs de sécurité.
- Développement de scripts d'attaque.
- Automatisation de tâches de sécurité complexes.

**5. Java:**

- **Langage robuste et portable** utilisé pour développer des applications de sécurité et des frameworks.
- **Large communauté de développeurs** et de nombreuses ressources disponibles.
- **Peut être plus complexe à apprendre** que d'autres langages.

**Exemples d'utilisations:**

- Développement d'applications de pentesting.
- Création d'outils de sécurité réseau.
- Implémentation de solutions de sécurité pour les applications web.

**6. C:**

- **Langage de bas niveau** offrant un contrôle précis sur le fonctionnement du système.
- **Utilisé pour développer des logiciels de sécurité** performants et des outils de bas niveau.
- **Nécessite une connaissance approfondie de la programmation et du fonctionnement des systèmes** pour l'utiliser efficacement.

**Exemples d'utilisations:**

- Développement de scanners de vulnérabilités.
- Création de rootkits et de logiciels malveillants.
- Implémentation de modules de sécurité pour le noyau du système d'exploitation.

## **Protocoles Réseau**

**Protocoles de communication:**

- **HTTP (Hypertext Transfer Protocol):** Transfert de données hypertextes (pages web).
- **DNS (Domain Name System):** Traduction des noms de domaine en adresses IP.
- **TCP (Transmission Control Protocol):** Communication fiable et orientée connexion.
- **UDP (User Datagram Protocol):** Communication non fiable et sans connexion.
- **ARP (Adress Resolution Protocol)** : Communication protocol used for discovering the link layer adress, such as a MAC adress, typically an IPv4 adress.
- **Telnet** : Le nom « Telnet » est l'abréviation de « Teletype Network Protocol ». En bref, **Telnet est un protocole informatique conçu pour interagir avec des ordinateurs distants**. Il permet la communication de terminal à terminal et peut être utilisé à des fins diverses.
- **FTP (File Transfert Protocol) :** 
- **DHCP :** 

![[DHCP handcheck.png]]

**Protocoles de sécurité et d'administration:**

- **SSH (Secure Shell):** Accès distant sécurisé aux machines.
- **RDP (Remote Desktop Protocol):** Contrôle à distance d'un ordinateur.

**Protocoles de messagerie:**

- **IMAP (Internet Message Access Protocol):** Accès aux emails depuis un client.
- **POP (Post Office Protocol):** Téléchargement des emails sur un client.
- **SMTP (Simple Mail Transfer Protocol):** Envoi d'emails.

## **Protocoles de routage**

**RIP (Routing Information Protocol)**

- **Type** : Vecteur de distance
- **Caractéristique clé** : Compte le nombre de sauts (hops) pour déterminer la route.
- **Utilisation** : Simple, mais limité à de petits réseaux avec une taille maximale de 15 sauts.
- **Inconvénient** : Lent à converger (30 secondes de mise à jour) et inefficace pour les grands réseaux.

**OSPF (Open Shortest Path First)**

- **Type** : État de lien
- **Caractéristique clé** : Utilise l’algorithme de Dijkstra pour calculer le chemin le plus court basé sur la bande passante et d’autres facteurs.
- **Utilisation** : Réseaux de moyenne à grande taille, avec une convergence rapide.
- **Avantage** : Très scalable et bien adapté aux grandes entreprises grâce à sa hiérarchisation en « aires ».

**EIGRP (Enhanced Interior Gateway Routing Protocol)**

- **Type** : Hybride (combinaison vecteur de distance/état de lien)
- **Caractéristique clé** : Utilise plusieurs métriques (bande passante, délai, etc.) pour choisir la meilleure route.
- **Utilisation** : Populaire dans les réseaux Cisco. Convergence rapide, simple à configurer.
- **Avantage** : Bon équilibre entre simplicité et performance, adapté aux réseaux d'entreprise de taille moyenne à grande.

**BGP (Border Gateway Protocol)**

- **Type** : Externe (vecteur de chemin)
- **Caractéristique clé** : Utilisé pour échanger des routes entre systèmes autonomes (AS), crucial pour l'Internet.
- **Utilisation** : Routage entre grandes organisations ou fournisseurs d'accès (inter-AS).
- **Avantage** : Très flexible et scalable pour les grandes topologies, essentiel pour l'Internet.

![[Protocoles de routage.png]]

**cf chatgpt tp conv Protocoles de routage détaillé**

## **Protocoles de switch**

**Spanning Tree Protocol (STP)**

- **But** : Éviter les boucles réseau dans un environnement avec des chemins redondants.
- **Fonctionnement** : STP désactive certains liens pour éviter les boucles tout en assurant des chemins de secours en cas de panne.
- **Variante importante** : **Rapid STP (RSTP)**, qui accélère la reconvergence du réseau en cas de changement de topologie.

**Link Aggregation Control Protocol (LACP)**

- **But** : Agréger plusieurs connexions physiques entre deux commutateurs pour augmenter la bande passante et assurer une tolérance aux pannes.
- **Fonctionnement** : Combine plusieurs liens physiques en un seul lien logique, redistribue le trafic si un lien tombe.

**Virtual LAN (VLAN) Trunking Protocol (VTP)**

- **But** : Gérer les VLANs de manière centralisée et synchroniser les informations de VLAN sur tous les commutateurs d’un réseau.
- **Fonctionnement** : Simplifie la configuration des VLANs sur plusieurs commutateurs en mode **Server**, **Client**, ou **Transparent**.

**Port Security (802.1X)**

- **But** : Contrôler l'accès réseau en authentifiant les utilisateurs et appareils.
- **Fonctionnement** : Un appareil doit s'authentifier via un serveur d'authentification (RADIUS) avant de pouvoir accéder au réseau.

**DHCP Snooping**

- **But** : Prévenir les attaques basées sur le DHCP (comme le DHCP spoofing).
- **Fonctionnement** : Filtre les messages DHCP non autorisés, seuls les ports de confiance peuvent répondre aux requêtes DHCP.

**Quality of Service (QoS)**

- **But** : Prioriser certains types de trafic (ex. voix, vidéo) pour garantir leur performance sur le réseau.
- **Fonctionnement** : Classe et priorise le trafic pour que les applications critiques disposent de la bande passante nécessaire.

**cf tp chatgpt con protocoles de switch réseau**


## **Codes HTTP**

**200 OK:**

- **Signification:** La requête a été reçue et traitée avec succès.
- **Exemple:** Un utilisateur visite une page web et le serveur web lui renvoie le code 200 OK, indiquant que la page web a été trouvée et envoyée avec succès.

**403 Forbidden:**

- **Signification:** L'accès à la ressource est interdit.
- **Exemple:** Un utilisateur tente d'accéder à un fichier ou à un dossier qui n'est pas accessible pour son compte, et le serveur web lui renvoie le code 403 Forbidden.

**302 Found:**

- **Signification:** La ressource a été déplacée vers une nouvelle adresse.
- **Exemple:** Un utilisateur visite une page web qui a été déplacée vers une nouvelle URL, et le serveur web lui renvoie le code 302 Found avec la nouvelle URL.

**407 Proxy Authentication Required:**

- **Signification:** Une authentification est nécessaire pour accéder à la ressource via un proxy.
- **Exemple:** Un utilisateur tente d'accéder à une ressource via un proxy qui nécessite une authentification, et le proxy lui renvoie le code 407 Proxy Authentication Required.

**503 Service Unavailable:**

- **Signification:** Le serveur est actuellement indisponible et ne peut pas traiter la requête.
- **Exemple:** Un serveur web est en maintenance et ne peut pas répondre aux requêtes des utilisateurs, et il renvoie le code 503 Service Unavailable.

**Autres codes HTTP courants :

- **400 Bad Request:** La requête est malformée.
- **401 Unauthorized:** L'authentification est nécessaire pour accéder à la ressource.
- **404 Not Found:** La ressource n'a pas été trouvée.
- **500 Internal Server Error:** Une erreur interne s'est produite sur le serveur.
- **502 Bad Gateway:** Le serveur a reçu une réponse invalide d'un autre serveur.

## **DNS (Serveur)**

Un serveur DNS (Domain Name System) est un composant crucial de l'infrastructure d'Internet. Il est responsable de la traduction des noms de domaine lisibles par l'humain (comme `example.com`) en adresses IP (comme `192.0.2.1`) que les ordinateurs et autres appareils utilisent pour communiquer entre eux sur le réseau.

### Étapes de la requête DNS :

#### 1. **Ton ordinateur regarde d’abord en local**

- Il vérifie si **l’adresse IP** du site a déjà été trouvée récemment (dans son **cache DNS local**).
    
- Si **oui** → il utilise cette adresse, **fin du processus**.
    
- Si **non** → il envoie une requête au **Serveur DNS récursif**.
    

---

#### 2. **Le serveur DNS récursif**

- C’est souvent celui de ton **fournisseur Internet (FAI)**, mais tu peux utiliser Google DNS (8.8.8.8), Cloudflare (1.1.1.1), etc.
    
- Il regarde aussi dans **son propre cache**.
    
    - Si l’info y est → il l’envoie à ton ordi, **fin du processus**.
        
    - Si **non**, il commence une **recherche complète**.
        

---

#### 3. **Les serveurs racine (Root Servers)**

- Ces serveurs sont le **point de départ** de la hiérarchie DNS.
    
- Ils ne donnent pas l’adresse IP directement.
    
- Ils te redirigent vers le **serveur du TLD** (Top Level Domain), selon la fin du domaine :
    
    - `.com`, `.fr`, `.org`, etc.
        

---

#### 4. **Le serveur TLD**

- Ce serveur connaît **les noms des serveurs autoritaires** qui gèrent le domaine demandé.
    
- Par exemple, pour `www.tryhackme.com`, il sait que le **serveur de nom** (nameserver) est chez **Cloudflare**.
    

---

#### 5. **Le serveur DNS autoritaire (Nameserver)**

- C’est lui qui contient les **vrais enregistrements DNS** du domaine (`A`, `MX`, `TXT`, etc.).
    
- Il envoie la bonne réponse (ex: l’adresse IP du site) au **serveur DNS récursif**.
    

---

#### 6. **Le serveur récursif donne la réponse à ton ordi**

- Il garde la réponse en **cache** (pendant une durée définie par le **TTL**).
    
- Il envoie l’adresse IP à ton ordinateur.
    
- Ton navigateur peut maintenant se connecter à ce site.
    

---

### ⏱️ Le **TTL (Time To Live)**

- Chaque réponse DNS a un **délai d’expiration**.
    
- Tant que ce temps n’est pas dépassé, ton ordi ou le serveur récursif peut réutiliser l’info **sans refaire tout le trajet**.

![[How does DNS work.png]]
![[How does DNS work 2.png]]

There are two types of TLD, gTLD (Generic Top Level) and ccTLD (Country Code Top Level Domain). Historically a gTLD was meant to tell the user the domain name's purpose; for example, a .com would be for commercial purposes, .org for an organisation, .edu for education and .gov for government. And a ccTLD was used for geographical purposes, for example, .ca for sites based in Canada, .co.uk for sites based in the United Kingdom and so on.

![[Domain Hierarchy.png]]
### DNS Record Types

DNS isn't just for websites though, and multiple types of DNS record exist. We'll go over some of the most common ones that you're likely to come across.

**A Record**

These records resolve to IPv4 addresses, for example 104.26.10.229

**AAAA Record**

These records resolve to IPv6 addresses, for example 2606:4700:20::681a:be5

**CNAME Record**

These records resolve to another domain name, for example, TryHackMe's online shop has the subdomain name [store.tryhackme.com](http://store.tryhackme.com/) which returns a CNAME record [shops.shopify.com](http://shops.shopify.com/). Another DNS request would then be made to [shops.shopify.com](http://shops.shopify.com/) to work out the IP address.

**MX Record**

These records resolve to the address of the servers that handle the email for the domain you are querying, for example an MX record response for [tryhackme.com](http://tryhackme.com/) would look something like [alt1.aspmx.l.google.com](http://alt1.aspmx.l.google.com/). These records also come with a priority flag. This tells the client in which order to try the servers, this is perfect for if the main server goes down and email needs to be sent to a backup server.

**TXT Record**

TXT records are free text fields where any text-based data can be stored. TXT records have multiple uses, but some common ones can be to list servers that have the authority to send an email on behalf of the domain (this can help in the battle against spam and spoofed email). They can also be used to verify ownership of the domain name when signing up for third party services.
## **DNSSEC**

DNSSEC (Domain Name System Security Extensions) est un ensemble de spécifications destinées à renforcer la sécurité du système DNS (Domain Name System). Le DNS est utilisé pour traduire des noms de domaine (comme `example.com`) en adresses IP (comme `192.0.2.1`) que les ordinateurs utilisent pour se connecter entre eux sur Internet. Cependant, le DNS classique ne vérifie pas l'authenticité des réponses qu'il reçoit, ce qui le rend vulnérable à certaines attaques, telles que l'empoisonnement du cache DNS (DNS cache poisoning).

DNSSEC vise à résoudre ces problèmes en ajoutant des mécanismes d'authentification aux enregistrements DNS, via des signatures numériques. Voici comment DNSSEC fonctionne :

1. **Signatures numériques :** Les enregistrements DNS sont signés numériquement avec une clé privée par l'entité qui gère le domaine (typiquement le bureau d'enregistrement ou le propriétaire du domaine)

3. **Chaîne de confiance :** Les clés publiques nécessaires pour vérifier ces signatures sont elles-mêmes signées par une autorité supérieure, créant ainsi une chaîne de confiance remontant jusqu'à une autorité racine. Par exemple, les clés publiques d'un domaine peuvent être signées par le TLD (Top-Level Domain, comme `.com`), qui est ensuite signé par la racine du DNS.

3. **Validation :** Lorsqu'un résolveur DNS (le serveur DNS utilisé par votre ordinateur ou réseau) interroge le DNS pour obtenir l'adresse IP d'un domaine, il vérifie la signature numérique de la réponse à l'aide de la clé publique correspondante. Si la signature est correcte, la réponse est considérée comme valide et authentique. Sinon, la réponse est rejetée.

Grâce à ces mécanismes, DNSSEC empêche des attaquants de falsifier les réponses DNS, protégeant ainsi les utilisateurs contre les redirections malveillantes vers des sites web frauduleux ou des serveurs malveillants.

Cependant, bien que DNSSEC améliore la sécurité, son adoption n'est pas encore universelle, principalement en raison de la complexité de sa mise en œuvre et de la nécessité pour tous les acteurs de la chaîne DNS de le supporter correctement.
## **VPN (Virtual Private Network)**

Un VPN crée un **tunnel chiffré** entre un client et un réseau distant pour transporter du trafic **IP** de façon sécurisée.

**Fonctionnement**  
Connexion → tunnel chiffré → routage IP → sortie réseau distante

**Types de VPN**
- **VPN L3 (routé)** : IP uniquement, pas de broadcast/ARP (cas standard : OpenVPN TUN, WireGuard, IPsec)
- **VPN L2 (bridgé)** : extension du LAN, broadcast/ARP possibles (OpenVPN TAP, L2TP)

**Point clé**  
Accès réseau ≠ appartenance au LAN  
Attaques LAN ⇒ L2 requis  
Labs type TryHackMe ⇒ VPN L3, IP only


## **VLAN**

Un **VLAN** (Virtual Local Area Network) est une technologie réseau qui permet de diviser un réseau physique en plusieurs réseaux logiques distincts. En d'autres termes, un VLAN permet de créer des sous-réseaux virtuels à l'intérieur d'un même réseau physique, comme si chaque sous-réseau était un réseau local indépendant.

**1. Configuration des VLAN sur les commutateurs:**

- Les VLAN sont configurés sur les commutateurs qui prennent en charge cette technologie.
- Chaque VLAN est identifié par un numéro unique (de 1 à 4094).
- Les ports du commutateur sont affectés à un VLAN spécifique.

**2. Marquage des trames :**

- Les trames Ethernet sont encapsulées avec un en-tête VLAN qui contient l'identifiant du VLAN.
- Cet en-tête permet aux commutateurs de diriger les trames vers le VLAN approprié.

**3. Routage entre les VLAN :**

- Les routeurs sont utilisés pour acheminer les paquets entre les VLAN.
- Le routeur doit être configuré avec les informations de routage pour chaque VLAN.

**Types de VLAN :**

- **VLAN par port :** Chaque port du commutateur est affecté à un VLAN spécifique.
- **VLAN par adresse MAC :** Les appareils sont affectés à un VLAN en fonction de leur adresse MAC.
- **VLAN par sous-réseau :** Les appareils sont affectés à un VLAN en fonction de leur adresse IP.

**Avantages des VLAN :**

- **Sécurité et confidentialité accrues :** Les VLAN isolent les groupes d'utilisateurs et de services les uns des autres, ce qui limite les risques d'accès non autorisés et de propagation des attaques.
- **Meilleure gestion du trafic et des ressources :** Les VLAN permettent de segmenter le trafic réseau et de le diriger vers les ressources appropriées.
- **Facilité d'administration du réseau :** Les VLAN simplifient la gestion des groupes d'utilisateurs et de devices.
- **Flexibilité :** Vous pouvez créer et supprimer des VLAN facilement en fonction de vos besoins.
- **Évolutivité :** Les VLAN permettent de faire évoluer votre réseau sans avoir à le reconfigurer entièrement.
- **Réduction des coûts :** Les VLAN peuvent vous aider à réduire les coûts en optimisant l'utilisation des ressources réseau.

**Exemples d'utilisations des VLAN :**

- **Séparer les départements d'une entreprise**
- **Créer un réseau pour les invités**
- **Isoler les appareils IoT**
- **Mettre en place un réseau DMZ (zone démilitarisée)**
- **Déployer des services VoIP**

## **Sous-réseau**

**Fonctionnement d'un sous-réseau :

**1. Division du réseau IP :**

- Un réseau IP est divisé en plusieurs sous-réseaux en utilisant un masque de sous-réseau.
- Le masque de sous-réseau est un nombre binaire qui détermine la partie de l'adresse IP qui est utilisée pour identifier le réseau et la partie qui est utilisée pour identifier l'hôte sur le réseau.

**2. Adresse IP et masque de sous-réseau :**

- Chaque appareil du réseau possède une adresse IP unique.
- L'adresse IP est composée de deux parties : l'adresse du réseau et l'adresse de l'hôte.
- Le masque de sous-réseau est utilisé pour déterminer quelle partie de l'adresse IP est l'adresse du réseau et quelle partie est l'adresse de l'hôte.

**3. Calcul du nombre d'hôtes :**

- Le nombre d'hôtes possibles sur un sous-réseau est déterminé par la taille du masque de sous-réseau.
- Plus le masque de sous-réseau est grand, plus le nombre d'hôtes possibles est petit.

**4. Avantages des sous-réseaux :**

- **Amélioration de la sécurité :** Les sous-réseaux permettent de limiter l'accès aux ressources du réseau à certains groupes d'utilisateurs.
- **Meilleure gestion du trafic :** Les sous-réseaux permettent de segmenter le trafic réseau et de le diriger vers les ressources appropriées.
- **Réduction des collisions :** Les sous-réseaux réduisent le nombre d'appareils qui partagent le même segment de réseau, ce qui diminue le nombre de collisions.
- **Facilité d'administration du réseau :** Les sous-réseaux simplifient la gestion des groupes d'utilisateurs et de devices.

**5. Exemples d'utilisations des sous-réseaux :**

- **Séparer les départements d'une entreprise**
- **Créer un réseau pour les invités**
- **Isoler les appareils IoT**
- **Mettre en place un réseau DMZ (zone démilitarisée)**

## **Masque de sous-réseau**

**Fonctionnement d'un masque de sous-réseau :

**1. Définition :**

Un masque de sous-réseau est un nombre binaire qui permet de **diviser un réseau IP en plusieurs segments plus petits** appelés sous-réseaux. Il fonctionne conjointement avec l'adresse IP pour déterminer :

- **L'adresse du réseau :** La partie de l'adresse IP qui identifie le réseau auquel appartient l'appareil.
- **L'adresse de l'hôte :** La partie de l'adresse IP qui identifie l'appareil spécifique sur le réseau.

**2. Représentation et format :**

- Le masque de sous-réseau est généralement représenté sous forme de quatre nombres décimaux compris entre 0 et 255, séparés par des points.
- Chaque nombre décimal correspond à un octet (8 bits) du masque binaire.
- Par exemple, le masque de sous-réseau 255.255.255.0 est représenté en binaire comme 11111111.11111111.11111111.00000000.

**3. Rôle du masque de sous-réseau :**

- Le masque de sous-réseau est utilisé par les routeurs et les commutateurs pour déterminer à quel sous-réseau appartient un appareil.
- Il permet de **filtrer le trafic réseau** et de ne le diriger que vers les appareils du même sous-réseau.
- Cela permet d'**améliorer la sécurité et les performances du réseau**.

**4. Calcul du nombre d'hôtes :**

- La taille du masque de sous-réseau détermine le nombre d'hôtes possibles sur un sous-réseau.
- Plus le masque de sous-réseau est grand (plus de bits à 1), plus le nombre d'hôtes possibles est petit.
- Des outils en ligne et des formules mathématiques permettent de calculer le nombre d'hôtes en fonction du masque de sous-réseau.

**5. Exemples d'utilisations du masque de sous-réseau :**

- **Séparer les départements d'une entreprise**
- **Créer un réseau pour les invités**
- **Isoler les appareils IoT**
- **Mettre en place un réseau DMZ (zone démilitarisée)**

## **Passerelle par défaut**

**1. Définition :**

Une passerelle par défaut est un **routeur qui permet aux appareils d'un réseau local de communiquer avec des appareils sur d'autres réseaux**. Elle est généralement configurée sur les **cartes réseau** des appareils ou sur le **routeur** lui-même.

**2. Rôle de la passerelle par défaut :**

- Lorsque un appareil envoie un paquet de données à un appareil sur un autre réseau, il utilise l'adresse IP de la passerelle par défaut comme **adresse de destination**.
- La passerelle par défaut **détermine le chemin** que le paquet doit emprunter pour atteindre sa destination.
- Elle peut **acheminer le paquet vers un autre routeur** ou directement vers l'appareil de destination.

**3. Configuration de la passerelle par défaut :**

- La passerelle par défaut est généralement configurée automatiquement par DHCP.
- Il est également possible de la configurer manuellement sur les cartes réseau des appareils ou sur le routeur.

**4. Importance de la passerelle par défaut :**

- La passerelle par défaut est un élément essentiel pour la communication entre les réseaux.
- Sans passerelle par défaut, les appareils d'un réseau local ne peuvent pas communiquer avec les appareils sur d'autres réseaux.

## **Réseau sans fil**

**Fonctionnement d'un réseau sans fil (Wi-Fi) :

**1. Ondes radio :**

Le Wi-Fi utilise des **ondes radio** pour transmettre des données entre les appareils. La fréquence la plus utilisée est de 2,4 GHz, mais la bande de 5 GHz est de plus en plus utilisée car elle offre un débit plus élevé et moins d'interférences.

**2. Points d'accès (AP) :**

Un **point d'accès (AP)** est un appareil qui permet aux appareils de se connecter au réseau Wi-Fi. Il émet et reçoit des ondes radio et convertit les données entre le format radio et le format filaire.

**3. Stations :**

Les **stations** sont les appareils qui se connectent au réseau Wi-Fi, comme les ordinateurs portables, les smartphones, les tablettes, etc. Elles disposent d'une carte réseau Wi-Fi qui leur permet d'envoyer et de recevoir des ondes radio.

**4. Processus de connexion :**

- Pour se connecter à un réseau Wi-Fi, une station recherche les points d'accès disponibles.
- La station choisit un point d'accès en fonction de la force du signal et de la configuration de la sécurité.
- La station s'authentifie auprès du point d'accès en utilisant une clé de sécurité (WEP, WPA, WPA2, etc.).
- Une fois authentifiée, la station peut échanger des données avec le point d'accès et les autres appareils du réseau.

**5. Débits et normes Wi-Fi :**

- Le débit d'un réseau Wi-Fi dépend de la norme utilisée (802.11b, 802.11g, 802.11n, 802.11ac, 802.11ax) et de la configuration du réseau.
- Les nouvelles normes offrent des débits plus élevés et une meilleure couverture.

**6. Sécurité Wi-Fi :**

- La sécurité est un élément important des réseaux Wi-Fi.
- Il est important de choisir une clé de sécurité forte et de configurer le chiffrement des données pour protéger votre réseau contre les intrusions.

**7. Avantages du Wi-Fi :**

- Mobilité : Le Wi-Fi permet aux utilisateurs de se connecter au réseau sans fil, ce qui offre une grande liberté de mouvement.
- Flexibilité : Le Wi-Fi est facile à installer et à configurer.
- Évolutivité : Le Wi-Fi peut être étendu facilement pour couvrir de grandes surfaces.
- Coût : Le Wi-Fi est une solution économique pour connecter des appareils à un réseau.

**8. Inconvénients du Wi-Fi :**

- Portée : La portée d'un réseau Wi-Fi est limitée par la puissance des ondes radio et par les obstacles présents dans l'environnement.
- Interférences : Les ondes radio peuvent être perturbées par d'autres appareils électroniques, tels que les micro-ondes et les babyphones.
- Sécurité : Les réseaux Wi-Fi peuvent être vulnérables aux attaques si la sécurité n'est pas correctement configurée.
## **Adresses IP**

**Types d'adresses IP:**

- **Publiques:** Identifient un ordinateur sur Internet de manière unique et globale. Attribuées par les FAI.
- **Privées:** Identifient un ordinateur au sein d'un réseau local privé. Ne sont pas accessibles depuis Internet.

**Fonctionnement des adresses IP:**

- Composées de 4 octets (0-255) séparés par des points.
- Définissent un numéro unique pour chaque appareil sur un réseau.
- Utilisées pour acheminer les paquets de données vers leur destination.

**Différences entre adresses IP publiques et privées:**

- **Visibilité:** Les adresses IP publiques sont visibles sur Internet, tandis que les adresses IP privées ne le sont pas.
- **Routage:** Les adresses IP publiques peuvent être routées sur Internet, tandis que les adresses IP privées ne le peuvent pas.
- **Utilisation:** Les adresses IP publiques sont utilisées pour se connecter à Internet et à des services externes, tandis que les adresses IP privées sont utilisées pour se connecter à des ressources locales.

**Translation d'adresses IP (NAT)**

**Définition et Fonctionnement:**

La translation d'adresses IP, ou NAT, est un processus qui permet de mapper les adresses IP privées d'un réseau local vers une adresse IP publique unique. Cela permet à plusieurs ordinateurs de partager une seule adresse IP publique pour se connecter à Internet.

**Types de NAT:**

- **NAT statique:** Association d'une adresse IP publique fixe à une adresse IP privée spécifique.
- **NAT dynamique:** Attribution d'une adresse IP publique provenant d'un pool d'adresses à une adresse IP privée selon les besoins.
- **NAT d'adresse de port (PAT):** Traduction des ports TCP/UDP en plus de l'adresse IP pour différencier les connexions provenant de plusieurs ordinateurs derrière la même adresse IP publique.

**Avantages de la NAT:**

- **Conservation des adresses IP:** Réduit le besoin d'adresses IP publiques, précieuses et limitées.
- **Sécurité accrue:** Masque les adresses IP privées et les rend invisibles depuis Internet.
- **Simplicité de configuration:** Facilite la gestion des réseaux locaux.

**Limites de la NAT:**

- **Complexité de la communication entrante:** Nécessite des configurations spécifiques pour les serveurs accessibles depuis Internet.
- **Impact sur les jeux en ligne et les applications peer-to-peer:** Peut perturber le fonctionnement de certains services.

## **IPv4 et IPv6**

**IPv4 (Internet Protocol version 4):**

- Version la plus ancienne et la plus largement utilisée du protocole IP.
- Utilise des adresses de 32 bits, limitant le nombre d'adresses disponibles à environ 4,3 milliards.
- En proie à des problèmes d'épuisement d'adresses et de fragmentation.

**IPv6 (Internet Protocol version 6):**

- Version plus récente du protocole IP conçue pour pallier les limitations d'IPv4.
- Utilise des adresses de 128 bits, offrant un nombre d'adresses pratiquement illimité.
- Intègre des fonctionnalités de sécurité et de performance accrues.

**Différences majeures entre IPv4 et IPv6:**

- **Espace d'adressage:** 32 bits pour IPv4 vs 128 bits pour IPv6.
- **En-têtes de paquets:** Plus simples et plus efficaces dans IPv6.
- **Sécurité:** Intègre des fonctionnalités de sécurité natives dans IPv6.
- **Autoconfiguration d'adresse:** Permet aux appareils de s'autoconfigurer une adresse IP dans IPv6.
- **Mobilité:** Facilite la transition entre différents réseaux dans IPv6.

**Avantages d'IPv6 par rapport à IPv4:**

- **Plus grande capacité d'adressage:** Résout le problème d'épuisement d'adresses.
- **Meilleure sécurité:** Protection contre les attaques et les intrusions.
- **Performances accrues:** Transferts de données plus rapides et plus efficients.
- **Meilleure prise en charge de la mobilité:** Facilite la connectivité des appareils nomades.

**Adoption d'IPv6:**

- Déploiement en cours, mais encore en phase de transition.
- De nombreux fournisseurs d'accès internet et d'entreprises commencent à adopter IPv6.
- Nécessite une mise à jour du matériel et des logiciels pour fonctionner.

![[IPv4 vs IPv6.png]]

## **Modèles OSI et TCP/IP**

**Définition et Fonctionnement:**

Les modèles OSI (Open Systems Interconnection) et TCP/IP (Transmission Control Protocol/Internet Protocol) sont deux frameworks fondamentaux qui décrivent les différentes couches de communication dans un réseau informatique. Ils permettent de visualiser et de comprendre le fonctionnement des réseaux en divisant les processus complexes en plusieurs étapes distinctes.

**Structure des modèles:**

- **Modèle OSI:** 7 couches (physique, liaison de données, réseau, transport, session, présentation, application).
- **Modèle TCP/IP:** 4 couches (accès au réseau, internet, transport, application).

**(Cf OSI model)**

## **WireShark**

**Wireshark** est un outil puissant et gratuit qui permet d'analyser le trafic réseau en temps réel. Il est utilisé par les administrateurs réseau, les développeurs, les professionnels de la sécurité et les étudiants pour :

- **Dépanner les problèmes de connectivité**
- **Analyser les performances du réseau**
- **Identifier les menaces et les attaques**
- **Comprendre les protocoles réseau**
- **Développer des applications réseau**

**Voici les étapes pour analyser un trafic réseau avec Wireshark :**

**1. Installer Wireshark**

Téléchargez et installez Wireshark depuis le site officiel : [https://www.wireshark.org/](https://www.wireshark.org/)
Possibilité de le télécharger (ou non) en ligne de commande sur des OS spécialisées

**2. Démarrer une capture**

Lancez Wireshark et sélectionnez l'interface réseau que vous souhaitez capturer.

**3. Filtrer le trafic**

Utilisez les filtres de Wireshark pour affiner la capture et ne voir que le trafic qui vous intéresse.

**4. Analyser les paquets**

Wireshark affiche une liste des paquets capturés. Vous pouvez double-cliquer sur un paquet pour afficher ses détails.

**5. Exporter les résultats**

Vous pouvez exporter les résultats de la capture dans différents formats pour les analyser ultérieurement.

**Voici quelques conseils pour analyser un trafic réseau avec Wireshark :**

- **Commencez par une capture simple.** N'essayez pas de capturer tout le trafic réseau en même temps. Commencez par une capture filtrée sur un seul protocole ou une seule adresse IP.
- **Utilisez les filtres de Wireshark.** Les filtres sont très puissants et vous permettent de ne voir que le trafic qui vous intéresse.
- **Apprenez à lire les détails des paquets.** Wireshark affiche une grande quantité d'informations pour chaque paquet. Il est important de savoir comment lire ces informations pour comprendre le trafic réseau.
- **Utilisez les outils d'analyse de Wireshark.** Wireshark dispose de nombreux outils d'analyse qui vous permettent de décoder les protocoles réseau, de suivre les flux de conversation et de générer des statistiques.
- **N'hésitez pas à demander de l'aide.** Il existe de nombreuses ressources en ligne pour vous aider à apprendre à utiliser Wireshark. Vous pouvez également poser des questions sur les forums et les listes de diffusion de Wireshark.

**Wireshark est un outil puissant qui peut vous aider à comprendre le trafic réseau et à résoudre les problèmes de connectivité. Il est important de l'utiliser avec précaution et de respecter la confidentialité des données que vous capturez.**

**Ressources utiles pour apprendre à utiliser Wireshark :**

- **Tutoriels Wireshark:** [https://www.youtube.com/watch?v=TkCSr30UojM](https://www.youtube.com/watch?v=TkCSr30UojM)
- **Documentation Wireshark:** [https://www.wireshark.org/docs/](https://www.wireshark.org/docs/)
- **Forums Wireshark:** [https://ask.wireshark.org/](https://ask.wireshark.org/)
- **Listes de diffusion Wireshark:** [https://www.wireshark.org/docs/dfref/](https://www.wireshark.org/docs/dfref/)

## **Serveur de journalisation**

**Définition d'un serveur de journalisation :

Un serveur de journalisation est un **logiciel ou un appareil qui centralise la collecte et le stockage des journaux** provenant de divers systèmes et applications d'un réseau informatique. Il s'agit d'un outil essentiel pour la surveillance, le dépannage et la sécurité des réseaux.

**Fonctions principales d'un serveur de journalisation :**

- **Collecte des journaux:** Le serveur de journalisation reçoit les journaux des sources configurées, généralement via des protocoles standard comme Syslog, SNMP ou TCP/IP.
- **Stockage des journaux:** Les journaux collectés sont stockés sur le serveur de journalisation pour une analyse ultérieure.
- **Analyse des journaux:** Le serveur de journalisation peut analyser les journaux pour identifier les erreurs, les problèmes de performance et les menaces de sécurité.
- **Recherche des journaux:** Le serveur de journalisation permet de rechercher des événements spécifiques dans les journaux stockés.
- **Alerte et notification:** Le serveur de journalisation peut générer des alertes et des notifications en cas d'événements importants détectés dans les journaux.

**Avantages d'utiliser un serveur de journalisation :**

- **Centralisation des journaux:** Simplifie la gestion et l'analyse des journaux en les regroupant dans un emplacement central.
- **Amélioration de la visibilité:** Permet d'avoir une vue d'ensemble de l'activité du réseau et des applications.
- **Dépannage plus facile:** Facilite l'identification des causes des problèmes et la résolution des incidents.
- **Meilleure sécurité:** Aide à détecter les intrusions et les attaques en analysant les journaux pour les événements suspects.
- **Conformité réglementaire:** Aide à répondre aux exigences de conformité en centralisant et en archivant les journaux.

**Configurer et installer un serveur de journalisation :

**1. Choisir un serveur de journalisation:**

Il existe plusieurs serveurs de journalisation disponibles, chacun avec ses propres fonctionnalités et avantages. Voici quelques exemples :

- **Syslog-ng:** Un serveur de journalisation open source et flexible.
- **Graylog:** Un serveur de journalisation open source avec une interface web puissante.
- **Papertrail:** Un serveur de journalisation hébergé avec une interface web intuitive.
- **Loggly:** Un serveur de journalisation hébergé avec des fonctionnalités avancées d'analyse et de recherche.

**2. Installer le serveur de journalisation:**

La procédure d'installation du serveur de journalisation dépend du serveur que vous avez choisi. Veuillez consulter la documentation du serveur pour obtenir des instructions spécifiques.

**3. Configurer les sources de journaux:**

Vous devez configurer les sources de journaux pour envoyer leurs journaux au serveur de journalisation. Cela peut se faire en modifiant la configuration des applications ou des périphériques qui génèrent les journaux.

**4. Configurer le serveur de journalisation:**

Vous devez configurer le serveur de journalisation pour recevoir les journaux, les stocker et les analyser. La configuration du serveur de journalisation dépend du serveur que vous avez choisi. Veuillez consulter la documentation du serveur pour obtenir des instructions spécifiques.

**5. Accéder aux journaux:**

Vous pouvez accéder aux journaux stockés sur le serveur de journalisation via l'interface web du serveur ou en utilisant un outil de ligne de commande.

**Voici quelques conseils pour configurer et installer un serveur de journalisation :**

- **Choisissez un serveur de journalisation qui répond à vos besoins.** Il est important de tenir compte de la taille de votre réseau, du nombre de sources de journaux et des fonctionnalités dont vous avez besoin.
- **Planifiez votre infrastructure de journalisation.** Vous devez décider où installer le serveur de journalisation, comment les sources de journaux enverront leurs journaux au serveur et comment vous allez accéder aux journaux.
- **Testez votre configuration de journalisation.** Assurez-vous que les sources de journaux envoient correctement leurs journaux au serveur et que vous pouvez accéder aux journaux.
- **Surveillez votre serveur de journalisation.** Assurez-vous que le serveur de journalisation dispose de suffisamment d'espace disque et de mémoire pour stocker les journaux.

## **Logiciels de simulation de réseau**

**Un logiciel de simulation de réseau est un outil permettant de créer et de tester des réseaux informatiques virtuels.** Il offre une alternative économique et flexible aux laboratoires physiques, permettant aux utilisateurs d'expérimenter et d'apprendre sans avoir besoin de matériel coûteux.

**Voici quelques-uns des logiciels de simulation de réseau les plus populaires :**

**1. GNS3:**

- **Open source et gratuit**
- Largement utilisé par les étudiants, les professionnels et les passionnés de réseau
- Offre une grande flexibilité et une large gamme d'appareils virtuels
- S'intègre avec des logiciels d'émulation et de virtualisation populaires
- **Points forts :** Flexibilité, communauté active, coût-efficacité
- **Points faibles :** Courbe d'apprentissage abrupte, documentation parfois dispersée

**2. Cisco Packet Tracer:**

- Développé par Cisco
- Offre une interface graphique intuitive et conviviale
- Particulièrement adapté aux débutants et aux étudiants
- Se concentre sur les technologies Cisco
- **Points forts :** Facilité d'utilisation, contenu pédagogique intégré, orientation Cisco
- **Points faibles :** Fonctionnalités moins avancées que GNS3, moins flexible

**3. EVE-NG:**

- **Plateforme open source puissante et évolutive**
- Conçue pour les réseaux de grande taille et complexes
- Offre des fonctionnalités avancées de simulation et d'automatisation
- Large choix d'appareils virtuels et de modules d'apprentissage
- **Points forts :** Fonctionnalités avancées, évolutivité, communauté active
- **Points faibles :** Complexité accrue, courbe d'apprentissage plus prononcée

**4. VirtualBox et VMware:**

- Plateformes de virtualisation populaires
- Peuvent être utilisées pour créer des environnements de réseau virtuel complets
- Offrent une grande flexibilité et un large choix de systèmes d'exploitation
- Nécessitent une configuration manuelle des logiciels de réseau
- **Points forts :** Flexibilité, large choix de systèmes d'exploitation, coût-efficacité
- **Points faibles :** Configuration manuelle, moins intuitif pour la simulation de réseau

**5. Iounix Workplace:**

- Développé par Juniper Networks
- Offre une simulation de haute fidélité des routeurs et des commutateurs Juniper
- Conçu pour les utilisateurs expérimentés et les professionnels de Juniper
- **Points forts :** Simulation précise des technologies Juniper, contenu pédagogique intégré
- **Points faibles :** Accès limité, uniquement pour les technologies Juniper

**Choisir un logiciel de simulation de réseau:**

Le choix du logiciel de simulation de réseau dépend de plusieurs facteurs, tels que :

- **Votre niveau d'expérience**
- **Vos besoins en matière de fonctionnalités**
- **Le type de réseau que vous souhaitez simuler**
- **Votre budget**

## **IDS (Intrusion Detection System)**

Un **détecteur d'intrusion informatique** (IDS - Intrusion Detection System) est un outil de sécurité qui surveille le trafic réseau et les activités sur un système informatique pour détecter les intrusions et les attaques malveillantes. Il fonctionne en analysant les paquets réseau et en recherchant des signatures d'attaque connues, des anomalies de comportement ou des violations des politiques de sécurité.

**Types de détecteurs d'intrusion informatique :**

- **IDS basés sur le réseau (NIDS):** Analysent le trafic réseau pour détecter les attaques et les intrusions.
- **IDS basés sur l'hôte (HIDS):** Surveillent les activités sur un système informatique pour détecter les logiciels malveillants et les modifications non autorisées.
- **IDS hybrides:** Combinent les fonctionnalités des NIDS et des HIDS.

**Fonctionnement d'un détecteur d'intrusion informatique :**

1. **Collecte de données:** Le détecteur d'intrusion collecte des données sur le trafic réseau ou les activités du système.
2. **Analyse des données:** Les données collectées sont analysées à la recherche de signes d'attaque ou d'intrusion.
3. **Génération d'alertes:** Si une attaque ou une intrusion est détectée, le détecteur d'intrusion génère une alerte.
4. **Réponse aux incidents:** Les alertes générées par le détecteur d'intrusion doivent être examinées et une réponse appropriée doit être apportée.

**Avantages des détecteurs d'intrusion informatique :**

- **Détection précoce des intrusions:** Les IDS peuvent détecter les intrusions dès qu'elles se produisent, ce qui permet d'intervenir rapidement et de minimiser les dégâts.
- **Protection contre les attaques connues:** Les IDS peuvent détecter les attaques connues en recherchant des signatures d'attaque dans le trafic réseau ou les activités du système.
- **Détection des anomalies:** Les IDS peuvent détecter les anomalies de comportement qui peuvent indiquer une attaque en cours.
- **Conformité aux réglementations:** Les IDS peuvent aider les organisations à se conformer aux réglementations en matière de sécurité informatique.

**Limites des détecteurs d'intrusion informatique :**

- **Faux positifs:** Les IDS peuvent générer des alertes pour des événements non malveillants.
- **Faux négatifs:** Les IDS peuvent ne pas détecter certaines attaques.
- **Nécessité d'une expertise:** La configuration et la gestion des IDS peuvent nécessiter une expertise en sécurité informatique.

**Différence entre IPS et IDS :**

- **IDS (Intrusion Detection System)** : Se limite à surveiller et alerter. Il n'empêche pas l'intrusion, mais fournit des informations pour que l'administrateur puisse agir.
- **IPS (Intrusion Prevention System)** : Va plus loin en prenant des mesures immédiates pour bloquer l'attaque avant qu'elle ne se propage.

## **IPS (Intrusion Prevention System)

Un **IPS** (Intrusion Prevention System) ou **système de prévention d'intrusion** est un dispositif de sécurité conçu pour détecter et empêcher des attaques malveillantes en temps réel. Il surveille activement le réseau ou les systèmes pour repérer des comportements anormaux ou des signatures d'attaques, et réagit automatiquement pour les bloquer avant qu'ils ne causent des dommages.

**Fonctionnement d'un IPS :**

1. **Surveillance en temps réel** : Un IPS analyse constamment le trafic réseau ou les activités système. Il peut identifier des paquets suspects ou des comportements dangereux grâce à des règles prédéfinies ou des signatures d'attaques connues.
    
2. **Détection des menaces** : Contrairement à un IDS (Intrusion Detection System) qui se contente de détecter des menaces et d'alerter l'administrateur, un IPS intervient directement en bloquant les activités malveillantes détectées. Il peut fonctionner à plusieurs niveaux :
    
    - **Basé sur des signatures** : L'IPS compare le trafic aux bases de données de signatures d'attaques connues.
    - **Basé sur des anomalies** : Il apprend le comportement normal d'un réseau ou d'un système et détecte des écarts.
    - **Basé sur des politiques** : Il suit des règles spécifiques établies par les administrateurs.
3. **Réaction active** : Lorsqu'une menace est identifiée, l'IPS peut prendre différentes actions :
    
    - **Bloquer le trafic** suspect ou malveillant.
    - **Réinitialiser la connexion** pour empêcher une intrusion.
    - **Alerter** l'administrateur ou les systèmes de gestion de la sécurité.

**Différence entre IPS et IDS :**

- **IDS (Intrusion Detection System)** : Se limite à surveiller et alerter. Il n'empêche pas l'intrusion, mais fournit des informations pour que l'administrateur puisse agir.
- **IPS (Intrusion Prevention System)** : Va plus loin en prenant des mesures immédiates pour bloquer l'attaque avant qu'elle ne se propage.

En résumé, un IPS est un dispositif ou un logiciel de sécurité essentiel dans la défense d'un réseau informatique, capable non seulement de repérer des activités malveillantes, mais aussi de les prévenir en temps réel.
## **Serveur Proxy**

Un **serveur proxy** est un intermédiaire entre votre appareil et Internet. Il agit comme un "relais" pour vos requêtes web, masquant votre adresse IP et acheminant votre trafic vers les sites web que vous souhaitez visiter.

**Fonctions principales d'un serveur proxy :**

- **Anonymisation:** Le serveur proxy masque votre adresse IP réelle, ce qui peut vous aider à protéger votre vie privée et à contourner les restrictions géographiques.
- **Sécurité:** Le serveur proxy peut filtrer les contenus malveillants et les attaques web, ce qui peut vous protéger contre les infections et les intrusions.
- **Mise en cache:** Le serveur proxy peut stocker des copies locales des pages web que vous visitez, ce qui peut améliorer la vitesse de chargement des pages.
- **Contrôle d'accès:** Le serveur proxy peut être utilisé pour contrôler l'accès à Internet, par exemple en bloquant certains sites web ou en limitant la bande passante pour certains utilisateurs.

**Types de serveurs proxy :**

- **Serveurs proxy transparents:** Ne modifient pas les requêtes web et ne masquent pas votre adresse IP.
- **Serveurs proxy anonymes:** Masquent votre adresse IP, mais ne modifient pas les requêtes web.
- **Serveurs proxy distants:** Modifient les requêtes web et masquent votre adresse IP.
- **Serveurs proxy inverses:** Cachent les serveurs web réels et protègent leur adresse IP.

**Avantages d'utiliser un serveur proxy :**

- **Protection de la vie privée:** Le serveur proxy peut vous aider à protéger votre vie privée en masquant votre adresse IP.
- **Sécurité accrue:** Le serveur proxy peut vous protéger contre les contenus malveillants et les attaques web.
- **Contournement des restrictions géographiques:** Le serveur proxy peut vous aider à accéder à des sites web qui sont bloqués dans votre région.
- **Amélioration de la vitesse de chargement des pages:** Le serveur proxy peut mettre en cache des copies locales des pages web que vous visitez, ce qui peut améliorer la vitesse de chargement des pages.

**Inconvénients d'utiliser un serveur proxy :**

- **Perte de performance:** Le serveur proxy peut ralentir votre connexion Internet.
- **Problèmes de compatibilité:** Certains sites web ne fonctionnent pas correctement avec les serveurs proxy.
- **Risques de sécurité:** Les serveurs proxy peuvent être utilisés pour des activités malveillantes.

![[Schema serveur Proxy.png]]

![[Schema proxy transparent.png]]
## **Reverse Proxy**

Un **reverse proxy** est un type de serveur proxy qui fonctionne à l'inverse d'un proxy classique. Au lieu de masquer votre adresse IP et de relayer vos requêtes vers des sites web externes, un reverse proxy agit comme un intermédiaire entre les utilisateurs externes et les serveurs internes d'un réseau.

**Fonctions principales d'un reverse proxy :**

- **Sécurité:** Le reverse proxy peut protéger les serveurs internes en masquant leur adresse IP et en filtrant les requêtes malveillantes.
- **Performance:** Le reverse proxy peut améliorer la performance des serveurs internes en mettant en cache les contenus statiques et en répartissant la charge entre plusieurs serveurs.
- **Disponibilité:** Le reverse proxy peut garantir la disponibilité des serveurs internes en redirigeant les requêtes vers un serveur de secours en cas de panne.
- **Facilité d'utilisation:** Le reverse proxy peut simplifier l'accès aux serveurs internes en fournissant un point d'entrée unique pour les utilisateurs externes.

**Avantages d'utiliser un reverse proxy :**

- **Amélioration de la sécurité:** Le reverse proxy peut protéger les serveurs internes contre les attaques et les intrusions.
- **Amélioration de la performance:** Le reverse proxy peut réduire le temps de chargement des pages web et améliorer la réactivité des applications web.
- **Amélioration de la fiabilité:** Le reverse proxy peut garantir la disponibilité des serveurs internes en cas de panne.
- **Simplification de l'administration:** Le reverse proxy peut simplifier la gestion des serveurs internes en centralisant la configuration et la sécurité.

**Inconvénients d'utiliser un reverse proxy :**

- **Coût:** Le reverse proxy peut nécessiter l'achat de matériel et de logiciels supplémentaires.
- **Complexité:** La configuration et la gestion d'un reverse proxy peuvent être complexes.
- **Point de défaillance unique:** Le reverse proxy peut devenir un point de défaillance unique si sa configuration n'est pas redondante.

**Utilisations courantes d'un reverse proxy :**

- **Hébergement web:** Les reverse proxies sont souvent utilisés pour héberger des sites web et des applications web.
- **Serveurs d'applications:** Les reverse proxies peuvent être utilisés pour protéger les serveurs d'applications et les API.
- **Services web:** Les reverse proxies peuvent être utilisés pour sécuriser et mettre en cache les services web.
- **Réseaux privés virtuels (VPN):** Les reverse proxies peuvent être utilisés pour fournir un accès distant aux réseaux privés.

**Exemples de logiciels de reverse proxy :**

- **Nginx**
- **Apache**
- **HAProxy**
- **Varnish**

![[Reverse proxy (intégré dans FW).png]]

![[Reverse proxy (serveur dédié).png]]
## **Burp Suite**

**Burp Suite** est un ensemble d'outils utilisés principalement pour les tests de sécurité des applications web. Développé par **PortSwigger**, Burp Suite est largement utilisé par les professionnels de la sécurité, les testeurs d'intrusion (pentesters), et les développeurs pour identifier, analyser, et exploiter les vulnérabilités dans les applications web.

**Fonctionnalités principales de Burp Suite

1. **Intercepting Proxy** : Burp Suite fonctionne comme un proxy HTTP/HTTPS qui intercepte et modifie les requêtes et réponses entre votre navigateur web et le serveur web cible. Cela permet aux testeurs de manipuler et d'analyser les données échangées pour trouver des vulnérabilités.
2. **Scanner** : La version professionnelle de Burp Suite inclut un scanner automatique qui détecte des vulnérabilités courantes telles que les injections SQL, les failles XSS (Cross-Site Scripting), et bien d'autres.
3. **Repeater** : Cet outil permet de réexécuter et de modifier des requêtes HTTP manuellement pour tester les réponses du serveur. C'est utile pour tester les failles logiques ou les vulnérabilités qui nécessitent une interaction spécifique.
4. **Intruder** : L'Intruder est un outil de force brute permettant d'automatiser les tests sur les paramètres d'une requête pour découvrir des vulnérabilités comme le bruteforcing de mots de passe ou l'injection de payloads malveillants.
5. **Sequencer** : Utilisé pour analyser l'entropie des jetons de session et autres identifiants afin de vérifier leur prévisibilité, ce qui pourrait indiquer une faiblesse dans la sécurité.
6. **Comparer** : Cet outil compare deux versions de données pour identifier des différences subtiles, ce qui peut aider à détecter des modifications d'état inattendues ou des failles.
7. **Decoder** : Le Decoder permet de décoder et d'encoder des données dans différents formats (comme Base64, URL encoding, etc.) afin de les analyser plus facilement.
8. **Extensibilité** : Burp Suite est extensible via des plugins écrits en Java, Python, ou Ruby. Cela permet d'ajouter des fonctionnalités personnalisées pour répondre à des besoins spécifiques.

Burp Suite est souvent utilisé lors des **tests d'intrusion** (pentests) pour identifier les failles de sécurité dans les applications web. Les testeurs peuvent intercepter le trafic, manipuler les requêtes et réponses, effectuer des attaques de type injection, bruteforce, et bien plus encore, pour évaluer la sécurité d'une application.

**Exemple d'usage :

- Intercepter une requête de connexion à un site web.
- Modifier les paramètres de la requête pour tester si l'application est vulnérable à une injection SQL.
- Utiliser l'outil Repeater pour tester différentes valeurs manuellement.
- Analyser les réponses du serveur pour trouver des indicateurs de vulnérabilités.

Site officiel et Plugin burpsuit : https://portswigger.net/bappstore
## **Pfsense**

**pfSense est un pare-feu et routeur open source basé sur FreeBSD.** Il est utilisé pour sécuriser les réseaux domestiques et professionnels en filtrant le trafic réseau et en bloquant les accès non autorisés.

**Quelques fonctionnalités de pfSense :**

- **Filtrage de paquets avancé:** pfSense utilise le moteur de filtrage de paquets PF pour contrôler le trafic réseau entrant et sortant.
- **VPN:** pfSense prend en charge plusieurs types de VPN, tels que IPsec, OpenVPN et WireGuard.
- **Captive Portal:** pfSense peut être utilisé pour créer un portail captif pour les utilisateurs invités.
- **Gestion des adresses IP:** pfSense peut être utilisé pour gérer les adresses IP et les noms de domaine.
- **Rapports et statistiques:** pfSense fournit des rapports et des statistiques détaillés sur le trafic réseau.

**pfSense est une solution de sécurité puissante et flexible qui peut être utilisée pour protéger les réseaux de toutes tailles.** Il est gratuit et open source, ce qui le rend accessible à tous.

**Voici quelques avantages d'utiliser pfSense :**

- **Sécurité accrue:** pfSense peut aider à protéger votre réseau contre les attaques et les intrusions.
- **Flexibilité:** pfSense peut être configuré pour répondre à vos besoins spécifiques.
- **Coût:** pfSense est gratuit et open source.
- **Communauté active:** pfSense dispose d'une communauté active d'utilisateurs et de développeurs qui peuvent vous aider à résoudre les problèmes et à trouver des solutions.

## **Snort**

**Snort est un système de détection d'intrusion (IDS) et de prévention d'intrusion (IPS) open source et gratuit.** Il est utilisé pour surveiller le trafic réseau et détecter les activités malveillantes.

**Quelques fonctionnalités de Snort :**

- **Détection d'attaques connues:** Snort peut détecter les attaques connues en recherchant des signatures d'attaque dans le trafic réseau.
- **Détection d'anomalies:** Snort peut détecter les anomalies de comportement qui peuvent indiquer une attaque en cours.
- **Prévention d'intrusion:** Snort peut bloquer les attaques en bloquant le trafic réseau malveillant.
- **Journalisation et analyse:** Snort peut journaliser le trafic réseau et les événements de sécurité pour une analyse ultérieure.

**Snort est un outil puissant et flexible qui peut être utilisé pour protéger les réseaux de toutes tailles.** Il est gratuit et open source, ce qui le rend accessible à tous.

**Quelques avantages d'utiliser Snort :**

- **Sécurité accrue:** Snort peut aider à protéger votre réseau contre les attaques et les intrusions.
- **Flexibilité:** Snort peut être configuré pour répondre à vos besoins spécifiques.
- **Coût:** Snort est gratuit et open source.
- **Communauté active:** Snort dispose d'une communauté active d'utilisateurs et de développeurs qui peuvent vous aider à résoudre les problèmes et à trouver des solutions.

**Solution de sécurité puissante et flexible pour votre réseau, Snort est une excellente option.

## **EDR**

Endpoint Detection and Response (EDR) is cybersecurity protection software that detects threats on end-user devices (endpoints) in an organization. Across a large, clamorous, worldwide arena of cybersecurity solutions, EDR stands out as a distinct category of telemetry tools that provide continuous monitoring of endpoints to identify and manage adversarial cyber threats such as malware and ransomware.

EDR technology is also sometimes referred to as endpoint detection and threat response (EDTR).

As a cyber telemetry tool, EDR solutions collect data from endpoints as part of threat monitoring and can correlate data from across an entire infrastructure, including its endpoint tools and applications. So EDR tools can be very powerful as threat protection and attack context technologies and formidable endpoint security measures.

![[EDR Functions.png]]
![[EDR Key Functions.png]]
## **XDR**

**Extended Detection and Response (XDR)** est une solution de qui intègre et corrèle automatiquement des données provenant de multiples sources de sécurité pour améliorer la détection, l'analyse, la réponse et la prévention des menaces. Contrairement aux solutions traditionnelles qui se concentrent sur un seul vecteur de menace (comme EDR pour les terminaux), XDR combine des données provenant de différentes couches de sécurité, telles que les terminaux, les réseaux, les e-mails, les serveurs et les applications.

**Avantages de XDR** :

- **Corrélation de données** : Il corrèle les alertes provenant de diverses sources pour une meilleure détection des menaces.
- **Réduction du bruit** : En regroupant les alertes liées, il réduit le nombre de faux positifs.
- **Réponse automatisée** : XDR peut automatiser les réponses aux menaces, réduisant ainsi le temps de réaction.
- **Visibilité accrue** : Il offre une vue unifiée des menaces sur l'ensemble de l'environnement informatique, permettant une meilleure prise de décision.

![[OPEN XDR.png]]

## **EDR, MDR, NDR, XDR**

![[EDR, MDR, NDR, XDR.png]]

## **SIEM (Security Information and Event Management)**

Un SIEM, ou **Security Information and Event Management**, est un système de gestion des informations et des événements de sécurité. Il s'agit d'une solution logicielle qui aide les entreprises à détecter, surveiller et répondre aux menaces de sécurité en temps réel.

Voici les principales fonctionnalités d'un SIEM :

1. **Collecte des données** : Un SIEM agrège et centralise des données provenant de diverses sources au sein d'une organisation, telles que des pare-feux, des systèmes d'exploitation, des applications, des bases de données, des dispositifs réseau, etc. Ces données peuvent inclure des journaux d'événements, des flux réseau, et d'autres types d'informations.

2. **Analyse en temps réel** : Une fois les données collectées, le SIEM analyse ces informations en temps réel pour identifier des activités suspectes ou des anomalies qui pourraient indiquer une menace potentielle.

3. **Corrélation des événements** : Le SIEM utilise des règles de corrélation pour associer différents événements et détecter des modèles ou des comportements anormaux. Par exemple, il pourrait repérer qu'une série d'échecs de connexion suivie d'une tentative de connexion réussie depuis une adresse IP inhabituelle constitue un risque potentiel.

4. **Alertes et rapports** : Lorsqu'une menace potentielle est identifiée, le SIEM génère des alertes pour que l'équipe de sécurité puisse prendre des mesures rapidement. Il fournit également des rapports et des tableaux de bord pour aider à la gestion continue de la sécurité.

5. **Conformité** : Un SIEM peut aider les entreprises à se conformer à diverses réglementations en matière de sécurité en conservant les journaux et en produisant des rapports détaillés sur les événements de sécurité.


En résumé, un SIEM joue un rôle clé dans la sécurité informatique en offrant une visibilité complète sur les activités réseau et en permettant une réponse rapide aux incidents de sécurité.

## **GPO**

**GPO** (Group Policy Object) est un ensemble de règles utilisées dans les environnements Microsoft Windows pour centraliser la gestion et la configuration des systèmes, des utilisateurs et des réseaux. Les GPOs permettent aux administrateurs réseau de définir des paramètres de sécurité, des configurations logicielles, et d’autres aspects du système d’exploitation à appliquer à des groupes d’utilisateurs ou d'ordinateurs au sein d’un **Active Directory (AD)**.

**Fonctionnement des GPO :**

1. **Centralisation des configurations** : Les GPOs sont configurées sur un serveur dans un domaine Active Directory et sont appliquées automatiquement à tous les ordinateurs ou utilisateurs qui appartiennent à ce domaine. Cela permet d’éviter les configurations manuelles individuelles.
    
2. **Appliquer des politiques spécifiques** : Les GPOs peuvent gérer de nombreux paramètres, comme :
    
    - **Politiques de sécurité** (ex. : mots de passe complexes, verrouillage automatique d’écran).
    - **Installation de logiciels** : Déploiement ou restrictions de logiciels.
    - **Restrictions d'accès** : Limitation des actions des utilisateurs (ex. : interdiction d'accéder au panneau de configuration ou à certains sites).
    - **Scripts d'ouverture ou de fermeture de session** : Automatisation de certaines tâches lors de la connexion/déconnexion des utilisateurs.
3. **Hiérarchie et héritage** : Les GPOs peuvent être appliquées à différents niveaux d’une organisation : **sites**, **domaines**, ou **unités organisationnelles (OU)**. Si plusieurs GPOs sont appliquées, des règles d’héritage et de priorité déterminent quelles politiques priment sur les autres.
    

**Exemples d'utilisation :**

- **Sécurité** : Par exemple, définir des GPOs pour forcer l’utilisation de mots de passe complexes, limiter le nombre de tentatives de connexion, ou empêcher l’utilisation de périphériques USB.
- **Déploiement logiciel** : Utiliser les GPOs pour installer automatiquement certains logiciels sur tous les postes d’une entreprise ou bloquer l’installation de logiciels non autorisés.
- **Personnalisation des environnements utilisateurs** : Configurer des bureaux standardisés ou des imprimantes réseau automatiquement mappées pour chaque utilisateur d’un département spécifique.

**Importance des GPOs dans la cybersécurité :**

Les GPOs jouent un rôle critique dans la gestion de la **cybersécurité** au sein des organisations, car elles permettent de :

- **Appliquer des politiques de sécurité cohérentes** à travers tout un réseau.
- **Réduire les erreurs humaines** en automatisant la configuration des systèmes.
- **Contrôler les accès et les permissions** des utilisateurs et des ordinateurs, minimisant ainsi les risques d'abus ou de cyberattaques internes.

En résumé, les GPOs sont un outil puissant pour gérer les environnements Windows en entreprise, offrant une centralisation et une automatisation des configurations de sécurité et d’administration.
## **Palo Alto**

**Palo Alto Networks est une entreprise américaine spécialisée dans la cybersécurité.** Fondée en 2005, elle est devenue l'un des leaders mondiaux dans ce domaine.

**L'entreprise propose une large gamme de solutions de sécurité pour les entreprises et les particuliers, notamment :**

- **Pare-feu nouvelle génération (NGFW):** Le produit phare de Palo Alto Networks, le NGFW est un pare-feu qui offre une protection contre les menaces connues et inconnues.
- **Systèmes de détection d'intrusion (IDS):** Les IDS de Palo Alto Networks permettent de détecter les activités suspectes sur le réseau.
- **Logiciels de prévention des menaces (ATP):** Les ATP de Palo Alto Networks bloquent les attaques avant qu'elles ne touchent le réseau.
- **Solutions de sécurité cloud:** Palo Alto Networks propose également des solutions de sécurité pour les environnements cloud.
- **Solutions de sécurité pour les terminaux:** Palo Alto Networks propose des solutions de sécurité pour protéger les ordinateurs portables, les tablettes et les smartphones.

**Palo Alto Networks est connue pour ses produits innovants et son expertise en matière de sécurité informatique.** L'entreprise a reçu de nombreuses récompenses pour ses solutions, et elle est considérée comme l'un des leaders du marché de la cybersécurité.

**Voici quelques-uns des avantages d'utiliser les solutions de sécurité Palo Alto Networks :**

- **Protection contre les menaces connues et inconnues:** Les solutions de Palo Alto Networks sont constamment mises à jour pour protéger contre les dernières menaces.
- **Facilité d'utilisation:** Les solutions de Palo Alto Networks sont faciles à installer et à utiliser.
- **Performances élevées:** Les solutions de Palo Alto Networks offrent des performances élevées sans sacrifier la sécurité.
- **Évolutivité:** Les solutions de Palo Alto Networks peuvent être adaptées aux besoins des entreprises de toutes tailles.



## **Fortinet FortiGate**

Fortinet FortiGate est une gamme de pare-feu nouvelle génération (NGFW) développée par Fortinet, une multinationale américaine spécialisée dans la cybersécurité. Considéré comme un leader du marché des NGFW, FortiGate est utilisé par les entreprises et les organisations de toutes tailles pour protéger leurs réseaux contre les menaces informatiques modernes.

**Fonctionnalités principales de FortiGate :**

- **Filtrage de paquets avancé:** FortiGate analyse et filtre le trafic réseau entrant et sortant en fonction de règles de sécurité configurables.
- **Inspection approfondie des paquets (DPI) :** FortiGate inspecte le contenu des paquets pour identifier les menaces cachées, telles que les logiciels malveillants et les intrusions.
- **Prévention des intrusions (IPS) :** FortiGate détecte et bloque les activités suspectes sur le réseau.
- **Anti-virus et anti-malware :** FortiGate analyse le trafic réseau et les fichiers pour identifier et bloquer les logiciels malveillants.
- **Filtrage web :** FortiGate peut être utilisé pour bloquer l'accès aux sites web malveillants ou indésirables.
- **VPN :** FortiGate prend en charge les connexions VPN pour un accès sécurisé aux réseaux distants.
- **Haute disponibilité (HA) :** FortiGate peut être configuré en mode haute disponibilité pour garantir la continuité des activités en cas de panne matérielle.

**Avantages d'utiliser Fortinet FortiGate :**

- **Sécurité multicouche :** FortiGate offre une protection contre une large gamme de menaces informatiques.
- **Performances élevées :** FortiGate peut gérer des volumes élevés de trafic réseau sans sacrifier la sécurité.
- **Facilité d'utilisation :** FortiGate est facile à installer et à gérer, même pour les administrateurs réseau novices.
- **Évolutivité :** FortiGate peut être adapté aux besoins des entreprises de toutes tailles.
- **Gestion centralisée :** Les administrateurs peuvent gérer plusieurs appareils FortiGate à partir d'une console centrale.

**Modèles de FortiGate :**

Fortinet propose une large gamme de modèles FortiGate pour répondre aux besoins de différents environnements réseau. Les modèles vont des versions d'entrée de gamme pour les petites entreprises aux modèles haut de gamme pour les grands centres de données

## **Check Point**

**Check Point Software Technologies** est un leader mondial en matière de cybersécurité, fournissant des solutions de sécurité réseau intégrées et unifiées pour les entreprises de toutes tailles.

**Check Point offre une large gamme de produits et services de sécurité, notamment :**

- **Pare-feu nouvelle génération (NGFW):** Le produit phare de Check Point, le NGFW offre une protection contre les menaces connues et inconnues, ainsi qu'une prévention des intrusions et un filtrage web.
- **Systèmes de détection d'intrusion (IDS):** Les IDS de Check Point détectent les activités suspectes sur le réseau et peuvent bloquer les attaques avant qu'elles ne causent des dommages.
- **Logiciels de prévention des menaces (ATP):** Les ATP de Check Point bloquent les attaques avant qu'elles ne touchent le réseau, y compris les ransomwares et les attaques zero-day.
- **Solutions de sécurité cloud:** Check Point propose des solutions de sécurité pour protéger les environnements cloud contre les menaces.
- **Solutions de sécurité pour les terminaux:** Check Point propose des solutions de sécurité pour protéger les ordinateurs portables, les tablettes et les smartphones contre les menaces.
- **Gestion des menaces unifiée (UTM):** Check Point propose une solution UTM qui combine plusieurs fonctions de sécurité dans un seul appareil.

**Check Point est connu pour ses produits innovants et son expertise en matière de sécurité informatique.** L'entreprise a reçu de nombreuses récompenses pour ses solutions, et elle est considérée comme l'un des leaders du marché de la cybersécurité.

**Voici quelques-uns des avantages d'utiliser les solutions de sécurité Check Point :**

- **Protection contre les menaces connues et inconnues:** Les solutions de Check Point sont constamment mises à jour pour protéger contre les dernières menaces.
- **Facilité d'utilisation:** Les solutions de Check Point sont faciles à installer et à utiliser.
- **Performances élevées:** Les solutions de Check Point offrent des performances élevées sans sacrifier la sécurité.
- **Évolutivité:** Les solutions de Check Point peuvent être adaptées aux besoins des entreprises de toutes tailles.
- **Gestion centralisée:** Les administrateurs peuvent gérer plusieurs solutions Check Point à partir d'une console centrale.

## **Cryptographie**

**La cryptographie est la science de protéger les informations en les rendant inintelligibles aux personnes non autorisées.** Elle utilise des techniques mathématiques pour transformer des données lisibles (texte clair) en un format crypté (texte chiffré) qui ne peut être déchiffré que par les personnes possédant la clé appropriée.

**Voici quelques-uns des objectifs de la cryptographie :**

- **Confidentialité:** Assurer que seuls les destinataires prévus peuvent lire les informations.
- **Intégrité:** Garantir que les informations n'ont pas été modifiées depuis leur création.
- **Authenticité:** Vérifier l'identité de l'expéditeur d'un message.
- **Non-répudiation:** Empêcher l'expéditeur d'un message de nier l'avoir envoyé.

**La cryptographie est utilisée dans une grande variété d'applications, notamment :**

- **Communications sécurisées:** Les communications électroniques, telles que les e-mails et les appels téléphoniques, peuvent être cryptées pour les protéger des écoutes indiscrètes.
- **Sécurité des données:** Les données sensibles, telles que les informations financières et médicales, peuvent être cryptées pour les protéger contre le vol et l'accès non autorisé.
- **Authentification des utilisateurs:** Les mots de passe et autres informations d'authentification peuvent être cryptés pour les protéger contre les attaques par force brute.
- **Signature numérique:** Les documents électroniques peuvent être signés numériquement pour garantir leur authenticité et leur intégrité.

**Il existe deux types principaux de cryptage :**

- **Cryptage symétrique:** Le même clé est utilisée pour crypter et décrypter les données.
- **Cryptage asymétrique:** Deux clés différentes sont utilisées pour crypter et décrypter les données. La clé publique est utilisée pour crypter les données, et la clé privée est utilisée pour les décrypter.

**La cryptographie est un outil puissant qui peut être utilisé pour protéger les informations sensibles.** Il est important de choisir le type de cryptage approprié à vos besoins et de mettre en place des mesures de sécurité adéquates pour protéger vos clés.

**Voici quelques ressources utiles pour en savoir plus sur la cryptographie :**

- **Coursera - Introduction à la cryptographie:** [https://www.coursera.org/learn/cryptography](https://www.coursera.org/learn/cryptography)
- Youtube - Crypto, chiffrement, hachage https://www.youtube.com/watch?v=NJT4g5gu50c&t=0s

## **HMAC**

**HMAC (Keyed-Hash Message Authentication Code)** is a type of message authentication code (MAC) that uses a cryptographic hash function in combination with a secret key to verify the authenticity and integrity of data.

An HMAC can be used to ensure that the person who created the HMAC is who they say they are, i.e., authenticity is confirmed; moreover, it proves that the message hasn’t been modified or corrupted, i.e., integrity is maintained. This is achieved through the use of a secret key to prove authenticity and a hashing algorithm to produce a hash and prove integrity.

The following steps give you a fair idea of how HMAC works.

1. The secret key is padded to the block size of the hash function.
2. The padded key is XORed with a constant (usually a block of zeros or ones).
3. The message is hashed using the hash function with the XORed key.
4. The result from Step 3 is then hashed again with the same hash function but using the padded key XORed with another constant.
5. The final output is the HMAC value, typically a fixed-size string.

The illustration below should clarify the above steps.

![[HMAC.png]]

Technically speaking, the HMAC function is calculated using the following expression:

_H__M__A__C_(_K_,_M_) = _H_((_K_⊕_o__p__a__d_)||_H_((_K_⊕_i__p__a__d_)||_M_))

Note that M and K represent the message and the key, respectively.
## **Base de données**

**Une base de données est une collection organisée d'informations** stockées sur un ordinateur. Elle permet de stocker et de retrouver facilement des données structurées, semi-structurées ou non structurées.

**Voici quelques-uns des avantages d'utiliser une base de données :**

- **Efficacité:** Les bases de données permettent de stocker et de retrouver des informations rapidement et facilement.
- **Fiabilité:** Les bases de données protègent les données contre la corruption et la perte.
- **Partage:** Les bases de données permettent de partager des informations avec plusieurs utilisateurs.
- **Sécurité:** Les bases de données permettent de contrôler l'accès aux informations.

**Il existe plusieurs types de bases de données, dont les plus courants sont :**

- **Bases de données relationnelles:** Elles stockent les données dans des tables, qui sont liées entre elles par des clés.
- **Bases de données NoSQL:** Elles stockent les données dans des formats non structurés, ce qui les rend plus flexibles que les bases de données relationnelles.
- **Bases de données en nuage:** Elles sont hébergées sur des serveurs distants, ce qui permet d'y accéder depuis n'importe quel endroit.

## **SQL**

**SQL (Structured Query Language)** est un langage de programmation standardisé utilisé pour interroger et modifier des données dans les bases de données relationnelles.

**Voici quelques-unes des fonctionnalités de SQL :**

- **Sélection de données:** SQL permet de sélectionner des données spécifiques dans une ou plusieurs tables.
- **Ajout de données:** SQL permet d'ajouter de nouvelles données à une table.
- **Modification de données:** SQL permet de modifier les données existantes dans une table.
- **Suppression de données:** SQL permet de supprimer des données d'une table.
- **Tri et filtrage des données:** SQL permet de trier et de filtrer les données en fonction de critères spécifiques.
- **Création de jointures:** SQL permet de combiner des données de plusieurs tables.
- **Création de vues:** SQL permet de créer des vues virtuelles des données.

**SQL est un langage relativement simple à apprendre et à utiliser.** Il existe de nombreuses ressources disponibles en ligne et dans les bibliothèques pour vous aider à apprendre SQL.

**Voici quelques exemples d'utilisation de SQL :**

- **Un développeur web peut utiliser SQL pour interroger une base de données d'utilisateurs afin d'afficher les informations d'un utilisateur sur une page web.**
- **Un analyste marketing peut utiliser SQL pour interroger une base de données de vente afin d'identifier les tendances du marché.**
- **Un administrateur système peut utiliser SQL pour créer des scripts pour automatiser des tâches de gestion de base de données.**

**SQL est un outil puissant et polyvalent qui peut être utilisé par une grande variété de personnes pour travailler avec des bases de données.**



## **MySQL**

**MySQL est un système de gestion de bases de données relationnelles (SGBDR) open source** largement utilisé pour stocker et gérer des données. Il est développé et soutenu par Oracle Corporation.

**MySQL est connu pour sa:**

- **Facilité d'utilisation:** Il est facile à installer, à configurer et à utiliser, même pour les débutants.
- **Flexibilité:** Il est compatible avec une large gamme de langages de programmation et de systèmes d'exploitation.
- **Performances élevées:** Il peut gérer de grandes quantités de données avec des performances élevées.
- **Évolutivité:** Il peut être facilement mis à l'échelle pour répondre aux besoins croissants de votre entreprise.
- **Coût:** La version communautaire de MySQL est gratuite et open source, ce qui la rend accessible à tous.

**MySQL est utilisé par une grande variété d'entreprises et d'organisations, notamment :**

- **Sites web:** WordPress, Drupal, Joomla et d'autres plateformes de contenu populaires utilisent MySQL pour stocker leurs données.
- **Applications web:** De nombreuses applications web, telles que les boutiques en ligne et les réseaux sociaux, utilisent MySQL pour stocker leurs données.
- **Entreprises:** De nombreuses entreprises, grandes et petites, utilisent MySQL pour stocker leurs données clients, produits et financières.
- **Organisations à but non lucratif:** De nombreuses organisations à but non lucratif utilisent MySQL pour stocker leurs données de membres, de dons et de programmes.
## **DVWA**

## **Flux RSS**

Un **flux RSS** (Really Simple Syndication) est un format de données utilisé pour diffuser fréquemment des mises à jour de contenu sur des sites web, des blogs ou des journaux en ligne. Il permet aux utilisateurs de recevoir automatiquement les dernières actualités ou publications sans avoir à visiter les sites web eux-mêmes. Un flux RSS présente souvent un titre, un résumé et un lien vers le contenu complet.

**Fonctionnement d’un flux RSS**

1. **Flux généré par un site** : Un site web génère un fichier XML contenant les informations à jour (nouveaux articles, mises à jour).
2. **Lecteur de flux RSS** : L’utilisateur s’abonne à ce flux via un lecteur RSS. Ce dernier vérifie régulièrement si de nouveaux éléments sont publiés.
3. **Notification de mises à jour** : Lorsque le site publie de nouveaux contenus, l'utilisateur reçoit une notification via son lecteur RSS avec un résumé et un lien vers le contenu.

**Utilisation des flux RSS en cybersécurité**

Dans le domaine de la cybersécurité, les flux RSS peuvent être un outil précieux pour rester informé des dernières menaces, vulnérabilités, correctifs, et actualités en matière de sécurité informatique.

**Utilisations concrètes dans la cybersécurité :**

1. **Surveillance des vulnérabilités** :
    
    - Les professionnels de la sécurité peuvent s’abonner aux flux RSS de bases de données de vulnérabilités telles que le **CVE (Common Vulnerabilities and Exposures)**. Ils reçoivent ainsi des alertes sur les nouvelles vulnérabilités découvertes dans les logiciels et systèmes.
2. **Mises à jour des correctifs** :
    
    - Des éditeurs de logiciels comme **Microsoft**, **Adobe** ou **Cisco** proposent des flux RSS pour notifier les nouvelles mises à jour de sécurité et correctifs publiés. Cela permet aux administrateurs réseau de rapidement déployer les correctifs.
3. **Alerte sur les menaces en temps réel** :
    
    - Les entreprises spécialisées dans la sécurité, comme **CERT** ou **Kaspersky**, publient souvent des alertes de menaces via RSS, informant des nouvelles cyberattaques, malwares ou ransomwares en circulation.
4. **Surveillance des blogs et recherches en sécurité** :
    
    - Les experts en cybersécurité tiennent souvent des blogs où ils partagent des recherches, des analyses ou des études de cas. S’abonner à ces flux permet de rester à jour sur les avancées dans le domaine.
5. **Conformité et réglementations** :
    
    - Certaines agences gouvernementales et organisations publient des changements dans les lois et règlements relatifs à la cybersécurité via des flux RSS. Cela aide les entreprises à rester en conformité avec les nouvelles normes de sécurité.

**Outils et plateformes pour utiliser les flux RSS dans la cybersécurité :**

- **Lecteurs RSS spécialisés** : Vous pouvez utiliser des outils comme **Feedly** ou **Inoreader** pour centraliser les flux de plusieurs sources.
- **Automatisation avec des scripts** : Certains professionnels de la sécurité utilisent des scripts pour analyser les flux RSS et automatiser des actions (par exemple, déclencher une alerte interne lorsqu’une vulnérabilité critique est détectée).
- **SIEM** : Certains systèmes d'information et d'événements de sécurité (SIEM) peuvent intégrer des flux RSS pour enrichir les données avec des alertes de vulnérabilités ou des menaces externes.

En résumé, le flux RSS est un outil simple mais efficace pour automatiser la surveillance des informations critiques en matière de cybersécurité et permettre une réaction rapide face aux menaces émergentes.
## **Metasploitable 2/3**

## **Punycode Attack**

**Punycode** : Unicode that converts words that cannot be written in ASCII, like the Greek word for thank you ‘ευχαριστώ’ into an ASCII encoding, like ‘xn--mxahn5algcq2e’ for use as domain names.

**How Punycode Attack work**

Unicode characters can look the same to the naked eye but actually, have a different web address. Some letters in the Roman alphabet, used by the majority of modern languages, are the same shape as letters in Greek, Cyrillic, and other alphabets, so it’s easy for an attacker to launch a domain name that replaces some ASCII characters with Unicode characters. For example, you could swap a normal T for a Greek Tau: τ, the user would see the almost identical T symbol but the punycode behind this, read by the computer, is actually xn--5xa. Depending on how the browser renders this information in the address bar, these sneaky little characters are impossible for us humans to identify.  
This technique is called a **homograph attack**, the URLs will look legitimate, and the content on the page might appear the same on the face of it but its actually a different website set up to steal the victim’s sensitive data or to infect the user’s device. These attacks use common techniques like phishing, forced downloads, and scams.

## **Cyber Kill Chain**

![[Cyber Kill Chain.png]]

## **Unified Kill Chain (UKC)**

![[Unified Kill Chain (UKC).png]]

## **YARA**

**YARA (Yet Another Recursive Acronym)** est un outil utilisé principalement pour la détection et l'analyse de logiciels malveillants (malware). Il permet aux chercheurs en sécurité de créer des règles basées sur des chaînes de caractères, des expressions régulières et des métadonnées, afin d'identifier des fichiers ou processus suspects en fonction de certains modèles ou comportements spécifiques.

**Eléments clés de YARA :

- **Règles YARA** : Elles sont écrites dans un langage spécifique à YARA et définissent des motifs ou des signatures qui correspondent à des caractéristiques observées dans des fichiers ou des programmes malveillants. Chaque règle est composée de plusieurs sections, notamment un nom, des métadonnées optionnelles, des chaînes de caractères (ou motifs) à rechercher, et une condition qui spécifie quand la règle doit se déclencher.

- **Conditions** : Elles peuvent inclure des opérations logiques (comme AND, OR, NOT), des comparaisons numériques ou de chaîne, et d'autres expressions complexes. Cela permet une grande flexibilité dans la détection.

- **Chaînes de caractères** : Ce sont les éléments principaux que YARA recherche dans les fichiers ou la mémoire. Les chaînes peuvent être simples (texte brut) ou des expressions régulières.

**Utilisation de YARA :

- **Détection de Malware** : YARA est couramment utilisé pour créer des signatures permettant de détecter des familles de malwares spécifiques. Ces signatures sont ensuite utilisées pour scanner des fichiers ou des systèmes afin de détecter la présence de ces malwares.

- **Analyse Forensique** : Lors d'une analyse post-mortem d'un incident de sécurité, YARA peut être utilisé pour rechercher des traces d'activités malveillantes dans les fichiers, la mémoire ou les processus d'un système.

- **Automatisation** : De nombreuses solutions de sécurité intègrent YARA pour effectuer des analyses automatiques en temps réel ou périodiques.


**Exemple simple de règle YARA :

`rule ExampleRule {     strings:         $text1 = "malicious string"         $text2 = "another suspicious pattern"              condition:         $text1 or $text2 }`

Cette règle détecte un fichier contenant soit la chaîne "malicious string" soit la chaîne "another suspicious pattern".

YARA est donc un outil puissant pour les experts en cybersécurité, leur permettant d'écrire des signatures personnalisées pour détecter des menaces spécifiques et d'analyser des fichiers suspects de manière efficace.

## **CTF**

## **Bandit**

## **DevOps**

Le **DevOps** est une approche qui vise à unifier le développement (Dev) et les opérations (Ops) d'une organisation. L'objectif principal du DevOps est d'améliorer la collaboration entre les équipes de développement et d'exploitation, d'accélérer la livraison des applications et d'augmenter la qualité des logiciels. Cela implique l'utilisation d'outils et de pratiques qui facilitent l'automatisation, l'intégration continue et le déploiement continu. Voici comment Kubernetes, Ansible, Jenkins et Docker s'intègrent dans cette démarche :

1. **Docker**

**Docker** est un outil de virtualisation légère qui permet de créer, déployer et exécuter des applications dans des conteneurs. Les conteneurs sont des unités standardisées qui regroupent une application et toutes ses dépendances, ce qui garantit qu'elle fonctionne de manière cohérente dans n'importe quel environnement.

- **Rôle dans DevOps** : Docker facilite le développement et le déploiement en rendant les applications portables et en réduisant les problèmes de compatibilité. Les développeurs peuvent créer et tester des environnements de manière locale sur leur machine, puis déployer ces conteneurs dans des environnements de production.

2. **Jenkins**

**Jenkins** est un serveur d'intégration continue (CI) et de livraison continue (CD) qui automatise les processus de construction, de test et de déploiement des applications.

- **Rôle dans DevOps** : Jenkins permet aux équipes de développement d'automatiser les builds et les tests de leurs applications chaque fois qu'un changement est apporté au code source. Cela garantit que les problèmes sont détectés rapidement et que le code est toujours dans un état déployable. Jenkins peut être configuré pour exécuter des conteneurs Docker, ce qui permet de construire des images de conteneurs et de les tester dans le cadre du pipeline CI/CD.

3. **Ansible**

**Ansible** est un outil d'automatisation de la configuration et de gestion des déploiements. Il utilise un langage de configuration simple basé sur YAML pour décrire les tâches d'automatisation.

- **Rôle dans DevOps** : Ansible est utilisé pour automatiser le déploiement et la configuration des environnements d'application, qu'ils soient physiques ou virtuels. Cela inclut l'installation de logiciels, la configuration des serveurs et la gestion des mises à jour. Dans un contexte DevOps, Ansible peut également être utilisé pour orchestrer le déploiement de conteneurs Docker et pour configurer les ressources nécessaires avant et après le déploiement.

4. **Kubernetes**

**Kubernetes** est un système d'orchestration de conteneurs qui facilite le déploiement, la mise à l'échelle et la gestion d'applications conteneurisées. Il permet de gérer des clusters de serveurs et d'assurer la disponibilité, la scalabilité et la résilience des applications.

- **Rôle dans DevOps** : Kubernetes s'intègre avec Docker pour gérer les conteneurs à grande échelle. Lorsqu'une application est développée avec Docker et testée avec Jenkins, Kubernetes prend en charge le déploiement des conteneurs dans des clusters, les équilibrant et les surveillant en continu. Cela simplifie la gestion des applications microservices, améliore la résilience et facilite les mises à jour sans temps d'arrêt.

**Flux de travail DevOps utilisant ces outils :**

1. **Développement avec Docker** :
    
    - Les développeurs créent des conteneurs pour leurs applications avec Docker, garantissant que toutes les dépendances sont incluses.
2. **Intégration continue avec Jenkins** :
    
    - Lorsqu'un développeur soumet des modifications de code (via un système de contrôle de version comme Git), Jenkins déclenche automatiquement un pipeline CI qui construit l'image Docker, exécute des tests, et produit une image prête à être déployée.
3. **Automatisation de la configuration avec Ansible** :
    
    - Ansible est utilisé pour configurer les environnements, installer des dépendances nécessaires et préparer les serveurs pour le déploiement des conteneurs. Cela peut inclure la configuration de réseaux, la sécurité et les bases de données.
4. **Orchestration avec Kubernetes** :
    
    - Kubernetes déploie et gère les conteneurs créés par Docker, en s'assurant qu'ils sont en ligne, évolutifs et résilients face aux pannes. Il fournit également des fonctionnalités comme la mise à l'échelle automatique et la gestion des mises à jour sans interruption du service.

**Conclusion**

Le DevOps, en intégrant Docker, Jenkins, Ansible et Kubernetes, crée un écosystème où le développement et les opérations peuvent travailler ensemble de manière fluide. Cela permet d'accélérer la livraison des logiciels tout en maintenant la qualité et la stabilité des applications. Les entreprises adoptant cette approche bénéficient d'une meilleure agilité, d'une réduction des délais de mise sur le marché et d'une plus grande satisfaction des clients.
## **DevSecOps**


## **DORA**

The Digital Operational Resilience Act (DORA) is a legislative framework designed to enhance the digital operational resilience of financial institutions across the European Union. By introducing standardized processes for managing, reporting, and reacting to ICT operational risks, DORA aims to foster a resilient financial ecosystem capable of withstanding cyberattacks, technological failures, and human errors.

## **Analyse d’Impact relative à la Protection des Données (AIPD)**

démarche visant à identifier et atténuer les risques liés au traitement de données personnelles. Elle est obligatoire lorsqu’un traitement est susceptible d'engendrer des risques élevés pour les droits et libertés des individus, par exemple lors de l’utilisation de nouvelles technologies.

## **RGPD**

**Astuce pour le RGPD :** Pensez à **3 P : Protéger, Prévenir, Permettre** :

- Protéger les données.
- Prévenir les violations.
- Permettre aux individus de garder le contrôle.

## **CNIL**

(Commission Nationale de l'Informatique et des Libertés)
## **Outils d'attaque**

- MITRE Att&CK : 
- Metasploit : 
- Openvas : 
- WPScan :
- Hydra :
- Nmap :
- Zenmap :
- Shodan :
- Wireshark :
- Aircrack-ng :
- Reaver :
- OWASP ZAP :
- BurpSuite :
- John the Ripper :
- Hashcat :

Pentester LAB :

Hack The Box :

TryHackMe :

PicoCTF :
