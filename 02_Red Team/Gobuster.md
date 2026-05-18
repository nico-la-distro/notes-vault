**Gobuster** est un outil offensif open-source en **Go** pour faire de l’**énumération par brute force** via **wordlists**. Sert à trouver : **répertoires/fichiers web**, **sous-domaines DNS**, **vhosts**, **buckets S3 / GCS** (et autres). Se place entre **reconnaissance** et **scanning**.

## **Key concept**

- **Enumeration** : lister des ressources (accessibles ou non) → ex : répertoires web.

- **Brute force** : tester toutes les possibilités jusqu’au match → Gobuster teste chaque mot d’une wordlist.

## **Commandes / flags**

**commands**

|Commande|Sert à quoi|Cible typique|
|---|---|---|
|`dir`|Énumération répertoires/fichiers|Site web (URL)|
|`dns`|Énumération sous-domaines|Domaine|
|`vhost`|Énumération vhosts|IP/host (souvent IP en URL)|
|`fuzz`|Fuzzing via mot-clé `FUZZ` (URL/headers/body)|Applis web|
|`s3`|Énumération buckets AWS S3|Noms de buckets|
|`gcs`|Énumération buckets Google Cloud Storage|Noms de buckets|
|`tftp`|Énumération TFTP|Serveur TFTP|

---

**flags**

|Short|Long|Rôle|Quand l’utiliser|
|---|---|---|---|
|`-t`|`--threads`|Nombre de threads (défaut 10)|Accélérer (si ressources OK)|
|`-w`|`--wordlist`|Wordlist utilisée|Toujours (brute force)|
|—|`--delay`|Délai entre requêtes|Éviter détection/rate limit|
|—|`--debug`|Debug output|Erreurs / comportement inattendu|
|`-o`|`--output`|Sortie vers fichier|Garder les résultats|

---

**exemples**

|Commande|Ce que ça fait|
|---|---|
|`gobuster dir -u "http://www.example.thm/" -w /usr/share/wordlists/dirb/small.txt -t 64`|Brute force de chemins web avec `small.txt` sur l’URL, en 64 threads (GET sur chaque `/mot/`).|


## **Directory / Files enumeration**

**flags dir importants**

| Flag | Long flag                  | Description (utile)                                      |
| ---- | -------------------------- | -------------------------------------------------------- |
| `-c` | `--cookies`                | Ajoute un cookie à chaque requête (ex: session id).      |
| `-x` | `--extensions`             | Teste des extensions de fichiers (ex: `.php`, `.js`).    |
| `-H` | `--headers`                | Ajoute un header complet à chaque requête.               |
| `-k` | `--no-tls-validation`      | Ignore la validation TLS (utile si certif self-signed).  |
| `-n` | `--no-status`              | N’affiche pas les status codes (sortie plus clean).      |
| `-P` | `--password`               | Avec `--username`, fait des requêtes authentifiées.      |
| `-s` | `--status-codes`           | Filtre les status codes affichés (ex: `200`, `300-400`). |
| `-b` | `--status-codes-blacklist` | Masque certains status codes (override `-s`).            |
| `-U` | `--username`               | Avec `--password`, fait des requêtes authentifiées.      |
| `-r` | `--followredirect`         | Suit les redirects (301/302) vers l’URL cible.           |

**utilisation**

- Format : `gobuster dir -u "http://target" -w /path/to/wordlist`
- Exemple redirect : `gobuster dir -u "http://www.example.thm" -w ... -r`
- Notes :
    - URL doit inclure le **protocole** (`http/https`) sinon échec
    - IP vs hostname : avec **IP** tu peux viser le “mauvais” site (virtual hosting) → préférer **hostname**
    - Pas de **récursivité** : si tu trouves `/resources/`, relance un scan sur `.../resources`
- Exemple extensions : `gobuster dir -u "http://www.example.thm" -w ... -x .php,.js` (cherche dossiers + fichiers `.php`/`.js`)

## **Subdomain enumeration**

**dns mode** : brute force des **sous-domaines** d’un domaine (important en pentest : un sous-domaine peut être vulnérable même si le domaine principal est patch).

**help** : `gobuster dns --help` (moins de flags que `dir`).

|Flag|Long flag|Description|
|---|---|---|
|`-d`|`--domain`|Domaine à énumérer (**requis**).|
|`-w`|—|Wordlist à utiliser (**requis**).|
|`-i`|`--show-ips`|Affiche les IP vers lesquelles le domaine/sous-domaines résolvent.|
|`-c`|`--show-cname`|Affiche les enregistrements CNAME (**incompatible avec `-i`**).|
|`-r`|`--resolver`|Utilise un serveur DNS (resolver) personnalisé pour la résolution.|

**Syntaxe**
- `gobuster dns -d example.thm -w /path/to/wordlist`

**Exemple**
- `gobuster dns -d example.thm -w /usr/share/wordlists/SecLists/Discovery/DNS/subdomains-top1million-5000.txt`
- Principe : chaque mot devient `<mot>.example.thm` (ex: `all.example.thm`) puis Gobuster affiche les sous-domaines trouvés (ex: `www`, `shop`, etc.).

## **vhost enumeration**

**vhost mode** : brute force des **virtual hosts** (plusieurs sites sur la **même machine/IP**). Peut ressembler à des sous-domaines, mais :
- **vhost** = basé sur l’IP + en-tête **Host** (sites sur le même serveur)
- **subdomain** = basé sur **DNS**

**Diff vhost vs dns**
- `vhost` : fait des requêtes web sur l’URL `-u` en changeant **Host** avec chaque mot de la wordlist
- `dns` : fait des **DNS lookups** sur `<mot>.<domaine>`

**Help** : `gobuster vhost --help`

|Flag|Long flag|Description|
|---|---|---|
|`-u`|`--url`|URL de base (cible) pour brute force des vhosts.|
|`-w`|—|Wordlist (**requis**).|
|`-m`|`--method`|Méthode HTTP à utiliser (GET, POST…).|
|`-r`|`--follow-redirect`|Suit les redirects HTTP.|
|—|`--domain`|Ajoute/fixe le domaine pour former un hostname valide (ex: `example.thm`).|
|—|`--append-domain`|Append le domaine de base à chaque mot (ex: `word.example.com`) → évite faux positifs.|
|—|`--exclude-length`|Exclut des résultats selon la taille du body (filtre faux positifs).|

**Syntaxe**
- `gobuster vhost -u "http://example.thm" -w /path/to/wordlist` (`-u` + `-w` requis)

**Principe (requêtes)**
- Gobuster envoie `GET /` en changeant `Host: <mot>.example.thm`
    - `<mot>` vient de la wordlist
    - `example.thm` vient de `--domain`

**Exemple “réaliste” (sans DNS complet)**
- `gobuster vhost -u "http://<IP>" --domain example.thm -w ... --append-domain --exclude-length 250-320`
- `--exclude-length` sert à virer les réponses “404 de même taille” → sinon beaucoup de **false positives** (on vise plutôt des réponses **200 OK** comme vrais positifs).


