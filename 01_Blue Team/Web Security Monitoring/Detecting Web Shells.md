## Web Shell Overview

### What is a Web Shell?

Script malveillant uploadé sur un serveur web → exécution de commandes à distance.

Rôle dual : **initial access** (via file upload vuln) + **persistence**.

Une fois déployé : recon, privesc, lateral movement, data exfiltration.

![[detecting web shells exemple.png]]

### Web Shell Deployment

Conditions requises : file upload vuln, misconfiguration, ou accès préalable.

Cause : absence de validation du type, extension, contenu ou destination du fichier.

### Examples in the Wild

**Hafnium (APT chinois) — ProxyLogon (Exchange)**

- Upload de `.aspx` web shells dans `\inetpub\wwwroot\aspnet_client\`
- Modification de fichiers `.aspx` existants dans `\install_path\FrontEnd\HttpProxy\owa\auth\`
- Suite : exécution de commandes, recon, dump credentials, nouveaux comptes, lateral movement, exfiltration, cover tracks

**Conti Ransomware — même vuln Exchange**

- Upload de `aspnetclient_log.aspx` dans `\aspnet_client\`
- En quelques minutes : backup web shell uploadé + cartographie réseau (computers, DC, domain admins)

---

## Anatomy of a Web Shell

**Legitimate Function Abuse**

Les web shells abusent de fonctions légitimes. En PHP : `shell_exec()`, `exec()`, `system()`, `passthru()`.

**Under the Hood**

Fonctionnement d'un web shell PHP basique :

1. Checks if the `cmd` parameter is present in the URL `?cmd=whoami`
2. Stores the user supplied command in the variable `$cmd`
3. Executes the command using `shell_exec()`
4. Displays the output
5. HTML for the user interface
6. Command to execute
7. Output

![[detecting web shells php web shell.png]]

Les web shells vont du one-liner à l'interface complète (GUI, auth, file manager).

### A Web Shell in Action

Accès via navigateur ou `curl`. Les commandes doivent être URL-encodées.

```
/awebshell.php?cmd=<encoded_command>
# ls -la → ls%20-la
```

Encodage rapide : [CyberChef](https://gchq.github.io/CyberChef/#recipe=URL_Encode\(false\))

---

## Log-Based Detection

### Web Server Logs

Les access logs incluent : IP client, timestamp, méthode HTTP, ressource, code réponse, user-agent, referrer.

![[detecting web shells web server logs.png]]

### Web Indicators

**Unusual HTTP Methods & Request Patterns**

|Méthode|Abus possible|
|---|---|
|GET|Recon / interaction avec web shell|
|POST|Upload ou interaction avec web shell|
|PUT|Upload direct d'un web shell|
|DELETE|Nettoyage des traces|
|OPTIONS|Reconnaissance|
|HEAD|Détection de fichiers|

Séquence d'attaque typique : GET répétés (fuzzing) → POST vers répertoire valide (upload) → GET répétés sur le fichier uploadé (interaction).

![[detecting web shells attack exemple in logs.png]]

**Suspicious User-Agents & IP Addresses**

- Altered User-Agents: `Mozilla/4.0+(+Windows+NT+5.1)` shortened to `Mozilla/4.0`
- Outdated User-Agents: `Mozilla/4.0 (compatible; MSIE 6.0)` MSIE 6.0 released in 2001
- Blacklisted User-Agents: `curl/1.XX.X` or `wget/1.XX.X` for example
- Suspicious IP Addresses: A network normally only sees internal traffic so an outside IP address would be suspicious

**Query Strings**

- Paramètres suspects : `cmd=`, `exec=`
- Encodage Base64 : `?query=whoami` → `?query=d2hvYW1p`

**Missing Referrer**

Referrer absent = accès direct possible → indicateur faible mais notable (des raisons légitimes existent).

**Sample suspicious web request** including some of the above indicators.

1. Known malicious or untrusted IP address.
2. Abnormal timestamp. Perhaps outside of normal business hours.
3. POST request with a search query string to a malicious file.
4. No referrer. So this page was accessed directly. (not always a valid indicator)
5. A suspicious User-Agent string that is not typically associated with a web browser.

![[detecting web shells suspicious web request.png]]

### Auditd

Utilitaire Linux natif → audit trail via `audit.log`. Les règles définissent ce qui est loggé (fichiers modifiés, programmes exécutés, etc.).

bash

```bash
ausearch -k web_shell

