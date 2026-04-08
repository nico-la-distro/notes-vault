La **Pyramid of Pain** sert à classer les indicateurs et comportements malveillants selon **la difficulté qu’aura l’attaquant à les changer**. Plus on détecte **haut dans la pyramide**, plus on lui fait mal : il ne suffit plus de changer un hash, une IP ou un domaine, il doit revoir ses outils, ses méthodes, voire abandonner la cible. La room relie ce modèle à la **CTI**, au **threat hunting** et à l’**incident response**.

![[Detection Engineering pyramide of pain.png]]
## La pyramide en une table

|Niveau|Ce que c’est|Pourquoi c’est utile|Douleur pour l’attaquant|
|---|---|---|---|
|**Hash values**|Empreintes de fichiers (MD5, SHA-1, SHA-256)|Identifier rapidement un malware/fichier connu|**Très faible** : modifier 1 bit change le hash|
|**IP addresses**|Adresses réseau utilisées par l’attaquant|Blocage firewall, corrélation réseau|**Faible** : changer d’IP est simple, surtout avec du **Fast Flux**|
|**Domain names**|Noms de domaine et sous-domaines|Détection via DNS/proxy/web logs|**Faible à modérée** : plus coûteux qu’une IP, mais reste remplaçable|
|**Host artifacts**|Traces laissées sur l’hôte : registre, fichiers, processus, événements|Très utile pour repérer l’exécution réelle du malware|**Modérée** : l’attaquant doit adapter ses méthodes/outils|
|**Network artifacts**|User-Agent, URI, motifs HTTP POST, C2, etc.|Détection dans PCAP, IDS, logs réseau|**Modérée** : changements possibles, mais plus coûteux|
|**Tools**|Malware, loaders, maldocs, backdoors, DLL/EXE, crackers|Signatures AV, règles Sigma/Snort, YARA, similarité|**Élevée** : recréer/remplacer l’outil demande temps/argent/compétences|
|**TTPs**|Tactiques, techniques et procédures (MITRE ATT&CK)|Détecter le mode opératoire global|**Très élevée** : l’adversaire doit revoir sa manière d’opérer|

## Résumé par section

### 1. Hash values

Les hashes servent à **identifier et référencer** des fichiers suspects ou malveillants. La room rappelle que **MD5** et **SHA-1** ne sont plus considérés comme sûrs, tandis que **SHA-256** est un standard courant. En pratique, un hash est utile pour faire des recherches sur des plateformes comme **VirusTotal** ou **MetaDefender**. Le gros défaut : **un changement minime dans le fichier produit un hash totalement différent**, donc c’est un IOC très fragile.

**À retenir :**  
Détection rapide, mais **très facile à contourner**.

### 2. IP addresses

Une IP malveillante peut être bloquée ou surveillée, ce qui reste utile en défense. Mais c’est aussi un indicateur **peu robuste**, car un attaquant peut rapidement basculer vers une nouvelle IP publique. La room introduit ici le **Fast Flux** : une technique DNS où un domaine pointe vers **plusieurs IP qui changent fréquemment**, afin de masquer l’infrastructure réelle et compliquer la détection.

**À retenir :**  
Utile pour bloquer vite, mais **facile à remplacer**.

### 3. Domain names

Les domaines sont un peu plus “coûteux” à changer que les IP : il faut les enregistrer, gérer le DNS, etc. La room insiste sur deux points :

- **Punycode / homoglyphes** : un domaine peut sembler légitime visuellement en utilisant des caractères Unicode proches des vrais caractères. Exemple montré avec une fausse variation d’**adidas.de**.
    
- **URL shorteners** : les attaquants masquent souvent la destination réelle via des services comme **bit.ly**, **tinyurl**, etc. La room donne une astuce pratique : **ajouter `+` à certaines URL raccourcies** pour prévisualiser la destination.
    

Elle rappelle aussi que dans **Any.run**, on peut observer :

- les **HTTP requests**,
    
- les **connections**,
    
- les **DNS requests**.
    

**À retenir :**  
Un domaine est plus parlant qu’une IP, mais reste **changeable**.

### 4. Host artifacts

Ici, on monte d’un cran. Les **host artifacts** sont les traces laissées localement par l’attaque :  
processus suspects, clés de registre, fichiers déposés, séquences d’événements, patterns d’exécution, etc. La room explique que détecter ce niveau **force souvent l’attaquant à revoir son outil ou sa chaîne d’attaque**, ce qui devient bien plus pénible pour lui.

**À retenir :**  
On n’observe plus juste un IOC isolé, mais **l’impact réel sur la machine**.

### 5. Network artifacts

Les **network artifacts** regroupent les éléments repérables dans le trafic :  
**User-Agent**, infos C2, motifs d’URI, contenus de **HTTP POST**, etc. La room cite l’analyse de **PCAP** avec **Wireshark/TShark** et l’exploitation de logs **IDS** comme **Snort**. Un exemple de commande TShark est donné pour extraire les **User-Agent** des requêtes HTTP.

**À retenir :**  
Très intéressant pour le hunting, surtout quand un malware utilise des comportements réseau atypiques.

### 6. Tools

À ce niveau, on cible **l’outil lui-même** : malware, stealer, dropper, macro malveillante, backdoor, DLL/EXE, etc. La room explique que des **signatures antivirus**, des **règles de détection** et surtout des **règles YARA** sont de très bons moyens de faire mal à l’attaquant. Elle cite aussi **MalwareBazaar**, **Malshare** et **SOC Prime** comme ressources utiles.

Elle introduit enfin le **fuzzy hashing** avec **SSDeep** : au lieu de comparer des fichiers strictement identiques, on mesure leur **similarité** même s’ils ont subi de petites modifications.

**À retenir :**  
Détecter l’outil ou sa famille est **beaucoup plus durable** que détecter juste un hash.

### 7. TTPs

Le sommet de la pyramide concerne les **TTPs** :  
**Tactics, Techniques & Procedures**. La room les relie explicitement à **MITRE ATT&CK**. Ici, on cherche à détecter **la manière d’opérer** de l’attaquant : phishing, persistance, mouvement latéral, exfiltration, etc. L’exemple donné est le **Pass-the-Hash**, qu’on pourrait détecter via la surveillance des journaux Windows. Si on bloque ce niveau, l’adversaire doit **repenser sa méthode**, se former davantage, reconfigurer ses outils ou abandonner.

**À retenir :**  
C’est le niveau **le plus valuable en défense**, car il cible le **comportement** et non un simple artefact.

## Mémo ultra-court

|Bas de pyramide|Haut de pyramide|
|---|---|
|Facile à collecter|Plus difficile à construire/détecter|
|Facile à contourner|Difficile à contourner|
|Hash / IP / Domain|Host artifacts / Network artifacts / Tools / TTPs|
|Peu de douleur|Beaucoup de douleur|

## Ce qu’il faut retenir

- **Tous les IOC ne se valent pas.**
    
- Plus un indicateur est **proche du comportement**, plus il est **durable**.
    
- **Hashes, IP, domaines** sont utiles, mais souvent vite remplacés.
    
- **Host artifacts** et **network artifacts** donnent déjà une détection plus solide.
    
- **Tools** et surtout **TTPs** sont les niveaux qui causent le plus de “pain” à l’adversaire.