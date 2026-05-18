_**R**eversing **E**ngineering **M**alware **n**ux ("nux" pour "Linux")_

- **Distribution Linux spécialisée** pour l’analyse malware.
- Fournit un **environnement type sandbox** pour disséquer des échantillons **sans exposer ton système principal**.

**outils inclus**

|Outil|Utilité (très court)|
|---|---|
|Volatility|Analyse de **dump mémoire**|
|YARA|Règles de détection / matching de patterns|
|Wireshark|Analyse de trafic réseau (PCAP/live)|
|oledump|Analyse de documents OLE (Office, etc.)|
|INetSim|**Faux services réseau** (simulation Internet)|

---
# **thm exo (pur)**

## **File Analysis**

_(cd : https://tryhackme.com/room/remnuxgettingstarted)_

### But de la tâche

- Faire une **analyse statique** d’un Excel potentiellement malveillant (`agenttesla.xlsm`) avec **oledump.py**.
    
- Repérer et extraire les **macros VBA** (souvent vecteur d’infection).
    

---

### oledump.py : ce que c’est

- Outil Python pour analyser les fichiers **OLE2** (Compound File Binary Format / Structured Storage).
    
- Les documents Office peuvent contenir **plusieurs “streams”** (flux de données) dans un seul fichier → utile pour **forensic / détection malware**.
    

---

### Workflow (étapes + commandes)

|Étape|Commande|Résultat attendu / pourquoi|
|---|---|---|
|1. Se placer dans le dossier|`cd /home/ubuntu/Desktop/tasks/agenttesla/`|Accès au sample|
|2. Lister les streams OLE|`oledump.py agenttesla.xlsm`|Voir où sont les macros (`vbaProject.bin`, streams VBA…)|
|3. Cibler le stream macro|`oledump.py agenttesla.xlsm -s 4`|Affiche le contenu du stream `A4` (souvent hex)|
|4. Décompresser le VBA|`oledump.py agenttesla.xlsm -s 4 --vbadecompress`|Rend la macro lisible (VBA décompressé)|

---

### Lecture de la sortie oledump (points clés)

|Indicateur|Signification|
|---|---|
|`A: xl/vbaProject.bin`|Conteneur VBA (souvent là que vivent les macros)|
|Streams `A1, A2, A3…`|“Data streams” internes|
|`M` (majuscule)|**Macro détectée** → stream à prioriser|
|Exemple : `A4: M ... 'VBA/ThisWorkbook'`|Macro dans `ThisWorkbook` (souvent auto-exécutée à l’ouverture)|

---

### Obfuscation VBA repérée : `Sqtnew`

- La macro contient une chaîne **obfusquée** (avec `*` et `^`) :
    
    - `Sqtnew = " ^p*o^*w*e*r*s..."`
        
    - puis :
        
        - `Replace(Sqtnew, "*", "")`
            
        - `Replace(Sqtnew, "^", "")`
            

#### Déobfuscation avec CyberChef

|Action|Opération CyberChef|
|---|---|
|Supprimer `*`|Find/Replace : Find `*` (Simple string) → Replace vide|
|Supprimer `^`|Find/Replace : Find `^` (Simple string) → Replace vide|

---

### Payload final (PowerShell) et explication

Après nettoyage, on obtient un PowerShell du style :

- `powershell -WindowStyle hidden -executionpolicy bypass; ...`
    
- Crée un fichier temporaire (`GetTempFileName`), le renomme en **.exe**
    
- Télécharge un exécutable via `Invoke-WebRequest`
    
- Lance l’exe via `Start-Process`
    

#### Commandes PowerShell importantes

|Élément|Rôle|
|---|---|
|`-WindowStyle hidden`|Exécution **invisible** pour l’utilisateur|
|`-executionpolicy bypass`|Contourne la politique d’exécution (temporairement)|
|`Invoke-WebRequest -Uri ... -OutFile ...`|**Télécharge** le fichier|
|`Start-Process $TempFile`|**Exécute** le binaire téléchargé|

---

### IOCs / éléments à retenir

|Type|Valeur|
|---|---|
|URL|`http://193.203.203.67/rt/Doc-3737122pdf.exe`|
|IP|`193.203.203.67`|
|Payload|`Doc-3737122pdf.exe`|
|Technique|Macro VBA → lance PowerShell → download + exec|

---

### Conclusion (en 1 phrase)

À l’ouverture de `agenttesla.xlsm`, une **macro VBA** déclenche un **PowerShell caché** qui **télécharge** un `.exe` depuis `193.203.203.67` puis **l’exécute** — technique classique “download & execute” pour éviter la détection.

## **Fake Network to Aid Analysis**

_**AID = Automatic Incident Detection** : détection automatique d’incidents._
### Pourquoi faire

- En **analyse dynamique**, le plus important est d’observer le **comportement réseau** (C2, téléchargement de payload, callbacks).
    
- Plutôt que de créer toute une infra, REMnux fournit **INetSim** : un outil qui **simule des services Internet** (DNS/HTTP/HTTPS/SMTP/FTP, etc.) pour “piéger” le malware dans un réseau contrôlé.
    

---

### Setup minimal (ce qui compte vraiment)

#### 1) Repérer l’IP de REMnux

- `ifconfig` (ou visible dans le prompt)  
    ➡️ Cette IP servira de **réponse DNS par défaut** et d’adresse du “serveur” simulé.
    

#### 2) Configurer INetSim pour renvoyer _l’IP de REMnux_ en DNS

- Éditer : `sudo nano /etc/inetsim/inetsim.conf`
    
- Trouver `#dns_default_ip 0.0.0.0`
    
- **Décommenter** et remplacer par l’IP REMnux :
    
    - `dns_default_ip 10.x.x.x`
        

Vérif :

- `cat /etc/inetsim/inetsim.conf | grep dns_default_ip`
    

#### 3) Lancer INetSim

- `sudo inetsim`
    
- Indicateur OK : **“Simulation running”**
    
- Certains services peuvent échouer (ex: `http_80_tcp - failed!`) → pas bloquant si le reste tourne (ex: HTTPS).
    

---

### Test côté “victime” (AttackBox)

- Ouvrir `https://<IP_REMNUX>` → accepter le risque (certificat auto-signé normal)
    
- Simuler un comportement malware “download” :
    
    - `sudo wget https://<IP_REMNUX>/second_payload.zip --no-check-certificate`
        
    - (idem pour `second_payload.ps1`)
        
- Les fichiers servis sont **fake** : l’objectif est de **voir les requêtes** et **loguer** l’activité.
    

---

### Où lire les preuves (logs)

- Stop INetSim → il écrit un **rapport**
    
- Chemin : `/var/log/inetsim/report/`
    
- Lire :
    
    - `sudo cat /var/log/inetsim/report/report.<id>.txt`
        

Ce que tu récupères dans le rapport :

|Champ utile|Exemple|
|---|---|
|Protocole|HTTPS|
|Méthode|GET|
|URL demandée|`https://IP/second_payload.zip`|
|Fichier servi (fakefile)|`/var/lib/inetsim/http/fakefiles/...`|
|Timeline|dates/heures des connexions|

---

### Résumé “one-liner”

**INetSim = faux Internet local** : tu forces le DNS à répondre avec l’IP REMnux, tu lances la simulation, puis tu observes/collectes dans les rapports toutes les requêtes (URLs, méthodes, protocole, fichiers demandés) comme si le malware téléchargeait un second payload.

## **Memory Investigation: Evidence Preprocessing**

- Pratique classique : **pré-traiter la preuve** (ici une **image mémoire**) en lançant des outils et en **sauvegardant les sorties** (txt/JSON).
    
- But : **accélérer l’enquête** ensuite (recherches rapides, grep, corrélation, tri, partage avec d’autres analystes).
    
- Outil central mémoire : **Volatility** (ici **Volatility 3**, déjà dans REMnux).
    

---

### Volatility 3 : workflow minimal

### Commande de base

- `vol3 -f wcry.mem <plugin>`
    

#### Plugins Windows importants (ce que chacun apporte)

|Plugin|Sert à quoi (ultra-court)|
|---|---|
|`windows.pstree.PsTree`|Arbre parent/enfant des processus (relations)|
|`windows.pslist.PsList`|Processus **actifs**|
|`windows.psscan.PsScan`|Scan “brut” des processus (peut retrouver des processus cachés/terminés)|
|`windows.cmdline.CmdLine`|Arguments/command line des process (souvent très révélateur)|
|`windows.dlllist.DllList`|DLL/modules chargés par process|
|`windows.filescan.FileScan`|Scan des objets fichiers présents en mémoire|
|`windows.malfind.Malfind`|Zones mémoire suspectes (injection/code)|

> Point mémo : **PsList vs PsScan** → PsList = “liste normale”, PsScan = “scan pour trouver plus”.

---

### Préprocess en bulk (le truc à vraiment retenir)

Au lieu de lancer chaque plugin à la main → **boucle + redirection vers fichiers** :

|Élément|Sens|
|---|---|
|`for plugin in ...; do ...; done`|boucle sur une liste de plugins|
|`vol3 -q`|mode silencieux (pas de progress)|
|`> wcry.$plugin.txt`|chaque sortie va dans un fichier nommé par plugin|

Commande type :

`for plugin in windows.malfind.Malfind windows.psscan.PsScan windows.pstree.PsTree windows.pslist.PsList windows.cmdline.CmdLine windows.filescan.FileScan windows.dlllist.DllList; do   vol3 -q -f wcry.mem $plugin > wcry.$plugin.txt done`

Résultat attendu : **un fichier texte par plugin** (7 fichiers).

---

### Préprocess avec `strings` (complément incontournable)

Objectif : extraire des **chaînes lisibles** (souvent URLs, chemins, clés, noms, commandes).

|Commande|Contenu extrait|
|---|---|
|`strings wcry.mem > wcry.strings.ascii.txt`|ASCII|
|`strings -e l wcry.mem > wcry.strings.unicode_little_endian.txt`|Unicode 16-bit little endian (Windows courant)|
|`strings -e b wcry.mem > wcry.strings.unicode_big_endian.txt`|Unicode 16-bit big endian|

---

### One-liner de révision

**Préprocess mémoire = Volatility (plugins clés) + strings, le tout en bulk vers des .txt**, pour que l’analyse derrière soit juste du tri/grep/recherche rapide.