time->Wed Jul 23 06:20:36 2025 // A log matching the web_shell rule 
"name = /uploads/webshell.php" 
"OGID = www-data"
```

### Web & Auditd Correlation

Croiser les logs web (POST suspect) avec auditd (syscall `creat` ou `execve`) → confirme création/exécution de fichier et l'utilisateur impliqué.

### Leverage SIEM Platforms

- Centralisation et corrélation multi-sources
- Requêtes ciblées pour détecter les web shells
- Analyse plus efficace des logs

---

## Beyond Logs

### File System Analysis

Le web shell doit être stocké quelque part. Note : WordPress/Django stockent le contenu en base de données → injection possible dans posts, thèmes ou settings, invisible en filesystem.

**Répertoires par défaut à surveiller**

|Serveur|Chemin|
|---|---|
|Apache|`/var/www/html/`|
|Nginx|`/usr/share/nginx/html/`|

Répertoires fréquemment ciblés : `/uploads/`, `/images/`, `/admin/`, `/tmp/`

**Fichiers suspects**

- Extensions exécutables : `.php`, `.jsp`
- Double extension : `image.jpg.php`
- Noms aléatoires ou déviants des fichiers applicatifs standards

**Commandes utiles**

bash

```bash
# Fichiers .php modifiés entre deux dates
find /var/www -type f -name "*.php" -newerct "2025-07-01" ! -newerct "2025-08-01"

/var/www/html/uploads/awebshell.php // Web shell created between the dates above.

# Recherche de fonctions suspectes
grep -r "eval(" wp-content

/wp-content/uploads/awebshell2.php :eval(b64_dd($['cmd'])); // Web shell containing eval(
```

### Network Traffic Analysis

Mêmes indicateurs qu'en log analysis, plus :

- Unusual HTTP Methods & Request Patterns
- Suspicious User-Agents & IP Addresses
- Encoded Payloads
- Malicious Code or Commands in Request Bodies
- Unexpected Protocols or Ports
- Unexpected Resource Usage
- Web Server Processes Spawning Command Line Tools

Les PCAPs permettent de voir le source code du web shell directement dans le payload (ex : `POST /upload.php` avec `webshell.php` visible).

![[detecting web shells wireshark.png]]

![[detecting web shells follow http for payload.png]]

**Filtres Wireshark utiles**

```
http.request.method == "POST"
http.request.uri contains ".php"
http.user_agent
```

Référence complète : [Wireshark HTTP filters](https://www.wireshark.org/docs/dfref/h/http.html)

---

## Questions

#### Which IP address likely belongs to the attacker?

```bash
cat /var/log/apache2/access.log | grep "404"
```

![[detecting web shells t6q1.png]]

**Answer** : 203.0.113.66

#### What is the first directory that the attacker successfully identifies?

```bash
cat /var/log/apache2/access.log | grep "200"
```

![[detecting web shells t6q2.png]]

**Answer** : /wordpress

#### What is the name of the `.php` file the attacker uses to upload the web shell?

```bash
cat /var/log/apache2/access.log | grep ".php"
```

![[detecting web shells t6q3.png]]

**Answer** : upload_form.php

#### What is the first command run by the attacker using the newly uploaded web shell?

We can see in the previous screen ?cmd=whoami in the query string

**Answer** : whoami

#### After gaining access via the web shell, the attacker uses a command to download a second file onto the server. What is the name of this file?

![[detecting web shells t6q4.png]]

Always with the same output of the last command, we can see that the attacker download onto the server linpeas.sh. This is a script using for hardening, so it can list vulnerabilities in the server.

**Answer** : linpeas.sh

#### The attacker has hidden a secret within the web shell.  Use `cat` to investigate the web shell code and find the flag.

```bash
cat /var/www/html/wordpress/wp-content/uploads/shadyshell.php
```

![[detecting web shells t6q5.png]]

**Answer** : THM{W3b_Sh3ll_Int3rnals}

---

