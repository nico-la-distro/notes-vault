## Learning Objectives

- Géolocaliser des IPs et interpréter les ASNs
- Détecter une infrastructure suspecte via Shodan/Censys
- Évaluer la réputation avec différents outils
- Enrichir des domaines via WHOIS, DNS, certificate transparency

ASN (**Autonomous System Number**) : identifiant unique attribué à un bloc d'adresses IP géré par une même organisation (FAI, hébergeur, entreprise).
### Scenario

Triage de 5 artefacts suspects (phishing + proxy logs) :

|Artefact|Source|
|---|---|
|`advanced-ip-sccanner[.]com`|Email phishing|
|`166[.]1[.]160[.]118`|Proxy logs|
|`64[.]31[.]63[.]194`|Proxy logs|
|`69[.]197[.]185[.]26`|Proxy logs|
|`85[.]188[.]1[.]133`|Proxy logs|

Process SOC : `verify -> enrich -> decide`

Pivot sur IP : géoloc, ASN, open-service footprint, passive DNS.

---

## IP Building Blocks

### Why DNS Matters in the SOC

DNS = convertit hostnames en IPs. Vecteur d'attaque majeur : les domaines suspects apparaissent dans les alertes **avant** qu'un hash soit connu. Objectif : transformer un domaine brut en artefact contextuel (propriétaire, IPs résolues, fréquence de changement, comportement).

### Core DNS Records for Triage

|Record|Utilité SOC|
|---|---|
|A / AAAA|IP du domaine. Rotation rapide entre réseaux différents -> suspicion. Vérifier l'IP sur VirusTotal.|
|NS|Nameservers du domaine. Changement récent = setup frais = red flag.|
|MX|Serveurs mail. Configurable pour phishing. Si alerte web -> juste noter si MX existe.|
|TXT|SPF/DKIM. Absence ou mauvaise config = risque accru (cas mail).|
|SOA|Autorité primaire de la zone + contact. Utile pour ownership.|
|TTL|TTL très bas (secondes/minutes) = changements fréquents = indice malveillant.|
- **SPF** (Sender Policy Framework) : liste les serveurs autorisés à envoyer des emails pour un domaine.
- **DKIM** (DomainKeys Identified Mail) : signature cryptographique ajoutée aux emails pour vérifier l'authenticité de l'expéditeur.
- **SOA** (Start of Authority) : enregistrement DNS qui désigne le serveur DNS principal d'une zone et contient les métadonnées d'administration (contact, numéro de série, TTL).

