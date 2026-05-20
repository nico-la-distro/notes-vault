## DoS and DDoS Attacks

**DoS** : submerger un site/app pour le rendre inaccessible (focus room : Layer 7 / application).

![[detecting web DDoS DoS.png]]

Succès = le service web ne fonctionne plus comme prévu, quelle que soit l'échelle.

**DDoS** : même principe mais via un **botnet** (machines infectées par malware) → contourne les limites CPU/RAM/bande passante d'une seule machine.

![[detecting web DDoS DDoS.png]]

### Types of Denial-of-Service Attacks

|Type|Description|
|---|---|
|Slowloris|Nombreuses requêtes HTTP partielles → épuise les connexions serveur|
|HTTP Flood|Volume massif de requêtes HTTP|
|Cache Bypass|Force l'origin server à répondre en contournant le CDN|
|Oversized Query|Requêtes volumineuses → charge CPU/mémoire excessive|
|Login/Form Abuse|Surcharge de la logique d'auth (login, reset password)|
|Faulty Input Validation Abuse|Exploitation d'une validation d'entrée défaillante|

Tous ces types peuvent être lancés en DoS (machine unique) ou DDoS (botnet).

---

## Attack Motives

|Motif|Description|Exemple|
|---|---|---|
|Financial Loss|Stopper les ventes|Flood e-commerce en période de fêtes|
|Extortion|Rançon pour arrêter l'attaque|Ransom DDoS contre une banque|
|Hacktivism|Protestation politique/sociale|Attaque de sites gouvernementaux|
|Distraction|Détourner l'attention pendant une autre attaque|DDoS pendant intrusion sur autre infra|
|Competition|Nuire à un concurrent|DDoS pendant le lancement produit rival|
|Denial of Wallet|Faire exploser les coûts cloud de la victime|Requêtes répétées sur AWS S3 (facturation à la requête)|
|Reputational Damage|Faire perdre la confiance des clients|Serveurs de jeu down le jour du lancement|

### In the Wild

- **BBC (31/12/2015)** : DDoS revendiqué par _New World Hacking_ — site offline plusieurs heures, motif : test de capacités.
- **Microsoft (2023)** : DDoS Layer 7 massif par _Anonymous Sudan_ — Azure, OneDrive, Outlook impactés. Techniques : HTTP Flood + Slowloris.

---

## Log Analysis

Les logs web (Apache, NGINX, IIS) sont la principale source d'analyse post-incident pour détecter un DoS/DDoS.

### Indicateurs clés

