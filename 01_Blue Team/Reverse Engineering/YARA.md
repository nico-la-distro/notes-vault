
GitHub : https://github.com/virustotal/yara

## What is YARA ?

**Yara** est un outil de **pattern matching** très utilisé en **malware analysis**.  
Il sert à **identifier des fichiers** à partir de **motifs textuels et binaires**, par exemple :

- des **chaînes de caractères** (_strings_)
    
- des **valeurs hexadécimales**
    
- d’autres **motifs présents dans un fichier**
    

_[Virustotal., 2020](https://virustotal.github.io/yara/))_
### Idée principale

Yara fonctionne avec des **règles** (_rules_) qui permettent de **repérer** et **étiqueter** ces motifs.  
Un usage classique est de **déterminer si un fichier est potentiellement malveillant** selon les caractéristiques qu’il contient.

### Importance des strings

Les **strings** sont essentielles, car les programmes stockent souvent du texte sous cette forme.  
Exemple simple en Python :
```python
print("Hello World!")
```
Ici, `"Hello World!"` est une **string**.  
Donc, une règle Yara peut chercher une string précise dans des fichiers ou programmes.

---
### Pourquoi les malwares utilisent des strings ?

Les malwares stockent eux aussi des données textuelles dans des strings.

| Type de malware | Exemple de donnée stockée       | Utilité                                             | Data                               |
| --------------- | ------------------------------- | --------------------------------------------------- | ---------------------------------- |
| **Ransomware**  | Adresse de portefeuille Bitcoin | Recevoir le paiement de la rançon                   | 12t9YDPgwueZ9NyMgw519p7AA8isjr6SMw |
| **Botnet**      | Adresse IP du serveur C2        | Communiquer avec le serveur de commande et contrôle | 12.34.56.7                         |

---

## YARA Rules (introduction)

Le langage de règles **Yara** est **facile à apprendre**, mais **difficile à maîtriser**.
L’efficacité d’une règle dépend surtout de ta capacité à **identifier les bons patterns** à rechercher.

---

### Utilisation de base

Chaque commande `yara` a besoin de **2 arguments** :

|Argument|Rôle|
|---|---|
|**Fichier de règle**|Le fichier `.yar` contenant la règle|
|**Cible**|Fichier, dossier ou PID à analyser|

#### Syntaxe

yara myrule.yar somedirectory

> `.yar` = extension standard des règles Yara

---

### Structure minimale d’une règle

Chaque règle doit contenir au minimum :

|Élément|Obligatoire|Rôle|
|---|---|---|
|**Nom**|Oui|Identifie la règle|
|**Condition**|Oui|Définit ce qui doit être vrai pour qu’elle corresponde|

#### Exemple minimal

```yara
rule examplerule {  
    condition: true  
}
```

#### Ce que fait cette règle

- `examplerule` = **nom de la règle**
    
- `condition: true` = la condition est **toujours vraie**
    

➡ Donc, si la cible existe et peut être ouverte, Yara retourne le nom de la règle.

---

### Démo pratique

#### Étapes
```bash
touch somefile  
touch myfirstrule.yar  
nano myfirstrule.yar
```

Contenu de `myfirstrule.yar` :
```yara
rule examplerule {  
    condition: true  
}
```

#### Exécution
```bash
yara myfirstrule.yar somefile
```
#### Résultat attendu si le fichier existe

examplerule somefile

#### Si le fichier n’existe pas

error scanning sometextfile: could not open file

---

### À retenir

- Une règle Yara valide doit avoir **un nom** + **une condition**
    
- Une commande Yara prend **un fichier de règle** + **une cible**
    
- `condition: true` est l’exemple **le plus simple possible**
    
- Si la cible existe, Yara affiche **le nom de la règle**
    
- Si la cible n’existe pas, Yara renvoie une **erreur d’ouverture**

---

## Expending on YARA Rules

Une règle Yara qui vérifie seulement l’existence d’un fichier est **peu utile**.  
La vraie puissance de Yara vient de :

- **`meta`** pour documenter
    
- **`strings`** pour définir ce qu’on cherche
    
- **`condition`** pour préciser quand la règle doit matcher
    

_moreso : https://yara.readthedocs.io/en/stable/writingrules.html_

---

### Structure des parties importantes

|Partie|Rôle|Impact sur le match|
|---|---|---|
|**meta**|Infos descriptives sur la règle|**Non**|
|**strings**|Motifs à rechercher (texte / hex)|**Oui**|
|**condition**|Logique qui détermine si la règle correspond|**Oui**|

---

### `meta`

Section utilisée pour **décrire la règle**.

Exemple d’usage :

- auteur
    
- description (`desc`)
    
- commentaire sur ce que la règle détecte
    

⚠️ **`meta` n’influence pas le résultat** de la règle.  
C’est uniquement pour la **lisibilité** et la **maintenance**.

---

### `strings`

Permet de chercher du **texte** ou des **motifs hexadécimaux** dans un fichier / programme.

#### Exemple simple
```yara
rule helloworld_checker {  
    strings:  
        $hello_world = "Hello World!"  
  
    condition:  
        $hello_world  
}
```

#### Signification

- `$hello_world` = variable qui stocke la chaîne recherchée
    
- `condition: $hello_world` = la règle matche si cette string est trouvée
    
### Sensibilité à la casse

Avec une string unique, Yara cherche une **correspondance exacte**.

Donc :

- `"Hello World!"` ✅
    
- `"hello world"` ❌
    
- `"HELLO WORLD"` ❌
    

#### Solution : plusieurs strings + `any of them`
```yara
rule helloworld_checker {  
    strings:  
        $hello_world = "Hello World!"  
        $hello_world_lowercase = "hello world"  
        $hello_world_uppercase = "HELLO WORLD"  
  
    condition:  
        any of them  
}
```
#### Effet

La règle matche si **au moins une** des strings est trouvée.

---

### Conditions utiles

Yara permet d’utiliser des opérateurs logiques/classiques.

|Opérateur / mot-clé|Signification|
|---|---|
|`<=`|inférieur ou égal|
|`>=`|supérieur ou égal|
|`!=`|différent de|
|`and`|ET|
|`or`|OU|
|`not`|NON|

---

### Compter les occurrences d’une string

Exemple :
```yara
rule helloworld_checker {  
    strings:  
        $hello_world = "Hello World!"  
  
    condition:  
        #hello_world <= 10  
}
```

#### Signification

- `#hello_world` = **nombre d’occurrences** de la string
    
- la règle matche seulement si `"Hello World!"` apparaît **10 fois ou moins**
    

---

### Combiner plusieurs conditions

Exemple :
```yara
rule helloworld_checker {  
    strings:  
        $hello_world = "Hello World!"  
  
    condition:  
        $hello_world and filesize < 10KB  
}
```

#### Signification

La règle matche seulement si :

1. la string `"Hello World!"` est présente
    
2. le fichier fait **moins de 10 KB**
    

➡ **Les deux conditions doivent être vraies**

---

### Anatomy of YARA Rules

![[YARA Anatomy of YARA Rules.png]]

Information security researcher "fr0gger_" has recently created a [handy cheatsheet](https://medium.com/malware-buddy/security-infographics-9c4d3bd891ef#18dd) that breaks down and visualises the elements of a YARA rule (shown above, all image credits go to him). It's a great reference point for getting started!

---
### À retenir

- **`meta`** = documentation, pas de rôle dans la détection
    
- **`strings`** = motifs recherchés
    
- **`condition`** = logique de match
    
- Une string seule est **sensible à la casse**
    
- `any of them` permet de matcher **plusieurs strings**
    
- `#nom_string` permet de **compter les occurrences**
    
- On peut combiner des critères avec **`and` / `or` / `not`**
    
- `filesize` permet d’ajouter un filtre sur la **taille du fichier**
    

---

## YARA Modules

Les **modules Yara** permettent d’enrichir les règles en s’appuyant sur des **outils externes** ou des **bibliothèques spécialisées**.  
➡ Ils rendent les règles **beaucoup plus précises et techniques**.

---

### Exemples de modules / intégrations

|Outil / module|Rôle|Intérêt pour Yara|
|---|---|---|
|**Cuckoo Sandbox**|Analyse automatisée de malware en exécution|Permet de créer des règles basées sur le **comportement** observé|
|**Python PE module**|Analyse de la structure des exécutables Windows|Permet de créer des règles à partir des **sections** et éléments d’un fichier PE|

---

### Cuckoo Sandbox

**Cuckoo Sandbox** est un environnement d’**analyse automatique de malware**.

https://github.com/cuckoosandbox/cuckoo

#### Ce que ça apporte à Yara

On peut générer des règles à partir de ce que le malware fait pendant son exécution, par exemple :

- **runtime strings**
    
- comportements observés
    
- artefacts laissés pendant l’exécution
    

➡ Utile pour détecter un malware selon son **comportement**, pas seulement selon son contenu statique.

---

### Python PE module

Le module **PE** de Python permet de travailler sur la structure des fichiers **PE (Portable Executable)**.

https://pypi.org/project/pefile/

#### Rappel

Le format **PE** est le format standard des :

- **exécutables Windows**
    
- **DLL**
    
- certaines bibliothèques utilisées par les programmes
    

#### Ce que ça apporte à Yara

Permet de créer des règles basées sur :

- les **sections** du binaire
    
- les **imports**
    
- la structure interne du fichier
    
- d’autres éléments caractéristiques d’un exécutable Windows
    

---

### Pourquoi c’est utile en malware analysis

L’analyse d’un fichier PE est une technique essentielle, car elle peut révéler des comportements comme :

- **cryptographie**
    
- **propagation / worming**
    
- usage de bibliothèques spécifiques
    

➡ Et cela **sans** forcément :

- exécuter le malware
    
- faire du reverse engineering complet
    

---

### À retenir

- Les modules Yara servent à **augmenter la précision** des règles
    
- **Cuckoo** aide pour des règles basées sur le **comportement dynamique**
    
- **Python PE** aide pour des règles basées sur la **structure des exécutables Windows**
    
- L’analyse PE permet souvent d’identifier des capacités d’un malware **sans exécution**

---

## Other tools and YARA

Créer ses propres règles Yara est utile, mais en pratique on peut aussi s’appuyer sur des **outils open source** et des **repos GitHub** déjà existants pour :

- la **threat hunting**
    
- l’**incident response**
    
- la recherche d’**IOC** (_Indicators of Compromise_)
    

_Github : https://github.com/InQuest/awesome-yara_

---

### Outils à connaître

|Outil|Rôle principal|Points clés|
|---|---|---|
|**LOKI**|Scanner IOC|Gratuit, open source, fonctionne sur **Windows** et **Linux**|
|**THOR Lite**|Scanner IOC + YARA|Multi-plateforme, version gratuite de THOR, pensé pour environnements pro|
|**FENRIR**|Bash IOC checker|Script bash, portable sur tout système capable d’exécuter bash|
|**YAYA**|Gestionnaire de règles Yara|Permet d’importer, gérer, modifier et lancer plusieurs rulesets|

---

### LOKI

**LOKI** est un scanner d’**IOC** open source créé par **Florian Roth**.

#### Méthodes de détection principales

|Méthode|But|
|---|---|
|**File Name IOC Check**|Détection via noms de fichiers suspects|
|**Yara Rule Check**|Détection via règles Yara|
|**Hash Check**|Détection via empreintes de fichiers|
|**C2 Back Connect Check**|Détection de connexions vers un serveur C2|

#### À retenir

- gratuit et open source
    
- disponible sur **Windows** et **Linux**
    
- combine **plusieurs techniques de détection**, pas seulement Yara
    

_Github : https://github.com/Neo23x0/Loki/blob/master/README.md_

_Download : https://github.com/Neo23x0/Loki/releases_

```bash
cmnatic@thm:~/Loki$ python3 loki.py -h
usage: loki.py [-h] [-p path] [-s kilobyte] [-l log-file] [-r remote-loghost]
               [-t remote-syslog-port] [-a alert-level] [-w warning-level]
               [-n notice-level] [--allhds] [--alldrives] [--printall]
               [--allreasons] [--noprocscan] [--nofilescan] [--vulnchecks]
               [--nolevcheck] [--scriptanalysis] [--rootkit] [--noindicator]
               [--dontwait] [--intense] [--csv] [--onlyrelevant] [--nolog]
               [--update] [--debug] [--maxworkingset MAXWORKINGSET]
               [--syslogtcp] [--logfolder log-folder] [--nopesieve]
               [--pesieveshellc] [--python PYTHON] [--nolisten]
               [--excludeprocess EXCLUDEPROCESS] [--force]

Loki - Simple IOC Scanner

optional arguments:
  -h, --help            show this help message and exit
```

---

### THOR Lite

**THOR Lite** est aussi développé par **Florian Roth / Nextron**.

#### Particularités

- scanner **IOC + YARA**
    
- **multi-plateforme** : Windows, Linux, macOS
    
- possède une fonction de **scan throttling** pour limiter l’usage CPU
    
- **THOR Lite** = version gratuite
    
- **THOR** complet = orienté clients entreprise
    

#### À retenir

- plus avancé / plus orienté usage pro
    
- intéressant pour scanner sans saturer la machine

_Ressources : https://www.nextron-systems.com/thor-lite/_

```bash
cmnatic@thm:~$ ./thor-lite-linux-64 -h
Thor Lite
APT Scanner
Version 10.7.3 (2022-07-27 07:33:47)
cc) Nextron Systems GmbH
Lite Version

> Scan Options
  -t, --template string      Process default scan parameters from this YAML file
  -p, --path strings         Scan a specific file path. Define multiple paths by specifying this option multiple times. Append ':NOWALK' to the path for non-recursive scanning (default: only the system drive) (default [])
      --allhds               (Windows Only) Scan all local hard drives (default: only the system drive)
      --max_file_size uint   Max. file size to check (larger files are ignored). Increasing this limit will also increase memory usage of THOR. (default 30MB)

> Scan Modes
      --quick     Activate a number of flags to speed up the scan at cost of some detection.
                  This is equivalent to: --noeventlog --nofirewall --noprofiles --nowebdirscan --nologscan --noevtx --nohotfixes --nomft --lookback 3 --lookback-modules filescan
```

---

### FENRIR

**FENRIR** est un autre outil de Florian Roth.

Il a été créé pour contourner certaines **contraintes de dépendances** des outils précédents.

#### À retenir

- c’est un **script bash**
    
- fonctionne sur tout système capable d’exécuter **bash**
    
- approche simple et portable

_Ressources : https://github.com/Neo23x0/Fenrir_

```bash
cmnatic@thm:~$ ./fenrir.sh
##############################################################
    ____             _
   / __/__ ___  ____(_)___
  / _// -_) _ \/ __/ / __/
 /_/  \__/_//_/_/ /_/_/
 v0.9.0-log4shell

 Simple Bash IOC Checker
 Florian Roth, Dec 2021
##############################################################
```

---

### YAYA

**YAYA** signifie **Yet Another Yara Automaton**.  
Outil créé par l’**EFF** en **septembre 2020**.

#### Rôle

Aide à **gérer plusieurs dépôts de règles Yara**.

#### Fonctions principales

|Commande / fonction|Rôle|
|---|---|
|**update**|Met à jour les rulesets|
|**edit**|Désactive ou retire certains rulesets|
|**add**|Ajoute un ruleset personnalisé|
|**scan**|Lance un scan Yara sur un répertoire|

#### À retenir

- utile pour **centraliser et maintenir** plusieurs règles
    
- supporte l’ajout de **règles perso**
    
- **Linux uniquement**

_Ressources : https://www.eff.org/deeplinks/2020/09/introducing-yaya-new-threat-hunting-tool-eff-threat-lab_

```bash
cmnatic@thm-yara:~/tools$ yaya
YAYA - Yet Another Yara Automaton
Usage:
yaya [-h]  
    -h print this help screen
Commands:
   update - update rulesets
   edit - ban or remove rulesets
   add - add a custom ruleset, located at 
   scan - perform a yara scan on the directory at 
```

---

### Synthèse rapide

- **LOKI** → scanner IOC polyvalent
    
- **THOR Lite** → scanner IOC + YARA plus complet, multi-plateforme
    
- **FENRIR** → version simple en bash, très portable
    
- **YAYA** → outil de **gestion/automatisation** des règles Yara

---

## Using LOKI and its Yara rule set

### Contexte d’usage

En tant qu’analyste sécu, on récupère souvent des **IOC** depuis :

- rapports de threat intel
    
- blogs
    
- analyses IR / forensic
    

Exemples d’IOC :

- **hash**
    
- **IP**
    
- **nom de domaine**
    

Ces IOC, ainsi que des **règles Yara**, servent à détecter des menaces dans l’environnement.  
Si les outils de détection n’ont rien vu, on peut **ajouter ses propres règles** dans **LOKI**.

---

### Idée clé sur LOKI

LOKI possède déjà un **ensemble de règles Yara prêtes à l’emploi**, ce qui permet de commencer à scanner immédiatement.

---

### Emplacement des outils

#### Commande

ls

#### Résultat

Loki  yarGen

#### À retenir

- les outils utiles ici sont :
    
    - **Loki**
        
    - **yarGen**
        

---

### Voir les options de LOKI

#### Commande

python loki.py -h

#### Résultat

Affiche l’**aide** et les **options disponibles** de LOKI.

#### À retenir

- permet de voir les paramètres de scan
    
- première commande utile pour prendre en main l’outil
    

---

### Mettre à jour les signatures

#### Commande

python loki.py --update

#### Rôle

Télécharge / met à jour le dossier **`signature-base`** utilisé par LOKI pour scanner les menaces connues.

#### Note

Dans la VM du lab, cette commande a **déjà été exécutée**.

---

### Vérifier la base de signatures

#### Commande

ls ~/tools/Loki/signature-base

#### Résultat

iocs  misc  yara

#### À retenir

- **`iocs`** → indicateurs de compromission
    
- **`misc`** → autres ressources
    
- **`yara`** → règles Yara utilisées par LOKI
    

---

### Aller voir les règles Yara de LOKI

Il est conseillé d’explorer le dossier **`yara`** pour comprendre **ce que les règles recherchent**.

#### Idée

Permet de savoir :

- quels malwares / comportements sont couverts
    
- comment les règles sont écrites
    
- quels patterns sont chassés
    

---

### Lancer un scan LOKI

#### Commande

python ../../tools/Loki/loki.py -p .

### Signification

- lance **LOKI**
    
- `-p .` = scanne le **répertoire courant**
    

#### Contexte de l’exemple

Commande lancée depuis :

~/suspicious-files/file1

Donc LOKI scanne **les fichiers suspects du dossier courant**.

---

### Scénario du lab

Tu es analyste sécurité dans un **cabinet d’avocats**.  
Des fichiers suspects ont été trouvés sur un **serveur web** pendant une mise à jour du site corporate.  
Ces fichiers ont été copiés sur ta machine pour analyse dans le dossier :

suspicious-files

Objectif : utiliser **LOKI** pour analyser ces fichiers.

---

### Tableau récapitulatif

|Action|Commande|Résultat / but|
|---|---|---|
|Lister les outils|`ls`|Affiche `Loki` et `yarGen`|
|Voir l’aide de LOKI|`python loki.py -h`|Affiche les options|
|Mettre à jour les signatures|`python loki.py --update`|Récupère `signature-base`|
|Voir la base de signatures|`ls ~/tools/Loki/signature-base`|Affiche `iocs misc yara`|
|Scanner le dossier courant|`python ../../tools/Loki/loki.py -p .`|Analyse les fichiers du répertoire|

### Mini mémo

- `--update` → met à jour les signatures
    
- `signature-base` → base utilisée par LOKI
    
- `-p .` → scanne le dossier courant
    
- `yara/` → contient les règles Yara
    
- `iocs/` → contient les IOC

## Creating yara rules with yarGen

### Problème

- **LOKI ne détecte pas file2**
    
- risque : le **web shell** passe inaperçu sur d’autres serveurs
    

---

### Vérifier la taille du taf

strings 1ndex.php | wc -l

**Résultat**

3580

**Idée** : trop de strings à revoir à la main

---

### yarGen

- **outil de génération de règles Yara**
    
- crée des règles depuis les **strings du malware**
    
- retire les strings aussi présentes dans des fichiers légitimes (**goodware**)
    

---

### Mise à jour

python3 yarGen.py --update

**Effet** :

- met à jour les DB :
    
    - `good-opcodes`
        
    - `good-strings`
        

**Note THM** :

- impossible dans la VM (pas d’accès Internet)
    

---

### Générer une règle pour file2

python3 yarGen.py -m /home/cmnatic/suspicious-files/file2 --excludegood -o /home/cmnatic/suspicious-files/file2.yar

### Options

|Option|Rôle|
|---|---|
|`-m`|chemin des fichiers à analyser|
|`--excludegood`|exclut les strings de goodware|
|`-o`|fichier de sortie `.yar`|

---

### Résultat attendu

[=] Generated 1 SIMPLE rules.  
[=] All rules written to /home/cmnatic/suspicious-files/file2.yar  
[+] yarGen run finished

---

### À retenir

- `strings | wc -l` → compte les strings
    
- **yarGen** → génère une règle Yara automatiquement
    
- `--excludegood` → réduit les **faux positifs**
    
- sortie = fichier **`.yar`**
    
- ensuite : **relire la règle** et supprimer les strings douteuses si besoin
    

---

### Mini mémo

- **LOKI rate file2** → créer une règle
    
- trop de strings à la main → **yarGen**
    
- générer → relire → tester

### Ressource

Another tool created to assist with this is called [yarAnalyzer](https://github.com/Neo23x0/yarAnalyzer/) (you guessed it - created by Florian Roth). We will not examine that tool in this room, but you should read up on it, especially if you decide to start creating your own Yara rules.

- [https://www.bsk-consulting.de/2015/02/16/write-simple-sound-yara-rules/](https://www.bsk-consulting.de/2015/02/16/write-simple-sound-yara-rules/)
- [https://www.bsk-consulting.de/2015/10/17/how-to-write-simple-but-sound-yara-rules-part-2/](https://www.bsk-consulting.de/2015/10/17/how-to-write-simple-but-sound-yara-rules-part-2/)
- [https://www.bsk-consulting.de/2016/04/15/how-to-write-simple-but-sound-yara-rules-part-3/](https://www.bsk-consulting.de/2016/04/15/how-to-write-simple-but-sound-yara-rules-part-3/)

---

## Valhalla

**Valhalla** is an online Yara feed created and hosted by [Nextron-Systems](https://www.nextron-systems.com/valhalla/) (erm, Florian Roth)
### C’est quoi

- **feed Yara en ligne**
    
- créé / maintenu par **Nextron Systems**
    
- contient des **milliers de règles Yara de qualité**
    

![[YARA Valhalla.png]]

---

### Utilité

- **recherche threat intel**
    
- retrouver des règles liées à un fichier / malware / technique
    
- aider à **confirmer qu’un fichier est malveillant**
    

---

### Recherche possible

|Recherche par|Exemple|
|---|---|
|**keyword**|nom de malware / mot-clé|
|**tag**|catégorie / famille|
|**ATT&CK technique**|technique MITRE|
|**sha256**|hash du fichier|
|**rule name**|nom exact d’une règle|

---

### Infos fournies par une règle

![[YARA Valhalla newest yara rules.png]]

- **nom de la règle**
    
- **description**
    
- **lien de référence**
    
- **date de publication / soumission**
    

---

### Dans le scénario

- les **2 fichiers sont liés**
    
- **LOKI** les marque suspects
    
- toi, tu suspectes **malveillant**
    
- **yarGen** sert à créer une règle de détection
    
- **Valhalla** sert à faire la **recherche de threat intel** pour appuyer l’analyse
    

---

### À retenir

- **Valhalla = base de règles Yara + threat intel**
    
- utile pour **rechercher**, **comprendre**, **corréler**
    
- pratique si tu ne sais pas trop **lire le code** mais veux des **indices fiables**
    

---

### Mini mémo

- **yarGen** → crée une règle
    
- **Valhalla** → cherche des infos / règles existantes
    
- but final → **justifier l’éradication** des fichiers sur le réseau