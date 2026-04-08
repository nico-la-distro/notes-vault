- **Hydra** : outil de **brute force en ligne** pour **tester / casser des mots de passe** sur des services de connexion (outil rapide de “login password cracking”).

- **Principe** : Hydra essaie automatiquement des combinaisons **(liste d’identifiants / mots de passe)** sur un service (ex : SSH, formulaires web, FTP…), beaucoup plus vite qu’à la main, pour **trouver le bon mot de passe**.

- **Protocoles supportés** (idée à retenir) : Hydra peut attaquer **de nombreux services** (exemples fréquents : **SSH, FTP, HTTP/HTTPS (formulaires GET/POST), SMB, RDP, SMTP, IMAP/POP3, LDAP, MySQL/Postgres/MSSQL**, etc. + beaucoup d’autres).

- **Point sécurité essentiel** : l’existence d’outils comme Hydra montre l’importance de :
    - mots de passe **longs** (plus de 8 caractères),
    - **complexes** (variés, caractères spéciaux),
    - **non courants** (éviter ceux présents dans des listes massives de mots de passe).

- **À retenir (exemple)** : beaucoup d’applications/équipements gardent des **identifiants par défaut** (ex : **admin:password**) → **à changer immédiatement** (cas fréquent : caméras CCTV, frameworks web, etc.).

**official repository** : https://github.com/vanhauser-thc/thc-hydra
**kali hydra tool page** : https://en.kali.tools/?p=220

## **Commandes Hydra**

- Les **options Hydra dépendent du service/protocole** attaqué (FTP, SSH, formulaire web, etc.).

### **Exemple FTP**

|Objectif|Commande|
|---|---|
|Brute force FTP avec un user + une liste de mots de passe|`hydra -l user -P passlist.txt ftp://MACHINE_IP`|

---

### **SSH (template + options)**

|Option|Signification|
|---|---|
|`-l`|username (login)|
|`-P`|fichier liste de mots de passe|
|`-t`|nombre de threads (tentatives en parallèle)|
`hydra -l root -P /full/path/to/wordlist MACHINE_IP -t 4 ssh`

---

### **Formulaire Web (POST)**

**Commande type :**  
`hydra -l <username> -P <wordlist> MACHINE_IP http-post-form "<path>:<login_credentials>:<invalid_response>" -V`

|Élément|Rôle|Exemple|
|---|---|---|
|`http-post-form`|précise que le formulaire est en POST|`http-post-form`|
|`<path>`|chemin de la page de login|`/` ou `login.php`|
|`<login_credentials>`|champs du formulaire + placeholders Hydra|`username=^USER^&password=^PASS^`|
|`<invalid_response>`|texte repéré quand l’auth échoue|`F=incorrect`|
|`-V`|affiche chaque tentative|`-V`|

|Exemple concret|Interprétation|
|---|---|
|`hydra -l <username> -P <wordlist> MACHINE_IP http-post-form "/:username=^USER^&password=^PASS^:F=incorrect" -V`|page login = `/`, remplace `^USER^` et `^PASS^`, échec si réponse contient “incorrect”|

**port non standard :**

|Cas|Option|Exemple|
|---|---|---|
|site sur un port ≠ 80/443|`-s <port>`|`... -s 8080 -V`|

## **Tips**

### **Checklist avant hydra**

|À vérifier|Comment|Pourquoi|
|---|---|---|
|**Méthode** (GET/POST)|DevTools → Network|sinon mauvais module|
|**Path exact**|ex `/login`|sinon tu attaques la mauvaise route|
|**Champs exacts**|ex `username`, `password`|sinon les creds sont ignorés|
|**Signal d’échec**|texte / code / redirect|sinon mauvais `F=`/`S=`|

### **"PATH : DATA : COND"**

|Élément|Règle|Exemple|
|---|---|---|
|**PATH**|chemin **sans** `http://`|`/login`|
|**DATA**|champs + placeholders|`username=^USER^&password=^PASS^`|
|**COND**|`F=` échec **ou** `S=` succès|`F=incorrect`|