| Indicateur                                                                                                       | Exemple                            | Signification                               |
| ---------------------------------------------------------------------------------------------------------------- | ---------------------------------- | ------------------------------------------- |
| High Request Rate                                                                                                | 1000 GET /login depuis une IP      | Flood d'un endpoint lourd (auth, DB)        |
| Odd User-Agents                                                                                                  | `curl/7.6.88`, `Python-urllib/3.x` | Trafic automatisé, contournement de filtres |
| Geographic Anomalies                                                                                             | IPs réparties mondialement         | Botnet distribué                            |
| Burst Timestamps                                                                                                 | 50 requêtes en 1 seconde           | Pattern non humain, automation évidente     |
| Server Errors ([5xx](https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Status#server_error_responses)) | Spike de 503                       | Ressources saturées, service en échec       |
| Logic Abuse                                                                                                      | `GET /products?limit=999999`       | Requête crafted pour surcharger le serveur  |

> Chercher des **signaux combinés**, pas un seul indicateur isolé.

### Targeted Resources

Endpoints prioritaires pour un attaquant (coût serveur élevé) :

`/login` · `/search` · `/api/*` · `/register` · `/contact` · `/cart` · `/checkout`

### Lecture d'un access.log

Pattern type d'un DoS :

1. **Normal User Traffic** - Every few seconds, a user requests a page and receives a response as expected
2. **DoS Attack** - Beginning at `10:01:10`, you can see the IP address `203.0.113.55` begin to send repeated `GET` requests to `/login.php`
3. **Web Server Down** - Users are requesting pages and receiving `503` responses indicating the service is unavailable

![[detecting web DoS logs.png]]

---

## Leveraging SIEMs

Un SIEM (ex: Splunk) remplace l'analyse manuelle de logs bruts : agrégation multi-sources, filtrage et tri par champ (IP, user-agent, response code), visualisation temporelle.

### Splunk — Requêtes utiles

spl

```spl
index="main"
```

Champs clés à exploiter dans les logs web :

- `clientip` — identifier les IPs à fort volume
- `useragent` — détecter les agents automatisés
- `status` — filtrer les 5xx
- `uri_path` — repérer les endpoints ciblés

### Pattern de détection (timechart)

1. **Trafic normal** : quelques requêtes/minute sur des pages variées
2. **DoS/DDoS** : spike brutal (ex: 1000 req/min sur `/login.php`)
3. **Pivot** : filtrer par `clientip` + `useragent` pour identifier l'origine

![[detecting web DDoS splunk view.png]]

Looking over the same requests but filtering by the user agent (`useragent`) and IP address (`clientip`) fields enables you to see more details about where the requests originated.

![[detecting web DDoS filters.png]]

### Questions

#### What was the most frequently requested `uri`?

index="main"

![[detecting web DDoS t5q1.png]]

**Answer** : /search

#### Which `clientip` made the most requests to the target `uri`?

![[detecting web DDoS t5q2.png]]

**Answer** : 203.0.113.7

#### How many IP addresses were part of the botnet that attacked your website?

filter with the subnet of the ip to list all hosts :

index="main" clientip="203.0.113.*"

![[detecting web DDoS t5q3.png]]

**Answer** : 60

#### Which `useragent` was most commonly used by the attacking traffic?

with the same filter go to useragent in "ineresting fields"

![[detecting web DDoS t5q4.png]]

**Answer** : Java/1.8.0_181

#### Use the `timechart` command to visualize the requests.  What is the peak number of requests made per second during the attack?

using this filter : index="main" clientip="203.0.113.*" | timechart span=1s count by request

we can see in the output that 207 request per second is the highest peak

![[detecting web DDoS t5q5.png]]

**Answer** : 207

#### Which legitimate (non-attacking) `clientip` received the first `503` response status post-attack?

filter : index="main" status=503 clientip = "10.10.0.*"

go to the last page to see the first 503 status cade. the ip at the bottom is 10.10.0.27

**Answer** : 10.10.0.27

---

## Defense

### Application Level Defense

**Secure Development Practices** : valider toutes les entrées (longueur, format, caractères) pour empêcher les requêtes malformées de surcharger le serveur.

**Challenges** :

- CAPTCHA — bloque/ralentit les bots
- JavaScript challenge — s'exécute en arrière-plan, invisible pour l'humain, échoue pour les bots

### Network and Infrastructure Defenses

#### Content Delivery Network (CDN)

Le CDN met en cache le contenu et le sert depuis des **edge servers** proches des utilisateurs → l'origin server ne traite qu'une fraction du trafic.

Rôles clés :

- **Absorption DDoS** : le trafic d'attaque frappe les edge servers, pas l'origin
- **Load-balancing** : distribue le trafic, reroute si un serveur tombe
- **Visibilité analytique** : breakdown par géographie, volume, patterns → distinguer trafic légitime et malveillant

![[detecting web DDoS geo dashboard.png]]

Lecture d'un dashboard CDN (ex: Cloudflare) :

1. Bandwidth total anormalement élevé (ex: 16 TB vs quelques centaines de GB habituellement) → signal d'attaque
2. Quasi-totalité du bandwidth servie par le cache → CDN a absorbé l'attaque avant l'origin
3. Spike net sur le graphe temporel → signature du DDoS

![[detecting web DDoS cdn dashboard.png]]

#### Web Application Firewall (WAF)

Intégré au CDN, le WAF inspecte chaque requête entrante et applique une décision : **allow / challenge / block**.

Fonctionne sur :

- Règles basées sur la threat intelligence (indicateurs d'attaques connus)
- Règles custom pour les menaces ciblées

Exemple — règle de rate-limiting :

```
/login.php > 5 req/min par IP → block temporaire ou challenge
```

![[detecting web DDoS WAF rule exemple.png]]

### Large-Scale Mitigation

- Google (2023) : 398 millions req/sec mitigées
- Cloudflare : 11,5 Tbps mitigées en 35 secondes

### Bypassing Security Measures

Techniques d'attaquants pour contourner CDN/WAF :

|Technique|Effet|
|---|---|
|Random query params (`/products?a=abcd`)|Force l'origin server à répondre (cache bypass)|
|Rotation de User-Agents|Contourne les règles WAF basées sur l'UA|
|Spoofing du referrer|Évite les filtres de provenance|
|IPs géographiquement diversifiées|Contourne les règles de blocage géographique|

---

