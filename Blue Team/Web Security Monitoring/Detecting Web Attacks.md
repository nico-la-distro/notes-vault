## Client-Side Attacks

Exploitent les failles dans le comportement utilisateur ou son appareil (navigateur, plugins tiers). Objectifs : vol de session, données sensibles, usurpation d'identité.

### SOC Limitations

Les logs serveur et captures réseau ne voient pas ce qui se passe **dans** le navigateur. Les attaques client-side ne génèrent pas de requêtes HTTP suspectes → détection difficile sans endpoint monitoring ou contrôles browser-side.

### Common Client-Side Attacks

|Attaque|Principe|
|---|---|
|**XSS**|Injection de script malveillant dans une page de confiance, exécuté dans le navigateur de la victime (vol de cookies, session)|
|**CSRF**|Le navigateur est trompé pour envoyer des requêtes non autorisées au nom de l'utilisateur|
|**Clickjacking**|Superposition d'éléments invisibles sur du contenu légitime pour piéger les clics|

---

## Server-Side Attacks

Exploitent les vulnérabilités du serveur, du code applicatif ou du backend. Objectifs : accès non autorisé, vol de données, dommages aux services.

### Catching Server-Side Attacks

Avantage défenseur : chaque requête est **loggée** et transite par le réseau → traces exploitables dans les logs et le trafic réseau.

### Common Server-Side Attacks

|Attaque|Principe|
|---|---|
|**Brute-force**|Tentatives répétées de credentials via outils automatisés|
|**SQLi**|Manipulation des requêtes SQL via concaténation non sécurisée → accès/modification BDD|
|**Command Injection**|Input utilisateur passé directement au système → exécution de commandes avec les permissions de l'appli|

---

## Log-Based Detection

### Access Log Format

| Champ            | Indicateur suspect                                              |     |
| ---------------- | --------------------------------------------------------------- | --- |
| Client IP        | IP malveillante connue, hors zone géographique attendue         | 1   |
| Timestamp / Page | Requêtes à heures inhabituelles ou répétées rapidement          | 2   |
| Status Code      | Nombreux 404 → scan de pages                                    | 3   |
| Response Size    | Taille anormalement petite ou grande                            | 4   |
| Referrer         | Page de référence incohérente avec la navigation normale        | 5   |
| User-Agent       | Navigateur obsolète ou outil d'attaque (ex: `sqlmap`, `wpscan`) | 6   |

![[detecting web attacks access log format.png]]

### Attacks in Logs

Séquence type d'une attaque :

1. **Directory fuzzing** → réponses `200` = répertoires/fichiers valides trouvés
2. **Brute-force** → nombreux `POST` rapides sur un form (ex: `login.php`) → dernier `302 Found` = login réussi, redirection vers `/account`
3. **SQLi** → payloads dans query string sur un form (ex: `/search`) : `' OR '1'='1`, `1' OR 'a'='a`

![[detecting web attacks.png]]

### Log Limitations

Les access logs **ne capturent pas** le body des requêtes POST (credentials, payloads soumis). Les GET peuvent logger le query string complet, mais selon la config serveur.

```
10.10.10.100 [12/Aug/2025:14:32:10] "POST /login HTTP/1.1" 200 532 "/home.html" "Mozilla/5.0"
# → On voit que la requête a eu lieu, pas ce qui a été soumis
```

GET : query string souvent loggé → payloads SQLi visibles. POST : body non loggé → détection limitée aux métadonnées.

---

## Network-Based Detection

Plus verbeux que les logs : capture les headers HTTP complets, bodies POST, cookies, fichiers transférés.

**Limite** : trafic chiffré (HTTPS, SSH) → payloads illisibles sans clés de déchiffrement.

### Attacks in Network Traffic

Les captures réseau complètent les logs en révélant ce que les logs cachent :

| Étape          | Ce que les logs montrent         | Ce que Wireshark ajoute                                |     |
| -------------- | -------------------------------- | ------------------------------------------------------ | --- |
| Brute-force    | Requêtes POST répétées           | Credentials réels testés (ex: `password123`)           | 1   |
| SQLi           | Payloads dans query string (GET) | Payload complet + réponse du serveur (données dumpées) | 2   |
| Directory fuzz | Codes 200/404                    | Requêtes complètes avec headers                        | 3   |

![[detecting web attacks wireshark.png]]

### Tips Wireshark

```
http                        # filtrer uniquement le trafic HTTP
ip.dst == 10.10.20.200      # filtrer par IP destination
http.user_agent             # filtrer par User-Agent
```

Clic droit sur un paquet → **Follow HTTP Stream** → reconstruit la requête + réponse complète.

![[detecting web attacks hydra.png]]

![[detecting web attacks sqli.png]]

---

## Web Application Firewall

WAF = gatekeeper entre internet et le serveur. Inspecte les requêtes complètes, peut déchiffrer le TLS (contrairement à Wireshark passif).

### Rules

|Type de règle|Exemple|
|---|---|
|Block common attack patterns|Bloquer User-Agent `sqlmap`|
|Deny known malicious sources|Bloquer IPs de botnets, géo-blocking|
|Custom-built rules|Autoriser uniquement GET/POST sur `/login`|
|Rate-limiting|Max 5 tentatives de login/minute/IP|

Exemple de règle custom :

```
If User-Agent contains "sqlmap"
then BLOCK
```

### Challenge-Response Mechanisms

Alternative au blocage pur : **CAPTCHA** pour vérifier si la requête vient d'un humain. Utile pour les règles à fort risque de faux positifs (ex: traffic hors zone géo).

### Integrating Known Indicators and Threat Intelligence

- Règles intégrées couvrant l'**OWASP Top 10**
- Threat intel feeds → blocage automatique d'IPs malveillantes et User-Agents suspects
- Mises à jour régulières : APT, CVE récentes, botnets, VPNs, anonymizers

[Check out(opens in new tab)](https://blog.cloudflare.com/new-waf-intelligence-feeds) how Cloudflare maintains curated IP lists, from sources like botnets, VPNs, anonymizers, and malware, based on global threat intelligence.