Outils : [nslookup.io](https://nslookup.io) / [dnschecker.org](https://dnschecker.org)

#### Attack Techniques Using DNS

- **Fast Flux** : rotation rapide d'IPs avec TTL court pour éviter les blocages -> noter et escalader si IPs changent fréquemment sur plusieurs providers.
- **CDN Abuse** : les vrais CDNs (Cloudflare, Akamai) changent aussi d'IP mais restent dans leur ASN -> si A record pointe un CDN connu et tout est normal, faire quand même les checks réputation/ownership.
- **Typosquatting** : clone visuel d'une marque (`paypa1[.]com`) -> traiter comme haut risque, escalader.
- **IDN / Punycode** : Unicode look-alike (`xn--ppaypal-3ya[.]com`) -> décoder le Punycode et comparer à la marque connue.

_**IDN** (Internationalized Domain Name) : nom de domaine contenant des caractères Unicode (accents, alphabets non-latins). Encodé en **Punycode** pour être compatible DNS (ex: `pàypal.com` -> `xn--pypal-4ve.com`)._

#### SOC Analyst Workflow

1. **Snapshot DNS** : capturer A, NS, MX, TXT, SOA, TTL en une vue. 
	_cf : Documents\thm\ip_and_domain_threat_intel\advanced-ip-scanner.com.pdf_
2. **WHOIS** : noter registrar, date de création, pattern de contact.
3. **Interpréter** : CDN légitime ou domaine jetable ?
4. **Logger** : screenshots ou exports JSON dans le ticket.
5. **Recommander** : bloquer (haut risque) / surveiller (inconclusive) / clore (bénin).

---

## IP Enrichment: Geolocation and ASN

### IP Enrichment Within the SOC

Une IP seule est ambiguë (CDN, routeur compromis, C2...). L'enrichissement ajoute : ownership, ASN, géoloc, contexte service -> décision basée sur des preuves.

### The Role of RDAP

**RDAP** (Registration Data Access Protocol) : source authoritative pour l'ownership d'une IP, maintenue par les **RIR** (RIPE NCC, ARIN, APNIC).

_**RIR** (Regional Internet Registry) : organisme régional responsable de l'allocation et de l'enregistrement des blocs d'adresses IP et des ASNs dans sa zone géographique._

Données obtenues :

|Champ|Contenu|
|---|---|
|NetRange|Plage d'adresses déléguée|
|Organisation|Titulaire du bloc (ex: Amazon, Vodafone)|
|Remarks|Usage du bloc (hosting, broadband, mobile)|
|Abuse Contact|Mailbox officielle pour signalement|

Conserver les données RDAP en **JSON brut** pour l'audit (évite de dépendre de sources secondaires potentiellement obsolètes).

### Autonomous Systems and Heuristics

|Type d'ASN|Caractéristique|Comportement SOC|
|---|---|---|
|Hosting|Petits netblocks, tenants divers|Domaines suspects fréquents ici|
|ISP résidentiel|Grandes plages, millions d'users|Souvent routeur/device compromis|
|Cloud/CDN|Anycast global, edges partagés|Ne pas bloquer le range entier|

Exemples :

- **AS32934** (Meta) : activité malveillante -> problème de compte, pas d'hébergement malveillant
- **AS16509** (AWS) : abus fréquent pour serveurs éphémères -> scope sur FQDN ou CIDR étroit, jamais bloquer tout l'ASN
- **AS124888** (Vodafone) : ISP -> device client compromis

Outils : [bgpview.io](https://bgpview.io) / [ipinfo.io](https://ipinfo.io)

### Geolocation: Value and Limitations

- Précision pays : utiliser **au moins 2 sources**, noter les divergences
- Précision ville : non fiable -> jamais justifier un blocage sur cette base
- Jurisdiction : utile pour les escalades légales (takedown via abuse contact si pays coopératif)

Outils : [ipinfo.io](https://ipinfo.io) / [iplocation.net](https://iplocation.net)

### SOC Analyst Workflow

1. **RDAP** : netrange, org, ASN, abuse contact
2. **ASN context** : rôle de l'AS (bgpview.io / ipinfo.io)
3. **Géoloc** : pays depuis 2 sources, noter divergences
4. **rDNS** : indice sur le type d'hébergement (ex: `*.btcentralplus.com` = broadband UK) -> indicatif seulement
5. **Logs internes** : IP vue dans les 30 derniers jours ? Dans quel contexte ?
6. **Classifier** : hosting / residential / CDN / cloud + justification
7. **Outreach** : si malveillant confirmé + ASN coopératif -> préparer rapport pour abuse contact

---

## Service Exposure

### Shodan Reconnaissance

[Shodan](https://www.shodan.io/) indexe les devices/services exposés sur Internet. Utile pour identifier ports ouverts, services actifs, configurations.

Données extraites :

|Donnée|Utilité SOC|
|---|---|
|Open Ports|Premier fingerprint d'exposition (ex: RDP 3389 -> cible brute-force)|
|Service Banners|Type de serveur, version logicielle, cookies RDP/HTTP -> operator markers|

Exemple de requête : `org:example.com` -> tous les systèmes d'une organisation.

### TLS Certificate Transparency

[crt.sh](https://crt.sh/) : logs publics de tous les certificats TLS émis.

|Champ|Ce qu'il révèle|
|---|---|
|Issuer|Let's Encrypt = neutre. Self-signed = système déployé à la va-vite -> red flag|
|Validity Period|90 jours = normal. Bursts de renouvellements -> infra de phishing probable|
|Subject Alternative Names (SANs)|Domaines couverts par le certificat -> pivot vers infra liée|
_**SAN** (Subject Alternative Names) : champ d'un certificat TLS listant tous les domaines/IPs couverts par ce certificat. Ex: un certificat pour `example.com` peut avoir comme SANs `www.example.com`, `mail.example.com`, `api.example.com`._

### Censys Search

[Censys.io](https://search.censys.io/) : alternative Shodan, détecte les services sur **ports non-standard** aussi (ex: `56003/SSH`). Certaines fonctions nécessitent un compte.

### SOC Analyst Workflow

1. **Shodan/Censys** : services exposés, misconfigurations
2. **TLS** : noter issuer, SANs, validity period
3. **Anomalies** : SANs multiples non liés, look-alikes de marques, bursts d'émission
4. **Pivot** : utiliser les artefacts banners/certificats pour trouver l'infra liée
5. **Blast radius** :

|Pattern|Interprétation|
|---|---|
|RDP/SSH sur ASN résidentiel|Endpoint compromis probable|
|TLS multi-SANs non liés sur CDN|Infra partagée -> ne pas bloquer l'IP|
|Self-signed TLS sur petit range|Panel attaquant ou proxy probable|

---

## Reputation Checks and Passive DNS

### Why Reputation and History Matter

IPs et domaines sont des **assets dynamiques** : un domaine de phishing peut être actif aujourd'hui et parked demain, une IP réassignée en quelques jours. -> Le contexte temporel est critique.

### Reputation Services

|Outil|Ce qu'il apporte|
|---|---|
|[VirusTotal](https://www.virustotal.com)|Detection ratio, relations entre indicateurs, First/Last Seen|
|[Cisco Talos](https://talosintelligence.com)|Score réputation web/email, category labels, CVEs + CVSS, règles Snort, données spam|
|[IP2Proxy](https://www.ip2proxy.com)|Détecte VPN, proxy, Tor exit nodes -> affaiblit l'attribution, ajuster la sévérité|

**Talos** : dashboard email traffic (légitime / spam / malware) + Reputation Centre (IPs et hashes SHA256) + onglet Email & Spam Data.

### Passive DNS

Historique des résolutions DNS d'un domaine dans le temps.

|Signal|Interprétation|
|---|---|
|First Seen / Last Seen|Domaine nouveau = risque élevé|
|Nombre d'IPs sur une fenêtre courte|Churn élevé -> fast flux ou agile hosting|
|ASN Spread|Plusieurs ASNs non liés -> suspect / un seul ASN -> stable ou CDN|

Autres sources historiques :

- **CT Logs** : bursts soudains de certificats sous un même thème -> infra phishing
- **Wayback Machine** : contenu historique du domaine. Blog pendant des années puis phishing kit la semaine dernière -> haut risque

### SOC Analyst Workflow

1. **VirusTotal** : detection ratio, First/Last Seen, notes communauté
2. **Cisco Talos** : score réputation, category, changements sur 30 jours
3. **IP2Proxy** : VPN/proxy/Tor détecté -> ajuster sévérité
4. **Passive DNS** : First/Last Seen, nb d'IPs sur 7 jours, ASN spread
5. **CT Logs** : bursts de certificats, SANs suspects
6. **Wayback Machine** : shifts de contenu (bénin -> phishing)
7. **Décision** : bloquer / surveiller / clore, avec expiry liée à l'activité observée

---

## Operational Integration

### Safe Integration Patterns

|Contexte|Action recommandée|
|---|---|
|Domaine stable sur CDN/anycast|Filtrer par hostname (DNS RPZ, proxy categories, SNI filtering) -> pas par IP|
|VPS dédié (staging/C2)|Bloquer en `/32` -> blast radius minimal|
|Tout blocage|Expiry 7-14 jours + auto-renew si ré-observation|

Documenter dans le SOAR : screenshots, RDAP, certificats, raisonnement.

### Geofencing Cautions

Géoblocking = attractif mais dangereux : collègues en déplacement, services tiers sur PoPs étrangers, TLS terminé dans des régions inattendues. -> Géoloc = enrichissement pour augmenter la priorité, **pas un contrôle primaire** sans validation business.

### Cloud and Large Provider Pitfalls

Ne jamais bloquer un range entier CDN/cloud (AWS, Microsoft...) -> IPs partagées entre des milliers de clients. -> Si hostname malveillant sur cloud edge : agir au niveau **domaine ou path**, utiliser le processus abuse du provider.

### Legal and Provider Considerations

Provider + pays -> détermine si evidence preservation ou takedown rapide est réalisable. -> Toujours conserver : ownership RIR + abuse contacts RDAP pour les escalades.

### From Data to Decision

|Étape|Action|
|---|---|
|Verify|Confirmer que l'indicateur est dans la télémétrie et pertinent|
|Enrich|Géoloc, ASN, banners, certificats, réputation, historique|
|Score|Appliquer la confidence matrix, documenter le raisonnement|
|Decide|Bloquer / surveiller / autoriser. Contrôles précis + expiry + documentation|
|Hunt & Notify|Chercher les indicateurs liés, informer les stakeholders, créer des follow-up tasks|

---

## Challenge

It’s 09:10 on a Monday. Over the weekend, Finance reported a burst of “account verification” emails that looked unusually polished. Your secure email gateway caught a subset; one clicked sample was redirected to `santagift[.]shop`.  
At the same time, your EDR shows workstations briefly beaconing to `170[.]130[.]202[.]134`.

Use the skills and tools covered in the room to enrich the three indicators and answer the questions below.

### Questions
#### What is the RIR associated with 170[.]130[.]202[.]134?

go check the ip info on ipinfo.io, then clic on the ASN and check for the RIR

![[ip_domain_ti_t7q1.png]]

**Answer** : ARIN

#### What ASN is the IP connected with?

we already have the ASN from the previous research

**Answer** : AS62904

#### Identify the number of NS records for the domain santagift[.]shop

go to nslookup.io then search for the domain santagift[.]shop

**Answer** : 4

#### Which NS is identified as the Start of Authority (SOA) for the domain?

check for SOA Data on the same research on nslookup.io

**Answer** : ns-298[.]awsdns-37[.]com

---

## Résumé - IP and Domain Threat Intel

### Process central

```
Verify -> Enrich -> Score -> Decide -> Hunt & Notify
```

### Les 4 axes d'enrichissement

**1. DNS & WHOIS**

- Records clés : A, NS, MX, TXT, SOA, TTL
- TTL bas + rotation d'IPs = fast flux
- WHOIS : date de création, registrar, pattern de contact
- Outils : nslookup.io, dnschecker.org, viewdns.info

**2. IP Ownership & ASN**

- RDAP -> source authoritative (org, netrange, abuse contact)
- ASN -> type d'organisation derrière l'IP

|Type ASN|Décision SOC|
|---|---|
|Hébergeur cheap|Bloquer `/32`|
|ISP résidentiel|Ne pas bloquer, chercher device compromis|
|Cloud/CDN|Agir au niveau domaine/path uniquement|

- Géoloc -> 2 sources minimum, indicatif seulement, jamais base de blocage seule
- Outils : client.rdap.org, bgpview.io, ipinfo.io

**3. Service Exposure**

- Ports ouverts + banners -> fingerprint d'exposition
- TLS : issuer, SANs, validity period, bursts de renouvellement
- Self-signed sur petit range = panel attaquant probable
- Outils : Shodan, Censys, crt.sh

**4. Réputation & Passive DNS**

- Detection ratio, First/Last Seen, community notes
- Passive DNS : churn d'IPs, ASN spread, historique de résolution
- CT logs : bursts de certificats -> infra phishing
- Wayback Machine : shift de contenu bénin -> malveillant
- Outils : VirusTotal, Cisco Talos, IP2Proxy

### Règles de blocage

|Situation|Action|
|---|---|
|VPS dédié confirmé|Bloquer `/32`, expiry 7-14j|
|Domaine stable|Filtrer par hostname (SNI/proxy)|
|CDN/cloud|Domaine ou path uniquement, jamais le range|
|Géolocation suspecte|Enrichissement uniquement, pas de geofencing sans validation|

### Red flags à retenir

- TTL très bas + rotation IPs multi-ASNs = fast flux
- Aucun TXT/MX + domaine récent = domaine jetable
- Self-signed TLS = déploiement rapide non professionnel
- Privacy/anonymisation détectée sur hébergeur = infra attaquant probable
- Burst de certificats sous un même thème = campagne phishing
- Exit node Tor = attribution impossible, augmenter la sévérité